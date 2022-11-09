---
layout: single
title:  "FastAPI Custom APIRoute to log JSON Body Parameters and Elapsed Time"
date:   2022-11-09 09:01:00 +0900
categories: fastapi
---

When you use Flask, it is effortless to get json body parameters to log with the 'request' instance.
```python
    body_param = request.json()
```
However, in the case of FastAPI, getting the body parameters is different from the way of Flask.
"request.json()" encounters TypeError as below,

> if body_params is not None and len(body_params) > 0:
TypeError: object of type 'coroutine' has no len()

Debugging lets us know Request.json is a co-routine object.
![request.json() error](/docs/assets/2022-11-09-json-body-param.png)

Since the request.json is a co-routine, it needs to be awaited. 
FastAPI is based on [Starlette](https://www.starlette.io/requests/#body) 
which methods return the request body asynchronously with async methods.<br>
Ref: [Using FastAPI in a sync way, how can I get the raw body of a POST request?](https://stackoverflow.com/questions/70658748/using-fastapi-in-a-sync-way-how-can-i-get-the-raw-body-of-a-post-request)
>Body
There are a few different interfaces for returning the body of the request:
> 
> The request body as bytes: await request.body()
> 
> The request body, parsed as form data or multipart: await request.form()
> 
> The request body, parsed as JSON: await request.json()

[Starlette Requests](https://www.starlette.io/requests/#body)

According to the above official document, we can access the request body as a stream, 
using the async as the following code.

```python
from starlette.requests import Request
from starlette.responses import Response


async def app(scope, receive, send):
    assert scope['type'] == 'http'
    request = Request(scope, receive)
    body = b''
    async for chunk in request.stream():
        body += chunk
    response = Response(body, media_type='text/plain')
    await response(scope, receive, send)
```

So, if the purpose of getting json body parameters is not logging, 
we can get the json with the async.
```python
from fastapi import Request

@app.post("/input")
async def input_request(request: Request):
    return await request.body()
```

But, if you use "await request.body()" or "await request.json()" in middleware blocks, 
the method is blocked. According to @dmontagu says, the blocking is a Starlette problem.
<br>
[[BUG] Awaiting request body in middleware blocks the application #394](https://github.com/tiangolo/fastapi/issues/394)
He suggests a neat workaround using "dependencies."
```python
from typing import Mapping

from starlette.requests import Request

from fastapi import FastAPI, APIRouter, Depends

app = FastAPI()
api_router = APIRouter()

@api_router.post("/")
def read_root(arg: Mapping[str, str]):
    return {"Hello": "World"}

async def log_json(request: Request):
    print(await request.json())

# the trick here is including log_json in the dependencies:
app.include_router(api_router, dependencies=[Depends(log_json)])

```

As the workaround that @dmontagu suggests, there is a log method example @"Will Udstrand" answered using "dependencies".
```python
app = FastAPI()
api_router = APIRouter()


async def log_request_info(request: Request):
    request_body = await request.json()

    logger.info(
        f"{request.method} request to {request.url} metadata\n"
        f"\tHeaders: {request.headers}\n"
        f"\tBody: {request_body}\n"
        f"\tPath Params: {request.path_params}\n"
        f"\tQuery Params: {request.query_params}\n"
        f"\tCookies: {request.cookies}\n"
    )


@api_router.get("/", summary="Status")
async def status_get():
    logger.debug('Status requested')
    return {'status': 'OK'}


@api_router.post("/", )
def status_post(urls: Optional[List[str]] = None):
    logger.debug('Status requested')
    return {'status': 'OK'}


app.include_router(api_router, dependencies=[Depends(log_request_info)])

```
[FASTAPI custom middleware getting body of request inside](https://stackoverflow.com/questions/69669808/fastapi-custom-middleware-getting-body-of-request-inside)

Using dependencies looks neat and simple. 
But I wonder if it is the most efficient way to log.
Adding dependencies may influence FastAPI's processing of tons of requests.
Moreover, it seems complicated to measure the elapsed time before/after request processing in the dependencies method.
Reversely, The elapsed time is measured by the following code in the middleware.
```python
@app.middleware('http')
async def logging_request_middleware(request: Request, call_next):
    start_time = time.time()
    print('request')
    response = await call_next(request)
    diff = time.time() - start_time

```

After more googling, I found another solution that @"Yagiz Degirmenci" answered using Custom Rout.
```python
from fastapi import APIRouter, FastAPI, Request, Response, Body
from fastapi.routing import APIRoute

from typing import Callable, List
from uuid import uuid4


class ContextIncludedRoute(APIRoute):
    def get_route_handler(self) -> Callable:
        original_route_handler = super().get_route_handler()

        async def custom_route_handler(request: Request) -> Response:
            request_id = str(uuid4())
            response: Response = await original_route_handler(request)

            if await request.body():
                print(await request.body())

            response.headers["Request-ID"] = request_id
            return response

        return custom_route_handler


app = FastAPI()
router = APIRouter(route_class=ContextIncludedRoute)


@router.post("/context")
async def non_default_router(bod: List[str] = Body(...)):
    return bod


app.include_router(router)
```

The official document of FastAPI is [Custom Request and APIRoute class](https://fastapi.tiangolo.com/advanced/custom-request-and-route/?h=custom#custom-apiroute-class-in-a-router)
The purpose of Custom APIRoute is to override the logic by APIRoute class.

>In some cases, you may want to override the logic used by the Request and APIRoute classes.
> 
>In particular, this may be a good alternative to logic in a middleware.
> 
> For example, if you want to read or manipulate the request body before it is processed by your application.

### The final version for logging JSON Body Parameters and Elapsed Time
Finally, logging JSON body parameters and elapsed time using the custom APIRoute is as below,

```python
import time
from typing import Callable

import uvicorn
from fastapi import FastAPI, Request, Response, APIRouter
from fastapi.routing import APIRoute
from requests import Response

from server.JAModel import ResponseOp
from util import GlobalInfo, Logger


class TimedRoute(APIRoute):
    def get_route_handler(self) -> Callable:
        original_route_handler = super().get_route_handler()
        super(TimedRoute, self).get_route_handler()

        async def custom_route_handler(request: Request) -> Response:
            before = time.time()
            response: Response = await original_route_handler(request)
            duration = time.time() - before
            response.headers["X-Response-Time"] = str(duration)
            print(f"route duration: {duration}")
            print(f"route response: {response}")
            print(f"route response headers: {response.headers}")

            print(await request.json())
            return response

        return custom_route_handler


api_router = APIRouter(route_class=TimedRoute)


@api_router.post("/reports", response_model=ResponseOp)
async def add_report():
    return {
        "success": True,
    }


logger = Logger()
logger.setup('log/test_server.json')

app = FastAPI()
app.include_router(api_router)
# the following way doesn't log the elapse time


if __name__ == "__main__":
    try:
        uvicorn.run(app, host="0.0.0.0", port=8000)
        logger.logger.info("Server Started")
    except Exception as e:
        logger.logger.exception(GlobalInfo.FATAL_ERROR)

```