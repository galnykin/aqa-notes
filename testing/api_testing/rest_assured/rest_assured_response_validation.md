# Проверка ответа API с помощью REST Assured: статус, тело, заголовки

REST Assured — это Java-библиотека, которая позволяет удобно тестировать REST API. Она использует цепочку вызовов `given().when().then()` для формирования запроса, его отправки и проверки ответа. Ниже — базовая структура и примеры проверки ключевых элементов ответа: статус-код, тело и заголовки.

---

## 🔧 Структура запроса в REST Assured

```java
given()
    .body("{\"username\":\"admin\",\"password\":\"1234\"}")
    .header("Content-Type", "application/json")
.when()
    .post("https://api.example.com/login")
.then()
    .log().all();
```

### `given()` — подготовка запроса
- `body(String)` — тело запроса в формате JSON, XML, текст.
- `header(String, String)` — заголовки запроса, например:
  ```java
  header("Content-Type", "application/json")
  header("Accept", "application/json")
  ```

### `when()` — отправка запроса
- `get(String)` — запрос на чтение.
- `post(String)` — запрос на создание.
- `put(String)` — обновление.
- `delete(String)` — удаление.

### `then()` — проверка ответа
- `log().all()` — вывод всех деталей ответа.
- `statusCode(int)` — проверка кода состояния.
- `body(String, Matcher)` — проверка содержимого.
- `header(String, Matcher)` — проверка заголовков.

---

## ✅ Примеры проверок

### Проверка статус-кода:
```java
.then()
    .statusCode(200);
```

### Проверка тела ответа:
```java
.then()
    .body("token", notNullValue())
    .body("user.name", equalTo("admin"));
```

### Проверка заголовков:
```java
.then()
    .header("Content-Type", containsString("application/json"));
```

---

## 📄 Форматы тела ответа

Ответ может быть:
- **JSON** — наиболее распространённый формат:
  ```json
  {
    "id": 1,
    "name": "Pizza",
    "available": true,
    "tags": ["cheese", "vegetarian"],
    "details": null
  }
  ```
- **HTML** — страница или фрагмент HTML.
- **Text** — простой текст.

REST Assured автоматически парсит JSON и позволяет обращаться к полям через путь (`user.name`, `token`, `tags[0]`).

---

## 🔗 Источники:
- [REST Assured Documentation](https://rest-assured.io/)
- [Hamcrest Matchers](http://hamcrest.org/JavaHamcrest/)
- [MDN HTTP Response Reference](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)

---
[**← Назад к оглавлению**](../../../README.md)
