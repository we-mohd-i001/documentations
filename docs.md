---
toc: true
title: Storage
description: null
---

Vaah Flutter allows its users to use storage options without any hassle. You just need to pass a couple of parameters in [environment configuration](../3.essentials/2.environments.md) and the storage service is ready to use.

- Flow diagram depicting the selection process where developers can choose both Local and Network Storage simultaneously, but only one option from the three available in each storage type.
    <img src="/images/flutter/storage/storage-class-options.png" alt="storage-class-options.png">


## Overview

The `Storage` class is an abstract class that provides an interface for [local storage](../5.directory_structure/3.vaahextendflutter/5.services/storage/1.local_storage.md) and [network storage](../5.directory_structure/3.vaahextendflutter/5.services/storage/2.network_storage.md)(not implemented yet) implementations. 

It supports two local storages to select from : Hive and Flutter Secure Storage. Depending on the configuration, it can create an instance of either `HiveStorageImpl` or `FlutterSecureStorageImpl`. If no valid storage type is specified, a `NoOpStorage` implementation is used, in which all methods are empty.

::alert{type="info" class="flex items-center p-4 mb-4 text-sm text-blue-800 rounded-lg bg-blue-50 dark:bg-gray-800 dark:text-blue-400" role="alert"}
Note: For Usage Guide and Source code. [Local](../5.directory_structure/3.vaahextendflutter/5.services/storage/1.local_storage.md) and [Network](../5.directory_structure/3.vaahextendflutter/5.services/storage/2.network_storage.md).
::

## Factory Constructors

|        Name        |    Parameters   |  Returns    |  Description |
|        :---        |     :---        |    :----    |     :---     |
|   `createLocal()`  | (Optional) `name (String)` default is 'default' | An instance of `HiveStorageImpl`, `FlutterSecureStorageImpl`, or `NoOpStorage`. | Creates a new local storage instance based on the environment configuration. |


## Methods

| Name          | Parameters |  Returns    |  Description    |
|    :---       |   :----    |    :----    |     :---        |
| `init()`      | None       | `Future<void>` | Initializes the storage. For `HiveStorageImpl`, it sets up the Hive directory and opens a box. Not required for `FlutterSecureStorageImpl`. |
| `create()`    | `key (String)` & `value (String)` | `Future<void>` | Creates or updates a single key-value pair in the storage. |
| `createMany()` | `values (Map<String, String>)` | `Future<void>` | Creates or updates multiple key-value pairs in the storage. |
| `read()`      | `key (String)` | `Future<String?>` | Reads the value of the item at the specified key from the storage. |
| `readMany()`   | (Optional) `keys (List<String>)` default is empty List  | `Future<Map<String, String?>>` | Reads multiple values from the storage. If no keys are provided, it returns all values. |
| `delete()`    | `key (String)` | `Future<String?>` | Deletes the item at the specified key from the storage. |
| `deleteMany()` | (Optional) `keys (List<String>)` default is empty List | `Future<void>` | Deletes multiple items from the storage. If no keys are provided, it deletes all values. | 



---
toc: true
title: Local Storage
description: Documentation on using local storage options (Hive and Flutter Secure Storage) with Vaah Flutter.
---

## Overview

Vaah Flutter provides two local storage options: Hive and Flutter Secure Storage. This section guides you through setting up and using these local storage solutions.

Select one option from Hive and Flutter Secure Storage.

| Hive  | Flutter Secure Storage |
| :---- |          :----         |
| The `HiveStorageImpl` class implements the `Storage` interface using Hive as the storage backend. It is used when `LocalStorageType.hive` is selected in the configuration. | The `FlutterSecureStorageImpl` class implements the `Storage` interface using Flutter Secure Storage as the storage backend. It is used when `LocalStorageType.flutterSecureStorage` is selected in the configuration. |

## Setup

### If you select Hive

Configure the storage type in the env.dart file.

```dart
final EnvironmentConfig defaultConfig = EnvironmentConfig(
  // other configurations
  localStorageType: LocalStorageType.hive,
  hiveConfig: HiveConfig(),
);
```

### If you select Flutter Secure Storage

Configure the storage type in the env.dart file.

```dart
final EnvironmentConfig defaultConfig = EnvironmentConfig(
  // other configurations
  localStorageType: LocalStorageType.flutterSecureStorage,
);
```

::alert{type="info" class="flex items-center p-4 mb-4 text-sm text-blue-800 rounded-lg bg-blue-50 dark:bg-gray-800 dark:text-blue-400" role="alert"}
For more information about environment configuration [click here](../../../../3.essentials/2.environments.md).
::

## Usage Guide

### Step 1: Create a Storage Instance

