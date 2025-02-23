# Storage

Vaah Flutter allows its users to use storage options without any hassle. You just need to add some dependencies and the storage service is ready to use.

![Abstraction chart](https://img-v1.dev.getdemo.dev/screenshot/chrome_sJoUbLh7vm.png)


Currently, it provides two local storage options: Hive and Flutter secure storage.

## Overview

The `Storage` class is an abstract class that provides an interface for local storage implementations. It supports two types of local storage: Hive and Flutter Secure Storage. Depending on the configuration, it can create an instance of either `HiveStorageImpl` or `FlutterSecureStorageImpl`. If no valid storage type is specified, a `NullStorage` implementation is used, which throws `UnimplementedError` for all methods.

## Factory Constructor

### `createLocal({String name = 'default'})`

Creates a new local storage instance based on the configuration in `EnvironmentConfig`.

- **Parameters:**
  - `name` (String): The name of the Hive box to be opened. Default is 'default'.
  
- **Returns:**
  - An instance of `HiveStorageImpl`, `FlutterSecureStorageImpl`, or `NullStorage`.

## Methods

### `init()`

Initializes the storage. For `HiveStorageImpl`, it sets up the Hive directory and opens a box. Not required for `FlutterSecureStorageImpl`.

- **Returns:**
  - `Future<void>`

### `create({required String key, required String value})`

Creates or updates a single key-value pair in the storage.

- **Parameters:**
  - `key` (String): The key for the item.
  - `value` (String): The value for the item, can be a JSON string or plain text.
  
- **Returns:**
  - `Future<void>`

### `createAll({required Map<String, String> values})`

Creates or updates multiple key-value pairs in the storage.

- **Parameters:**
  - `values` (Map<String, String>): A map of key-value pairs to be saved or updated.
  
- **Returns:**
  - `Future<void>`

### `read({required String key})`

Reads the value of the item at the specified key from the storage.

- **Parameters:**
  - `key` (String): The key for the item.
  
- **Returns:**
  - `Future<String?>`: The value of the item, or `null` if not found.

### `readAll({List<String> keys = const []})`

Reads multiple values from the storage. If no keys are provided, it returns all values.

- **Parameters:**
  - `keys` (List<String>): A list of keys to be read. Default is an empty list.
  
- **Returns:**
  - `Future<Map<String, String?>>`: A map of key-value pairs.

### `delete({required String key})`

Deletes the item at the specified key from the storage.

- **Parameters:**
  - `key` (String): The key for the item to be deleted.
  
- **Returns:**
  - `Future<void>`

### `deleteAll({List<String> keys = const []})`

Deletes multiple items from the storage. If no keys are provided, it deletes all values.

- **Parameters:**
  - `keys` (List<String>): A list of keys to be deleted. Default is an empty list.
  
- **Returns:**
  - `Future<void>`

## **How to use**

Follow the step-by-step process given below.

### **Step 1**: Add the depedencies to your `pubspec.yaml` file.
- For Flutter Secure Storage
  ```yaml
  flutter_secure_storage: ^9.1.1
  ```
- For Hive
  ```yaml
  hive: ^2.2.3
  path_provider: ^2.1.3
  ```  

### **Step 2**: In the `env.dart` file, provide the local storage type you want to use, if you select Hive, provide the `HiveConfig` too in the same `env.dart` file as shown in the snippet below.

```dart 
final EnvironmentConfig defaultConfig = EnvironmentConfig(
  appTitle: 'VaahFlutter',
  appTitleShort: 'VaahFlutter',
  envType: 'default',
  version: version,
  build: build,
  backendUrl: '',
  apiUrl: '',
  timeoutLimit: 20 * 1000, // 20 seconds
  enableLocalLogs: true,
  enableCloudLogs: true,
  enableApiLogInterceptor: true,
  pushNotificationsServiceType: PushNotificationsServiceType.none,
  internalNotificationsServiceType: InternalNotificationsServiceType.none,
  showDebugPanel: true,
  debugPanelColor: AppTheme.colors['black']!.withOpacity(0.8),
  localStorageType: LocalStorageType.hive, //select storage type
  hiveConfig: HiveConfig(), //Provide HiveConfig
);
```



### **Step 3**: Create an object of the `Storage` class using the named constructors and store it in a variable. 
```dart
final Storage storage = Storage.createLocal(name: 'local'); 
//no need to provide ‘name’ in case of flutter secure storage.
```




### **Step 4**: Initialize the storage before using it.
```dart
@override
void initState() {
  super.initState();
  storage.init(); //initialize the storage
}

// you can use any other approach for initialization 
```
    Note: There is no need to initialize if you have selected flutter secure storage.

### **Step 5**: Use the methods from the Storage class to manage your data.

- `create()` method, creates or updates an item at a `key` with a `value`.
  ```dart
  await storage.create(key: 'key34', value: '34');
  ```

- `createAll()` method, created multiple items using the `Map<String, String>` provided.
  ```dart
  await storage.createAll(values: {
        'key2': 'Value2',
        'key3': 'Value3',
        'key4': 'Value4',
        'key5': 'Value5',
      });
  ```
- `read()` method, reads a single item at the `key` provided.
  ```dart
  final value = await storage.read(key: 'key34');
  ```

- `readAll()` method, reads multiple items according to the parameter `keys` provided.

  - Read all items.
  ```dart
  final values = await storage.readAll();
  ```
  
  - Read some items.
  ```dart
  final values = 
    await storage.readAll(keys: ['key1', 'key2', 'key3']); //pass the parameter keys containing keys 
  ```

- `delete()` method, deletes a single item at the `key` provided.
  ```dart
  await storage.delete(key: 'key34');
  ```

- `deleteAll()` method, deletes multiple items according to the parameter `keys` provided. 

  - Delete all items.
  ```dart
  await storage.deleteAll();
  ```
  - Delete some items.
  ```dart
  await storage.deleteAll(keys: ['key1', 'key2', 'key3']); //pass the parameter keys containing keys 
  ```

## Best practices

### Using toJson and fromJson Methods
To store and retrieve complex objects, use toJson and fromJson methods. This approach allows you to handle any type of data by converting objects to JSON strings before storing them and parsing them back to objects after reading them.

### Example
#### Define your data model with toJson and fromJson methods:

```dart

class User {
  String id;
  String name;
  String email;
  User({
    required this.id,
    required this.name,
    required this.email,
  });

  User copyWith({
    String? id,
    String? name,
    String? email,
  }) {
    return User(
      id: id ?? this.id,
      name: name ?? this.name,
      email: email ?? this.email,
    );
  }

  Map<String, dynamic> toMap() {
    return <String, dynamic>{
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

  factory User.fromJson(String source) => User.fromMap(json.decode(source) as Map<String, dynamic>);
}

```
#### Store a User object:

```dart
final User user = User(id: '1', name: 'John Doe', email: 'john.doe@example.com');
await storage.create(key: 'user_1', value: user.toJson());
```

#### Retrieve a User object:

```dart
final String? userJson = await storage.read(key: 'user_1');
if (userJson != null) {
  final User user = User.fromJson(userJson);
  // Use the user object
}
```
This method ensures your data models are stored and retrieved in a type-safe manner, leveraging the power of JSON serialization and deserialization.

## Source code

### HiveStorageImpl Class
This class implements Storage interface using Hive as storage backend.It is used when `LocalStorageType.hive` is selected in the configuration.
```dart
import 'package:hive/hive.dart';

import '../../storage.dart';

class HiveStorageImpl implements Storage {
  final String name;

  Box? _box;

  HiveStorageImpl({required this.name});

  @override
  Future<void> init() async {
    _box = await Hive.openBox(name);
  }

  @override
  Future<void> create({required String key, required String value}) async {
    assert(_box != null, 'Box is null, not initiized.');

    _box!.put(key, value);
  }

  @override
  Future<void> createAll({required Map<String, String> values}) async {
    assert(_box != null, 'Box is null, not initiized.');

    for (String k in values.keys) {
      await _box!.put(k, values[k]);
    }
  }

  @override
  Future<String?> read({required String key}) async {
    assert(_box != null, 'Box is null, not initiized.');

    String? result = _box!.get(key);
    return result;
  }

  @override
  Future<Map<String, String?>> readAll({List<String> keys = const []}) async {
    assert(_box != null, 'Box is null, not initiized.');

    if (keys.isEmpty) {
      Map<String, String?> result =
          _box!.toMap().map((key, value) => MapEntry(key.toString(), value?.toString()));
      return result;
    } else {
      Map<String, String?> result = {};
      for (String k in keys) {
        if (_box!.containsKey(k)) {
          result[k] = _box!.get(k);
        }
      }
      return result;
    }
  }

  @override
  Future<void> delete({dynamic key}) async {
    assert(_box != null, 'Box is null, not initiized.');

    await _box!.delete(key);
  }

  @override
  Future<void> deleteAll({List<String> keys = const []}) async {
    assert(_box != null, 'Box is null, not initiized.');

    if (keys.isEmpty) {
      await _box!.clear();
    } else {
      _box!.deleteAll(keys);
    }
  }
}
```

### FlutterSecureStorageImpl Class
This class implements Storage interface using Flutter Secure Storage as storage backend.It is used when `LocalStorageType.flutterSecureStorage` is selected in the configuration.
```dart
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

import '../../storage.dart';

class FlutterSecureStorageImpl implements Storage {
  final _storage = const FlutterSecureStorage();

  @override
  Future<void> init() async {}

  @override
  Future<void> create({required String key, required String value}) async {
    await _storage.write(key: key, value: value);
  }

  @override
  Future<void> createAll({required Map<String, String> values}) async {
    for (String k in values.keys) {
      await _storage.write(key: k, value: values[k]);
    }
  }

  @override
  Future<String?> read({required String key}) async {
    String? result = await _storage.read(key: key);
    return result;
  }

  @override
  Future<Map<String, String?>> readAll({List<String> keys = const []}) async {
    if (keys.isEmpty) {
      final Map<String, String> result = await _storage.readAll();
      return result;
    } else {
      Map<String, String?> result = {};
      for (String k in keys) {
        result[k] = await _storage.read(key: k);
      }
      return result;
    }
  }

  @override
  Future<void> delete({required String key}) async {
    await _storage.delete(key: key);
  }

  @override
  Future<void> deleteAll({List<String> keys = const []}) async {
    if (keys.isEmpty) {
      await _storage.deleteAll();
    } else {
      for (String k in keys) {
        await _storage.delete(key: k);
      }
    }
  }
}
```

### NullStorage Class

The `NullStorage` class is a placeholder implementation of the `Storage` interface that throws `UnimplementedError` for all methods. It is used when `LocalStorageType.none` is selected in the configuration.

```dart
class NullStorage implements Storage {
  @override
  Future<void> init() async {}

  @override
  Future<void> create({required String key, required String value}) {
    throw UnimplementedError();
  }

  @override
  Future<void> createAll({required Map<String, String> values}) {
    throw UnimplementedError();
  }

  @override
  Future<String?> read({required String key}) {
    throw UnimplementedError();
  }

  @override
  Future<Map<String, String?>> readAll({List<String> keys = const []}) {
    throw UnimplementedError();
  }

  @override
  Future<void> delete({required String key}) {
    throw UnimplementedError();
  }

  @override
  Future<void> deleteAll({List<String> keys = const []}) {
    throw UnimplementedError();
  }
}
```