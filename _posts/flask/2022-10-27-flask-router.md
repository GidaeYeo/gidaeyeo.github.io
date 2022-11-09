---
layout: single
title:  "FastAPI distribute (separate) tons of routers using APIRouter and include_router"
date:   2022-10-27 13:04:00 +0900
categories: fastapi
---

FastAPI is an excellent choice when you need to create a RESTful API server quickly with reliable and robust features
always required while running real commercial production services.

My previous project used Flask for a Python-based backend server.
Flask was quite efficient in building a backend server compared to the vast and complex Spring.
But some missed features could have been more convenient.
One of those features is the Response Model, which gets away with tons of typing keys worrying about misspellings.
FastAPI naturally provides Response Model!!!

Moreover, FastAPI includes routers defined on the other files with include_routers.

```python
    from fastapi import FastAPI

    app = FastAPI()
    app.include_router(api_router)
```

Routers are defined anywhere by APIRouter.

```python
    from fastapi import APIRouter

    api_router = APIRouter()

    @api_router.post("/users", response_model=ResponseOp)
    async def add_user(user: UserModel):
        ja_server = JAServer()
        user_key = ja_server.add_user(user)

        return {
            "success": True,
            "data": [
                {
                    "user_key": user_key
                }
            ]
        }
```
    
Just use APIRouter instead of FastAPI on the annotation decorator.