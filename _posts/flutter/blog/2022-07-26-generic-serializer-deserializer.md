---
layout: post
title: Create a Generic Serializer and Deserializer in Dart
description: A long requested feature in Dart for accessing constuctors, methods or instance variables from the generic type is not available yet in Dart yet. Such feature would be helpful in many use cases and one of them is use it as a generic serializer and deserializer.
categories:
  - Dart
img: "assets/images/blog/Generic Deserialization.png"
dartpad: true
---

A long requested feature in Dart for accessing constuctors, methods or instance variables from the generic type `<T>` is not yet available in Dart. Although we can use static abstract methods for achieving similar behaviour, I wanted to simplify and make the code efficient for deserialization purpose only. Hence, I implemented a different and minimal approach by creating a helper method. For other use cases, the feature to access member from generic would be much helpful to reduce the boilerplate and increase code reusability. One of common use cases would to create a generic serializer and deserializer for the API response where every other fields like status, message etc. remains consistent and the details field is only different like in the example below.

The helper method we are going to create works for `fromJson` constructor whether it is manual created or generated from code generation library like `json_serializable`. It is completely type safe and allows static type checking too. It can be used for deserializing objects, lists of objects and complex JSON objects as well.

Let's get started!

Suppose we have following `Response` class with manual written `fromJson` decoder constructor.

```dart
class Response<T> {
  const Response({this.status, this.message, this.details});

  factory Response.fromJson(
    Map<String, dynamic> json,
    T Function(Map<String, dynamic> mapObj) decoder) =>
      Response(
        status: json['status'],
        message: json['message'],
        details: json['details'] == null
            ? null
            : deserialize<T>(json['details'], decoder)
      );

  final String? status;
  final String? message;
  final T? details;
}

```

In the above class, we can see that `details` field is of generic type `T`.

Let's consider two more classes `Person` and `Student` which will be the type for details `T` depending on the response.

```dart
class Person {
  const Person({this.name, this.age});

  factory Person.fromJson(Map<String, dynamic> json) =>
      Person(name: json['name'], age: int.tryParse(json['age']));

  final String? name;
  final int? age;
}
```

```dart
class Student {
  const Student({this.name, this.college});

  factory Student.fromJson(Map<String, dynamic> json) =>
      Student(name: json['name'], college: json['college']);

  final String? name;
  final String? college;
}
```

Looking at the above example, only `details` field is different in `Person` and `Student` class. In order to enable deserialization of `Person` and `Student` class directly from the `Response`, we have our helper method in the `fromJson` decoder which takes additional parameter type to determine which decoder to use. This allows `Response` class to be directly decoded for both `Person` and `Student` with the following helper method.

```dart
T deserialize<T>(
  Map<String, dynamic> map,
  T Function(Map<String, dynamic> mapObj) decoder) =>
    decoder(map);
```

Here, we are taking two parameters in the above method. `map` is simply a `Map<String, dynamic>` that we pass as arguement in the `fromJson` constructor. Second parameter `decoder` is a `typedef` for the `fromJson` constructor. `typedef` is an alias of the function which can be used like variable. Let's use this helper to decode the response with details for `Person` and `Student`.

Suppose we have following response object for `Person`.

```dart
{
  "status": "Success",
  "message": "The person is active.",
  "details": {"name": "Lucas", "age": 5}
}
```

We can decode this response like this.

```dart
Map<String, dynamic> personRes = {
  "status": "Success",
  "message": "The person is active.",
  "details": {"name": "Lucas", "age": 5}
};

Response personResponse = Response<Person>.fromJson(resP, Person.fromJson);
```

Similarly, let's assume following response object for `Student`.

```dart
{
  "status": "Success",
  "message": "The person is active.",
  "details": {"name": "Lucas", "age": 5}
}
```

We can decode the response object for `Student` as follow.

```dart
Map<String, dynamic> studentRes = {
  "status": "Success",
  "message": "The student is active.",
  "details": {"name": "Lucy", "college": "Oxford"}
};

Response studentResponse = Response<Student>.fromJson(resS, Student.fromJson);
```

In similar manner, we can create the helper method for list of objects too. Following helper method can be used for deserializing the `List<T>`.

```dart
List<T> deserializeList<T>(
  List<Map<String, dynamic>> maps,
  T Function(Map<String, dynamic> mapObj) decoder,
) =>
    List<T>.from(maps.map((map) => decoder(map)));
```

Here, the usage is exactly same as deserializing the object. The only difference is that it is used for generic lists in the `Response` decoder like following.

```dart
deserializeList<T>(json['details'], decoder)
```

Here is an interactive example of the above example.

```run-dartpad:theme-dark:mode-inline:run-true:null_safety-true:split-60:width-100%:height-500px:ga_id-visibility_widget_explained_with_example
void main() {
  Map<String, dynamic> personRes = {
    "status": "Success",
    "message": "The person is active.",
    "details": {"name": "Lucas", "age": 5}
  };

  Response personResponse =
      Response<Person>.fromJson(personRes, Person.fromJson);

  print({
    "Type": "Person",
    "Name": personResponse.details.name,
    "Age": personResponse.details.age,
  });

  Map<String, dynamic> studentRes = {
    "status": "Success",
    "message": "The student is active.",
    "details": {"name": "Lucy", "college": "Oxford"}
  };

  Response studentResponse =
      Response<Student>.fromJson(studentRes, Student.fromJson);

  print({
    "Type": "Student",
    "Name": studentResponse.details.name,
    "Age": studentResponse.details.college,
  });
}

T deserialize<T>(Map<String, dynamic> map,
        T Function(Map<String, dynamic> mapObj) decoder) =>
    decoder(map);

List<T> deserializeList<T>(
  List<Map<String, dynamic>> maps,
  T Function(Map<String, dynamic> mapObj) decoder,
) =>
    List<T>.from(maps.map((map) => decoder(map)));

class Response<T> {
  const Response({this.status, this.message, this.details});

  factory Response.fromJson(Map<String, dynamic> json,
          T Function(Map<String, dynamic> mapObj) decoder) =>
      Response(
          status: json['status'],
          message: json['message'],
          details: json['details'] == null
              ? null
              : deserialize<T>(json['details'], decoder));

  final String? status;
  final String? message;
  final T? details;
}

class Person {
  const Person({this.name, this.age});

  factory Person.fromJson(Map<String, dynamic> json) =>
      Person(name: json['name'], age: json['age']);

  final String? name;
  final int? age;
}

class Student {
  const Student({this.name, this.college});

  factory Student.fromJson(Map<String, dynamic> json) =>
      Student(name: json['name'], college: json['college']);

  final String? name;
  final String? college;
}
```
