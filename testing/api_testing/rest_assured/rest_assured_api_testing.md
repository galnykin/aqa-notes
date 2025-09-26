# Автоматизация тестирования API с помощью REST Assured: структура, синтаксис, практика

**REST Assured** — это Java-библиотека, предназначенная для автоматизации тестирования REST API. Она позволяет формировать HTTP-запросы, отправлять их, получать ответы и выполнять проверки в удобном и читаемом формате. REST Assured особенно популярен среди Java-разработчиков и тестировщиков благодаря своей лаконичности и интеграции с JUnit/TestNG.

---

## 📘 REST Assured: определения и понятия

- **REST API** — архитектурный стиль взаимодействия между клиентом и сервером через HTTP.
- **REST Assured** — библиотека, реализующая fluent-интерфейс для тестирования REST API.
- **Запрос** — формируется с помощью `given()`.
- **Отправка** — выполняется через `when()`.
- **Ответ** — обрабатывается через `then()`.
- **Проверки** — реализуются через `assertThat()` и методы из `Hamcrest`.

---

## 🧱 Формирование запроса: `given()`

Метод `given()` используется для подготовки запроса:
- заголовки;
- параметры;
- тело;
- авторизация.

Пример:
```java
given()
    .baseUri("https://api.example.com")
    .header("Content-Type", "application/json")
    .body("{\"username\":\"admin\",\"password\":\"1234\"}")
```

Можно добавлять query-параметры:
```java
given()
    .queryParam("category", "pizza")
```

---

## 📡 Отправка запроса: `when()`

Метод `when()` указывает, какой HTTP-метод использовать:
```java
.when()
    .post("/login")
```

Другие методы:
- `.get("/users")`
- `.put("/users/123")`
- `.delete("/users/123")`
- `.patch("/users/123")`

---

## 📥 Получение ответа: `then()`

Метод `then()` позволяет обработать и проверить ответ:
```java
.then()
    .statusCode(200)
    .log().body();
```

Можно логировать:
- `.log().all()` — всё;
- `.log().headers()` — только заголовки;
- `.log().body()` — только тело.

---

## ✅ Проверки: `assertThat()`

Проверки выполняются с помощью `assertThat()` и матчеров из `Hamcrest`:
```java
.then()
    .assertThat()
    .body("token", notNullValue())
    .body("user.name", equalTo("admin"))
    .body("roles", hasItem("ADMIN"));
```

Можно проверять:
- наличие поля;
- значение поля;
- структуру массива;
- длину, тип, пустоту.

---

## 🔗 Источники:
- [REST Assured Documentation](https://rest-assured.io/)
- [Hamcrest Matchers](http://hamcrest.org/JavaHamcrest/)
- [MDN HTTP Overview](https://developer.mozilla.org/en-US/docs/Web/HTTP)
- [Postman vs REST Assured Comparison](https://rest-assured.io/#comparison)

---
[**← Назад к оглавлению**](../../../README.md)
