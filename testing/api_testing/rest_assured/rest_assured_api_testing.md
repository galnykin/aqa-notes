# Автоматизация тестирования API с помощью REST Assured: структура, синтаксис, практика

**REST Assured** — это Java-библиотека, предназначенная для автоматизации тестирования REST API. Она позволяет формировать HTTP-запросы, отправлять их, получать ответы и выполнять проверки в удобном и читаемом формате. REST Assured особенно популярен среди Java-разработчиков и тестировщиков благодаря своей лаконичности и интеграции с JUnit/TestNG.

---

## 📘 REST Assured: определения и понятия

* **REST API** — архитектурный стиль взаимодействия между клиентом и сервером через HTTP
* **REST Assured** — библиотека, реализующая fluent-интерфейс для тестирования REST API
* **Запрос** — формируется с помощью `given()`
* **Отправка** — выполняется через `when()`
* **Ответ** — обрабатывается через `then()`
* **Проверки** — реализуются через `assertThat()` и методы из `Hamcrest`

---

## 🧱 Формирование запроса: `given()`

Метод `given()` используется для подготовки запроса:

* заголовки
* параметры
* тело
* авторизация

Пример:

```java
given()
    .baseUri("https://api.example.com")
    .header("Content-Type", "application/json")
    .body("{\"username\":\"admin\",\"password\":\"1234\"}");
```

Query-параметры:

```java
given()
    .queryParam("category", "pizza");
```

Path-параметры:

```java
given()
    .pathParam("id", 123)
.when()
    .get("/users/{id}");
```

---

## 📡 Отправка запроса: `when()`

Метод `when()` определяет HTTP-метод:

```java
.when()
    .post("/login");
```

Другие методы:

* `.get("/users")`
* `.put("/users/123")`
* `.delete("/users/123")`
* `.patch("/users/123")`

---

## 📥 Получение ответа: `then()`

Метод `then()` позволяет обработать и проверить ответ:

```java
.then()
    .statusCode(200)
    .log().body();
```

Варианты логирования:

* `.log().all()` — всё
* `.log().headers()` — только заголовки
* `.log().body()` — только тело

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

* наличие поля
* значение поля
* массивы и коллекции
* длину и пустоту

---

## 🧪 Интеграция с JUnit и TestNG

### JUnit 5

```java
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.*;
import static org.hamcrest.Matchers.*;

public class ApiTest {

    @Test
    void loginShouldReturnToken() {
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

### TestNG

```java
import org.testng.annotations.Test;
import static io.restassured.RestAssured.*;
import static org.hamcrest.Matchers.*;

public class ApiTestNg {

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

## 🛠 Дополнительные возможности REST Assured

* Авторизация:

  ```java
  given()
      .auth().basic("user", "pass");
  ```

* JSON Schema Validation:

  ```java
  .then()
      .body(matchesJsonSchemaInClasspath("userSchema.json"));
  ```

* Сериализация и десериализация (POJO):

  ```java
  User user = given()
      .get("/users/1")
      .as(User.class);
  ```

---

## 🔗 Источники

* [REST Assured Documentation](https://rest-assured.io/)
* [Hamcrest Matchers](http://hamcrest.org/JavaHamcrest/)
* [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
* [TestNG Documentation](https://testng.org/doc/)

---

[**← Назад к оглавлению**](README.md)

