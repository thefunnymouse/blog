---
layout: post
title:  "[Java] JSONObject"
date:   2019-03-23 15:23:00 +0700
categories: java
---

Để viết chuỗi Json trong java, ta có thể viết được dưới dạng:
```java
String json = "{\"name\": \"abc\", \"age\": 21}";
```
Tuy nhiên cách viết này khó nhìn, do đó ta dùng JSONObject để quản lý chuỗi json.

### Import dependency
```xml
<dependency>
    <groupId>org.json</groupId>
    <artifactId>json</artifactId>
    <version>20180813</version>
</dependency>
```

### Encode Json
#### Sử dụng hàm `put(key, value)`
![Alt](/blog/assets/images/201903/java-json-object.png)

Để lấy chuỗi Json, ta sử dụng hàm `toString()`
```java
JSONObject jsonObject = new JSONObject();
jsonObject.put("name", "abc");
jsonObject.put("age", 21);

System.out.println(jsonObject.toString());
```

#### Sử dụng hàm xây dựng `JSONObject(Map)`
JSONObject có thể nhận đầu vào là 1 map. Bởi vì map cũng là tập hợp các `key => value`
```java
Map<String, Object> map = new HashMap<>();
map.put("name", "abc");
map.put("age", 21);
JSONObject jsonObject = new JSONObject(map);
System.out.println(jsonObject.toString());
```