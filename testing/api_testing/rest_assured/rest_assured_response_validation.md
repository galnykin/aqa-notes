# Проверка ответа API с помощью REST Assured: статус, тело, заголовки

REST Assured — это Java-библиотека, которая позволяет удобно тестировать REST API. Она использует цепочку вызовов `given().when().then()` для формирования запроса, его отправки и проверки ответа. Ниже — расширенная структура и примеры проверки ключевых элементов ответа: статус-код, тело и заголовки, а также работа с временем ответа, негативные проверки и валидация схемы.

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

* `body(String)` — тело запроса в формате JSON, XML, текст
* `header(String, String)` — заголовки запроса
* `queryParam(String, Object)` — query-параметры
* `pathParam(String, Object)` — path-параметры
* `auth().basic()` / `auth().oauth2()` — авторизация

### `when()` — отправка запроса

* `get(String)` — чтение данных
* `post(String)` — создание ресурса
* `put(String)` — полное обновление
* `patch(String)` — частичное обновление
* `delete(String)` — удаление

### `then()` — проверка ответа

* `statusCode(int)` — проверка кода состояния
* `body(String, Matcher)` — проверка содержимого
* `header(String, Matcher)` — проверка заголовков
* `time(lessThan())` — проверка времени ответа
* `log().all()` — вывод всего ответа

---

## ✅ Примеры проверок

### Проверка статус-кода

```java
.then()
    .statusCode(200);
```

### Проверка тела ответа (JSON)

```java
.then()
    .body("token", notNullValue())
    .body("user.name", equalTo("admin"))
    .body("roles", hasItems("USER", "ADMIN"));
```

### Проверка заголовков

```java
.then()
    .header("Content-Type", containsString("application/json"))
    .header("Cache-Control", equalTo("no-cache"));
```

### Проверка времени ответа

```java
.then()
    .time(lessThan(2000L)); // не дольше 2 секунд
```

---

## ❌ Негативные проверки

Важно тестировать ошибки и граничные случаи:

### Ошибка авторизации

```java
given()
    .header("Content-Type", "application/json")
    .body("{\"username\":\"wrong\",\"password\":\"bad\"}")
.when()
    .post("/login")
.then()
    .statusCode(401)
    .body("error", equalTo("Unauthorized"));
```

### Отсутствие ресурса

```java
.when()
    .get("/users/9999")
.then()
    .statusCode(404)
    .body("message", containsString("not found"));
```

---

## 📄 Форматы тела ответа

REST Assured поддерживает разные форматы:

* **JSON** (наиболее распространённый):

  ```json
  {
    "id": 1,
    "name": "Pizza",
    "available": true,
    "tags": ["cheese", "vegetarian"],
    "details": null
  }
  ```
* **XML** (например, в SOAP или старых API):

  ```xml
  <product>
    <id>1</id>
    <name>Pizza</name>
  </product>
  ```
* **Text / HTML** — можно проверять через `.body(containsString("..."))`

---

## 🧩 Валидация схемы JSON

Для проверки структуры ответа используется JSON Schema Validation:

```java
import static io.restassured.module.jsv.JsonSchemaValidator.*;

.then()
    .body(matchesJsonSchemaInClasspath("schemas/userSchema.json"));
```

Это помогает контролировать не только данные, но и их формат.

---

## 🧪 Интеграция с TestNG

Пример автотеста на TestNG:

```java
import org.testng.annotations.Test;
import static io.restassured.RestAssured.*;
import static org.hamcrest.Matchers.*;

public class LoginApiTest {

    @Test
    public void loginShouldReturnToken() {
        given()
            .header("Content-Type", "application/json")
            .body("{\"username\":\"admin\",\"password\":\"1234\"}")
        .when()
            .post("https://api.example.com/login")
        .then()
            .statusCode(200)
            .body("token", notNullValue());
    }
}
```

---

## 🔗 Источники

* [REST Assured Documentation](https://rest-assured.io/)
* [Hamcrest Matchers](http://hamcrest.org/JavaHamcrest/)
* [MDN HTTP Response Reference](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
* [TestNG Documentation](https://testng.org/doc/)

---

[**← Назад к оглавлению**](README.md)
