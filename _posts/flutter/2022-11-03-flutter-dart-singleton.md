---
layout: single
title:  "flutter dart singleton example"
date:   2022-11-03 13:42:00 +0900
categories: flutter
---

Singleton is useful to handle globally used instances as loggers and DB connectors. 
Dart provides the factory constructor to implement the Singleton class.

> #### Factory constructors <br>
Use the factory keyword when implementing a constructor that doesn’t always create a new instance of its class. For example, a factory constructor might return an instance from a cache, or it might return an instance of a subtype. Another use case for factory constructors is initializing a final variable using logic that can’t be handled in the initializer list.

Ref: [A tour of the Dart language](https://dart.dev/guides/language/language-tour#factory-constructors)

The following code is a simple example of the Dart Singleton class.
```dart
class TestClass {
  
  int num = 0;
  
  static final TestClass _instance = TestClass._();
  factory TestClass() {
    return _instance;
  }
  
  TestClass._() {
    num = 3;
  }
  
}

void main() {
  
  var t = TestClass();
  t.num = 5;
  print(t.num);
  
  var t2 = TestClass();
  print(t2.num);
  
}
```
Instance t and t2 are the same singleton class. Therefore, the console output is as below,
```
5
5
```
## References
[How do you build a Singleton in Dart?](https://stackoverflow.com/questions/12649573/how-do-you-build-a-singleton-in-dart)