```dart
final Storage storage = Storage.createLocal(name: 'local'); 
```

### Step 2: Initialize Storage (Only in case of Hive)

```dart
@override
void initState() {
  super.initState();
  storage.init(); // initialize for Hive
}
```

### Step 3: Use the methods

#### **Create or Update Items**

```dart
await storage.create(key: 'key34', value: '34'); // single
await storage.createMany(values: {
  'key2': 'Value2',
  'key3': 'Value3',
}); // multiple
```

#### **Read Items**

```dart
final value = await storage.read(key: 'key34'); // single
final values = await storage.readMany(keys: ['key2', 'key3']); // multiple
final values = await storage.readMany(); // all
```

#### Delete Items

```dart
await storage.delete(key: 'key34'); // single
await storage.deleteMany(keys: ['key2', 'key3']); // multiple
await storage.deleteMany(); // all
```

## Best Practices
### With `toJson` and `fromJson` Methods
To store and retrieve complex objects, use toJson and fromJson methods for serialization and deserialization.

### Example
1. Define Data Model
``` dart
class User {
  String id;
  String name;
  String email;

  User({
    required this.id,
    required this.name,
    required this.email,
  });

  Map<String, dynamic> toMap() {
    return {
      'id': id,
      'name': name,
      'email': email,
    };
  }

  factory User.fromMap(Map<String, dynamic> map) {
    return User(
      id: map['id'] as String,
      name: map['name'] as String,
      email: map['email'] as String,
    );
  }

  String toJson() => json.encode(toMap());
  factory User.fromJson(String source) => User.fromMap(json.decode(source));
}
```

2. Store a User Object
```dart
final User user = User(id: '1', name: 'John Doe', email: 'john.doe@example.com');
await storage.create(key: 'user_1', value: user.toJson());
```

3. Retrieve a User Object
```dart
final String? userJson = await storage.read(key: 'user_1');
if (userJson != null) {
  final User user = User.fromJson(userJson);
}
```

## Source Code
### HiveStorageImpl Class
This class implements the Storage interface using Hive as the storage backend.

```dart
import 'package:hive/hive.dart';
import '../../storage.dart';

class HiveStorageImpl implements Storage {
  final String name;

  Box? _box;

  HiveStorageImpl({this.name = 'default'});

  @override
  Future<void> init() async {
    _box = await Hive.openBox(name);
  }

  @override
  Future<void> create({required String key, required String value}) async {
    await _box?.put(key, value);
  }

  @override
  Future<void> createMany({required Map<String, String> values}) async {
    await _box?.putAll(values);
  }

  @override
  Future<String?> read({required String key}) async {
    return _box?.get(key);
  }

  @override
  Future<Map<String, String?>> readMany
({List<String> keys = const []}) async {
    if (keys.isEmpty) {
      return Map<String, String?>.from(_box?.toMap() ?? {});
    } else {
      return Map.fromEntries(keys.map((key) => MapEntry(key, _box?.get(key))));
    }
  }

  @override
  Future<void> delete({required String key}) async {
    await _box?.delete(key);
  }

  @override
  Future<void> deleteMany({List<String> keys = const []}) async {
    if (keys.isEmpty) {
      await _box?.clear();
    } else {
      await _box?.deleteMany(keys);
    }
  }
}
```

### FlutterSecureStorageImpl Class

This class implements the Storage interface using Flutter Secure Storage as the storage backend.

```dart
import 'package:flutter_secure_storage/flutter_secure_storage.dart';
import '../../storage.dart';

class FlutterSecureStorageImpl implements Storage {
  final _storage = FlutterSecureStorage();

  @override
  Future<void> init() async {
    // No initialization required for Flutter Secure Storage
  }

  @override
  Future<void> create({required String key, required String value}) async {
    await _storage.write(key: key, value: value);
  }

  @override
  Future<void> createMany({required Map<String, String> values}) async {
    await Future.wait(values.entries.map((e) => _storage.write(key: e.key, value: e.value)));
  }

  @override
  Future<String?> read({required String key}) async {
    return await _storage.read(key: key);
  }

  @override
  Future<Map<String, String?>> readMany
({List<String> keys = const []}) async {
    if (keys.isEmpty) {
      return await _storage.readMany
    ();
    } else {
      Map<String, String?> result = {};
      for (String key in keys) {
        result[key] = await _storage.read(key: key);
      }
      return result;
    }
  }

  @override
  Future<void> delete({required String key}) async {
    await _storage.delete(key: key);
  }

  @override
  Future<void> deleteMany({List<String> keys = const []}) async {
    if (keys.isEmpty) {
      await _storage.deleteMany();
    } else {
      for (String key in keys) {
        await _storage.delete(key: key);
      }
    }
  }
}
```