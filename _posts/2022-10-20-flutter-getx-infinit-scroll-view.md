---
layout: post
title:  "flutter infinite scroll view with getx"
date:   2022-10-20 11:13:18 +0900
categories: flutter
---

This post is about the infinite scroll view with getx.
There are lots of pages regarding the infinite scroll view. But I needed to know how to use the infinite scroll view with getx correctly.
I use getx for state management and routing.




#### Triggering specific callbacks when an event occurs.
    /// Called every time `count1` changes.
    ever(count1, (_) => print("$_ has been changed"));

    /// Called only first time the variable $_ is changed
    once(count1, (_) => print("$_ was changed once"));
    
    /// Anti DDos - Called every time the user stops typing for 1 second, for example.
    debounce(count1, (_) => print("debouce$_"), time: Duration(seconds: 1));
    
    /// Ignore all changes within 1 second.
    interval(count1, (_) => print("interval $_"), time: Duration(seconds: 1));

#### http query parameters

    final queryParameters = {
        'param1': 'one',
        'param2': 'two',
    };

    final uri =
        Uri.https('www.myurl.com', '/api/v1/test/${widget.pk}', queryParameters);
    final response = await http.get(uri, headers: {
        HttpHeaders.authorizationHeader: 'Token $token',
        HttpHeaders.contentTypeHeader: 'application/json',
    });

#### References
* [Flutter : Implementing Pagination (Infinite Scroll) with GetX State Management](https://anangnugraha.medium.com/flutter-implementing-pagination-with-getx-state-management-6b824b1e1eb5)
  * I wrote this example referring to this post.
* [How do you add query parameters to a Dart http request?](https://stackoverflow.com/questions/52824388/how-do-you-add-query-parameters-to-a-dart-http-request)
  * http query parameters
* [Reactive State Manager](https://chornthorn.github.io/getx-docs/state-management/reactive-state-manager/index/)
  * about ever()
* [Dart getters and setters](https://dev.to/newtonmunene_yg/dart-getters-and-setters-1c8f)
  * about dart getter and setter
  * int get page;