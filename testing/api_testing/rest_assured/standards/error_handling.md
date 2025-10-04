# Обработка ошибок

Обработка ошибок в API-тестах важна для проверки поведения системы в негативных сценариях. RestAssured позволяет тестировать различные HTTP-статус-коды ошибок (400, 401, 403, 404, 500), проверять сообщения об ошибках и логировать детали для анализа. В этой статье рассматриваются подходы к тестированию ошибок, проверка их сообщений и логирование.

## Негативные сценарии: 400, 401, 403, 404, 500

Негативные тесты проверяют поведение API при некорректных запросах или сбоях. Основные статус-коды ошибок:
- **400 (Bad Request)**: Некорректный запрос (например, неверный формат данных).
- **401 (Unauthorized)**: Отсутствие или неверные учетные данные.
- **403 (Forbidden)**: Недостаточно прав для выполнения действия.
- **404 (Not Found)**: Запрашиваемый ресурс не существует.
- **500 (Internal Server Error)**: Внутренняя ошибка сервера.

### Пример тестирования негативных сценариев (JUnit 5)
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class ErrorHandlingTest {
    @Test
    void testBadRequest() {
        given()
            .contentType(ContentType.JSON)
            .body("{ \"name\": \"\", \"email\": \"invalid\" }") // Некорректные данные
        .when()
            .post("https://api.example.com/v1/users")
        .then()
            .statusCode(400)
            .body("error.message", equalTo("Invalid email format"));
    }

    @Test
    void testUnauthorized() {
        given()
            .contentType(ContentType.JSON)
        .when()
            .get("https://api.example.com/v1/users/123") // Без токена
        .then()
            .statusCode(401)
            .body("error.message", equalTo("Authorization token required"));
    }

    @Test
    void testForbidden() {
        given()
            .contentType(ContentType.JSON)
            .header("Authorization", "Bearer invalid_token")
        .when()
            .delete("https://api.example.com/v1/users/123")
        .then()
            .statusCode(403)
            .body("error.message", equalTo("Access denied"));
    }

    @Test
    void testNotFound() {
        given()
            .contentType(ContentType.JSON)
            .header("Authorization", "Bearer token123")
        .when()
            .get("https://api.example.com/v1/users/999") // Несуществующий ID
        .then()
            .statusCode(404)
            .body("error.message", equalTo("User not found"));
    }

    @Test
    void testServerError() {
        given()
            .contentType(ContentType.JSON)
            .header("Authorization", "Bearer token123")
        .when()
            .post("https://api.example.com/v1/server/crash") // Эндпоинт, вызывающий ошибку
        .then()
            .statusCode(500)
            .body("error.message", equalTo("Internal server error"));
    }
}
```

### Пример тестирования негативных сценариев (TestNG)
```java
import io.restassured.http.ContentType;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class ErrorHandlingTest {
    @Test
    public void testBadRequest() {
        given()
            .contentType(ContentType.JSON)
            .body("{ \"name\": \"\", \"email\": \"invalid\" }")
        .when()
            .post("https://api.example.com/v1/users")
        .then()
            .statusCode(400)
            .body("error.message", equalTo("Invalid email format"));
    }

    @Test
    public void testUnauthorized() {
        given()
            .contentType(ContentType.JSON)
        .when()
            .get("https://api.example.com/v1/users/123")
        .then()
            .statusCode(401)
            .body("error.message", equalTo("Authorization token required"));
    }

    @Test
    public void testForbidden() {
        given()
            .contentType(ContentType.JSON)
            .header("Authorization", "Bearer invalid_token")
        .when()
            .delete("https://api.example.com/v1/users/123")
        .then()
            .statusCode(403)
            .body("error.message", equalTo("Access denied"));
    }

    @Test
    public void testNotFound() {
        given()
            .contentType(ContentType.JSON)
            .header("Authorization", "Bearer token123")
        .when()
            .get("https://api.example.com/v1/users/999")
        .then()
            .statusCode(404)
            .body("error.message", equalTo("User not found"));
    }

    @Test
    public void testServerError() {
        given()
            .contentType(ContentType.JSON)
            .header("Authorization", "Bearer token123")
        .when()
            .post("https://api.example.com/v1/server/crash")
        .then()
            .statusCode(500)
            .body("error.message", equalTo("Internal server error"));
    }
}
```

## Проверка сообщений об ошибке

Сообщения об ошибках, возвращаемые в теле ответа, помогают понять причину сбоя. RestAssured позволяет извлекать и проверять их с помощью `body()` или `jsonPath()`.

### Пример проверки сообщения об ошибке
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;
import static org.assertj.core.api.Assertions.assertThat;

public class ErrorMessageTest {
    @Test
    void testErrorMessage() {
        String errorMessage = given()
            .contentType(ContentType.JSON)
            .body("{ \"name\": \"\", \"email\": \"invalid\" }")
        .when()
            .post("https://api.example.com/v1/users")
        .then()
            .statusCode(400)
            .extract().jsonPath().getString("error.message");

        assertThat(errorMessage).isEqualTo("Invalid email format");
    }
}
```

