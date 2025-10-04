# Проверка ответов

Проверка ответов в API-тестах с использованием RestAssured позволяет убедиться в корректности работы сервера. Это включает проверку статус-кодов, тела ответа, заголовков, времени ответа и схем. В этой статье рассматриваются методы проверки с использованием `statusCode()`, `body()`, а также валидация заголовков и производительности.

## Проверка статус-кодов

Метод `statusCode()` в RestAssured используется для проверки HTTP-статус-кода ответа. Это базовая проверка, которая подтверждает, что сервер обработал запрос ожидаемым образом.

### Пример проверки статус-кода (JUnit 5)
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;

public class StatusCodeTest {
    @Test
    void testStatusCode() {
        given()
            .contentType(ContentType.JSON)
        .when()
            .get("https://api.example.com/v1/users/123")
        .then()
            .statusCode(200);
    }
}
```

### Пример проверки статус-кода (TestNG)
```java
import io.restassured.http.ContentType;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;

public class StatusCodeTest {
    @Test
    public void testStatusCode() {
        given()
            .contentType(ContentType.JSON)
        .when()
            .get("https://api.example.com/v1/users/123")
        .then()
            .statusCode(200);
    }
}
```

**Примечания**:
- Используйте стандартные статус-коды: `200` (OK), `201` (Created), `400` (Bad Request), `404` (Not Found), `500` (Server Error).
- Для сложных сценариев проверяйте диапазон кодов (например, `2xx`):
```java
.then().statusCode(greaterThanOrEqualTo(200)).statusCode(lessThan(300));
```

## Проверка тела ответа

Метод `body()` используется для проверки содержимого ответа, чаще всего в формате JSON, с применением Hamcrest-матчеров. Это позволяет проверять конкретные значения или списки в теле ответа.

### Пример проверки тела (JUnit 5)
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;
import static org.hamcrest.Matchers.hasItems;

public class BodyValidationTest {
    @Test
    void testBodyValidation() {
        given()
            .contentType(ContentType.JSON)
        .when()
            .get("https://api.example.com/v1/users")
        .then()
            .statusCode(200)
            .body("data.id", equalTo(123))
            .body("data.users.name", hasItems("John Doe", "Jane Smith"));
    }
}
```

### Пример проверки тела (TestNG)
```java
import io.restassured.http.ContentType;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;
import static org.hamcrest.Matchers.hasItems;

public class BodyValidationTest {
    @Test
    public void testBodyValidation() {
        given()
            .contentType(ContentType.JSON)
        .when()
            .get("https://api.example.com/v1/users")
        .then()
            .statusCode(200)
            .body("data.id", equalTo(123))
            .body("data.users.name", hasItems("John Doe", "Jane Smith"));
    }
}
```

### Использование jsonPath для сложных проверок
Для более сложных проверок можно извлечь данные с помощью `jsonPath()` и использовать AssertJ:
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.assertj.core.api.Assertions.assertThat;

public class JsonPathTest {
    @Test
    void testJsonPathValidation() {
        String id = given()
            .contentType(ContentType.JSON)
        .when()
            .get("https://api.example.com/v1/users/123")
        .then()
            .statusCode(200)
            .extract().jsonPath().getString("data.id");

        assertThat(id).isEqualTo("123");
    }
}
```

## Проверка заголовков, времени ответа, схем

### Проверка заголовков
Метод `header()` позволяет проверять HTTP-заголовки ответа, такие как `Content-Type` или пользовательские заголовки.

#### Пример проверки заголовков
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class HeaderValidationTest {
    @Test
    void testHeaderValidation() {
        given()
            .contentType(ContentType.JSON)
        .when()
            .get("https://api.example.com/v1/users/123")
        .then()
            .statusCode(200)
            .header("Content-Type", equalTo("application/json"))
            .header("X-Request-Id", notNullValue());
    }
}
```

### Проверка времени ответа
Время ответа проверяется с помощью метода `time()` для оценки производительности API.

#### Пример проверки времени ответа
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.lessThan;

public class ResponseTimeTest {
    @Test
    void testResponseTime() {
        given()
            .contentType(ContentType.JSON)
        .when()
            .get("https://api.example.com/v1/users/123")
        .then()
            .statusCode(200)
            .time(lessThan(2000L)); // Время ответа < 2 секунд
    }
}
```

### Проверка JSON-схем
`JsonSchemaValidator` используется для проверки структуры JSON-ответа. Это гарантирует, что ответ соответствует ожидаемой схеме.

#### Настройка зависимости
Добавьте в `pom.xml`:
```xml
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>json-schema-validator</artifactId>
    <version>5.4.0</version>
    <scope>test</scope>
</dependency>
```

#### Пример JSON-схемы (user-schema.json)
Создайте файл `src/test/resources/user-schema.json`:
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "data": {
      "type": "object",
      "properties": {
        "id": { "type": "integer" },
        "name": { "type": "string" },
        "email": { "type": "string" }
      },
      "required": ["id", "name", "email"]
    }
  },
  "required": ["data"]
}
```

#### Пример проверки схемы
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static io.restassured.module.jsv.JsonSchemaValidator.matchesJsonSchemaInClasspath;

public class SchemaValidationTest {
    @Test
    void testJsonSchema() {
        given()
            .contentType(ContentType.JSON)
        .when()
            .get("https://api.example.com/v1/users/123")
        .then()
            .statusCode(200)
            .body(matchesJsonSchemaInClasspath("user-schema.json"));
    }
}
```

## Связь с клиент-серверной моделью

Проверка ответов моделирует поведение клиента, который анализирует данные, возвращаемые сервером. Проверка статус-кодов подтверждает успешность запроса, проверка тела ответа и схем гарантирует корректность данных, а проверка заголовков и времени ответа обеспечивает соблюдение протоколов и производительности.

## Лучшие практики и отладка

- **Стабильность**: Используйте стабильные JSON-пути (например, `data.id` вместо индексов для массивов). Проверяйте обязательные заголовки, такие как `Content-Type`.
- **Логирование**: Включите логирование ответов с помощью `log().all()`:
```java
given()
    .log().all()
.when()
    .get("/users/123")
.then()
    .log().all()
    .statusCode(200);
```

- **Отладка**: Используйте точки останова в IntelliJ IDEA для проверки JSON-путей или заголовков. Для схем применяйте онлайн-валидаторы JSON Schema.
- **Распространённые ошибки**:
  - Неправильные JSON-пути (используйте JSONPath Finder).
  - Ошибки в схемах (например, неверный тип данных).
  - Слишком строгие проверки времени ответа (учитывайте сетевые задержки).

## Источники
- [RestAssured Official Documentation](http://rest-assured.io/)
- [JSON Schema Validator Documentation](https://github.com/rest-assured/rest-assured/wiki/Usage#json-schema-validation)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [TestNG Documentation](https://testng.org/doc/)

---
[**← Назад к оглавлению**](../README.md)