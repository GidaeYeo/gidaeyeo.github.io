---
layout: single
title:  "flutter sqlite db location in iOS simulation"
date:   2022-11-01 07:29:00 +0900
categories: flutter
---

In the case of iOS simulator, you can easily find the sqlite database storage location with the following code.

```flutter
    getDatabasesPath().then( (value) => print(value) );
```

The result is as below,
flutter: /Users/*** your name *** /Library/Developer/CoreSimulator/Devices/4769664D-2588-4E47-A29F-5CD9DB801595/data/Containers/Data/Application/3F53C2F3-687C-4E3D-8611-66A6D8BFC84B/Documents

## References
[flutter sqlite database storage location](https://stackoverflow.com/questions/64590291/flutter-sqlite-database-storage-location/74057754#74057754)