**Примечания**:
- Используйте точные JSON-пути для извлечения сообщений (например, `error.message`).
- Для сложных структур ошибок создавайте DTO для десериализации:
```java
import lombok.Data;

@Data
public class ErrorResponse {
    private String message;
    private String code;
}
```

```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.assertj.core.api.Assertions.assertThat;

public class ErrorDtoTest {
    @Test
    void testErrorDto() {
        ErrorResponse error = given()
            .contentType(ContentType.JSON)
            .body("{ \"name\": \"\", \"email\": \"invalid\" }")
        .when()
            .post("https://api.example.com/v1/users")
        .then()
            .statusCode(400)
            .extract().as(ErrorResponse.class);

        assertThat(error.getMessage()).isEqualTo("Invalid email format");
        assertThat(error.getCode()).isEqualTo("INVALID_EMAIL");
    }
}
```

## Логирование тела ошибки и причины

Логирование ошибок помогает в отладке и анализе тестов. RestAssured поддерживает логирование запросов и ответов, а для более детального логирования можно использовать SLF4J с Logback.

### Пример логирования ответа об ошибке
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;

public class ErrorLoggingTest {
    @Test
    void testLogErrorResponse() {
        given()
            .contentType(ContentType.JSON)
            .body("{ \"name\": \"\", \"email\": \"invalid\" }")
            .log().all() // Логирование запроса
        .when()
            .post("https://api.example.com/v1/users")
        .then()
            .log().ifError() // Логирование ответа только при ошибке
            .statusCode(400);
    }
}
```

### Настройка SLF4J с Logback
Добавьте зависимости в `pom.xml`:
```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>2.0.12</version>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.4.14</version>
    <scope>test</scope>
</dependency>
```

Создайте файл `logback-test.xml`:
```xml
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="info">
        <appender-ref ref="CONSOLE"/>
    </root>
</configuration>
```

### Пример логирования с SLF4J
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import static io.restassured.RestAssured.given;

public class Slf4jErrorLoggingTest {
    private static final Logger logger = LoggerFactory.getLogger(Slf4jErrorLoggingTest.class);

    @Test
    void testLogErrorWithSlf4j() {
        String response = given()
            .contentType(ContentType.JSON)
            .body("{ \"name\": \"\", \"email\": \"invalid\" }")
        .when()
            .post("https://api.example.com/v1/users")
        .then()
            .statusCode(400)
            .extract().asString();

        logger.error("Error response: {}", response);
    }
}
```

## Связь с клиент-серверной моделью

Обработка ошибок моделирует поведение клиента, который получает и анализирует ответы сервера в случае сбоев. Проверка статус-кодов и сообщений об ошибках позволяет убедиться, что сервер возвращает понятные и корректные данные, а логирование помогает клиенту (тесту) фиксировать причины для дальнейшего анализа.

## Лучшие практики и отладка

- **Стабильность**: Проверяйте сообщения об ошибках на основе документации API. Используйте стабильные JSON-пути (например, `error.message`).
- **Логирование**: Используйте `log().ifError()` для экономии места в логах, фиксируя только ошибки.
- **Отладка**: В IntelliJ IDEA установите точки останова для анализа тела ответа. Используйте JSONPath Finder для проверки путей в JSON.
- **Распространённые ошибки**:
  - Неправильные ожидания статус-кодов (проверяйте документацию API).
  - Неправильные JSON-пути для сообщений об ошибках.
  - Игнорирование логирования (всегда включайте логи для ошибок).

## Источники
- [RestAssured Official Documentation](http://rest-assured.io/)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [TestNG Documentation](https://testng.org/doc/)
- [SLF4J Documentation](https://www.slf4j.org/)
- [Logback Documentation](https://logback.qos.ch/)

---
[**← Назад к оглавлению**](../README.md)