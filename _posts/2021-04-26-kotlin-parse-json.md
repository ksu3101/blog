---
title: How to parse JSON in Android using Kotlin
author: 강성우
layout: post
category: [Kotlin]
tag: [kotlin, json]
---

> 이 글은 John Codeos의 [How to parse JSON in Android using Kotlin](https://johncodeos.com/how-to-parse-json-in-android-using-kotlin/)을 번역 하였다. 

이 글에서는 외부 라이브러리를 사용하지 않고 Java클래스인 `JSONTokener`를 이용하여 JSON을 파싱 하는 방법에 대해서 소개 하려 한다. 

`JSONTokener`를 사용하면 JSON문자열을 객체화할 수 있게 파싱해준다. 이러한 객체는 `JSONObject`또는 `JSONArray`로 타입 캐스팅하여 사용할 수 있다. 이렇게 하면 JSON의 값들을 읽어 사용할 수 있게 된다. 

이 글에서는 JSON을 파싱하는 방법 3가지 예제들을 다루려 한다. 

# Simple JSON

간단한 JSON에는 배열이나 중첩된 JSON객체등과 같은 복잡한 구조가 아니며 다음과 같이 간단하다.

```json
{
    "id": "1",
    "employee_name": "Jack Full",
    "employee_salary": "300800",
    "employee_age": "61"
}
```

HTTP요청을 통해 응답을 받은 후 JSON문자열을 `JSONObject`로 파싱하여 JSON의 객체를 만드려 한다. 

```kotlin
val jsonObject = JSONTokener(response).nextValue() as JSONObject

// ID
val id = jsonObject.getString("id")
Log.i("ID: ", id)

// Employee Name
val employeeName = jsonObject.getString("employee_name")
Log.i("Employee Name: ", employeeName)

// Employee Salary
val employeeSalary = jsonObject.getString("employee_salary")
Log.i("Employee Salary: ", employeeSalary)

// Employee Age
val employeeAge = jsonObject.getString("employee_age")
Log.i("Employee Age: ", employeeAge)
```

# Array JSON

가끔 JSON이 배열로 구성되어 있으며 키가 없을때가 있다. 

```json
[
    {
        "id": "1",
        "employee_name": "Tiger Nixon",
        "employee_salary": "320800",
        "employee_age": "61"
    },
    {
        "id": "2",
        "employee_name": "Garrett Winters",
        "employee_salary": "170750",
        "employee_age": "63"
    },
    // ...
]
```

응답을 받은 후 JSON문자열을 `JSONArray`로 파싱하고 이 배열내부 원소를 JSON으로 반복하여 객체를 얻는다. 

```kotlin
val jsonArray = JSONTokener(response).nextValue() as JSONArray
for (i in 0 until jsonArray.length()) {
    // ID
    val id = jsonArray.getJSONObject(i).getString("id")
    Log.i("ID: ", id)

    // Employee Name
    val employeeName = jsonArray.getJSONObject(i).getString("employee_name")
    Log.i("Employee Name: ", employeeName)

    // Employee Salary
    val employeeSalary = jsonArray.getJSONObject(i).getString("employee_salary")
    Log.i("Employee Salary: ", employeeSalary)

    // Employee Age
    val employeeAge = jsonArray.getJSONObject(i).getString("employee_age")
    Log.i("Employee Age: ", employeeAge)

    // Save data using your Model

    // Notify the adapter
}

// Pass adapter to the RecyclerView adapter
```

> 팁 : 일부 JSON객체에 키가 없을 경우 다음과 같이 할 수도 있다. `jsonArray.getJSONObjet(i).optString("id)`

# Nested JSON

JSON객체가 다른 JSON객체 안에 있는 경우 이를 'Nested'(중첩)이라고 하며 다음 JSON구조와 비슷할 것 이다. 

```json
{
    "data": [
        {
            "id": "1",
            "employee": {
                "name": "Tiger Nixon",
                "salary": {
                    "usd": 320800,
                    "eur": 273545
                },
                "age": "61"
            }
        },
        {
            "id": "2",
            "employee": {
                "name": "Garrett Winters",
                "salary": {
                    "usd": 170750,
                    "eur": 145598
                },
                "age": "63"
            }
        },
        // ...
    ]
}
```

응답을 받은 후 JSON문자열을 `JSONObject`로 파싱한 다음 `data`를 `JSONArray`로 파싱한다. 

```kotlin
val jsonObject = JSONTokener(response).nextValue() as JSONObject

val jsonArray = jsonObject.getJSONArray("data")

for (i in 0 until jsonArray.length()) {

    // ID
    val id = jsonArray.getJSONObject(i).getString("id")
    Log.i("ID: ", id)

    // Employee
    val employee = jsonArray.getJSONObject(i).getJSONObject("employee")

    // Employee Name
    val employeeName = employee.getString("name")
    Log.i("Employee Name: ", employeeName)

    // Employee Salary
    val employeeSalary = employee.getJSONObject("salary")

    // Employee Salary in USD
    val employeeSalaryUSD = employeeSalary.getInt("usd")
    Log.i("Employee Salary in USD: ", employeeSalaryUSD.toString())

    // Employee Salary in EUR
    val employeeSalaryEUR = employeeSalary.getInt("eur")
    Log.i("Employee Salary: ", employeeSalaryEUR.toString())

    // Employee Age
    val employeeAge = employee.getString("age")
    Log.i("Employee Age: ", employeeAge)

    // Save data using your Model

    // Notify the adapter
}

// Pass adapter to the RecyclerView adapter
```
