# Логирование запросов и ответов

Логирование запросов и ответов в RestAssured помогает в отладке API-тестов, предоставляя детальную информацию о взаимодействии с сервером. RestAssured поддерживает гибкое логирование через методы `log().all()`, `log().body()`, `log().headers()`, а также условное логирование при падении тестов. Интеграция с Allure позволяет добавлять в отчёты запросы в формате curl, JSON-ответы и заголовки. В этой статье описываются эти возможности, их настройка и лучшие практики.

## log().all(), log().body(), log().headers()

RestAssured предоставляет методы для логирования различных аспектов запросов и ответов:
- **`log().all()`**: Логирует полный запрос и ответ, включая метод, URL, заголовки, параметры и тело.
- **`log().body()`**: Логирует только тело запроса или ответа.
- **`log().headers()`**: Логирует только заголовки запроса или ответа.

### Пример логирования (JUnit 5)
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class LoggingTest {
    @Test
    void testFullLogging() {
        given()
            .contentType(ContentType.JSON)
            .log().all() // Логируем полный запрос
        .when()
            .get("https://api.example.com/v1/users/123")
        .then()
            .log().all() // Логируем полный ответ
            .statusCode(200)
            .body("name", equalTo("John Doe"));
    }

    @Test
    void testBodyLogging() {
        given()
            .contentType(ContentType.JSON)
            .body("{ \"name\": \"Jane Smith\", \"email\": \"jane@example.com\" }")
            .log().body() // Логируем только тело запроса
        .when()
            .post("https://api.example.com/v1/users")
        .then()
            .log().body() // Логируем только тело ответа
            .statusCode(201);
    }

    @Test
    void testHeadersLogging() {
        given()
            .contentType(ContentType.JSON)
            .header("Authorization", "Bearer token123")
            .log().headers() // Логируем только заголовки запроса
        .when()
            .get("https://api.example.com/v1/users/123")
        .then()
            .log().headers() // Логируем только заголовки ответа
            .statusCode(200);
    }
}
```

### Пример логирования (TestNG)
```java
import io.restassured.http.ContentType;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class LoggingTest {
    @Test
    public void testFullLogging() {
        given()
            .contentType(ContentType.JSON)
            .log().all()
        .when()
            .get("https://api.example.com/v1/users/123")
        .then()
            .log().all()
            .statusCode(200)
            .body("name", equalTo("John Doe"));
    }

    @Test
    public void testBodyLogging() {
        given()
            .contentType(ContentType.JSON)
            .body("{ \"name\": \"Jane Smith\", \"email\": \"jane@example.com\" }")
            .log().body()
        .when()
            .post("https://api.example.com/v1/users")
        .then()
            .log().body()
            .statusCode(201);
    }

    @Test
    public void testHeadersLogging() {
        given()
            .contentType(ContentType.JSON)
            .header("Authorization", "Bearer token123")
            .log().headers()
        .when()
            .get("https://api.example.com/v1/users/123")
        .then()
            .log().headers()
            .statusCode(200);
    }
}
```

**Примечания**:
- `log().all()` полезен для полной отладки, но может создавать большие логи.
- `log().body()` удобен для проверки JSON/XML-данных.
- `log().headers()` помогает анализировать заголовки, такие как `Content-Type` или `Authorization`.

## Включение логов только при падении

Метод `enableLoggingOfRequestAndResponseIfValidationFails()` позволяет включать логирование только в случае, если тест завершился с ошибкой. Это сокращает объём логов и упрощает анализ проблем.

### Пример условного логирования
```java
import io.restassured.RestAssured;
import io.restassured.http.ContentType;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;

public class ConditionalLoggingTest {
    @BeforeEach
    void setUp() {
        RestAssured.enableLoggingOfRequestAndResponseIfValidationFails();
    }

    @Test
    void testFailedRequest() {
        given()
            .contentType(ContentType.JSON)
            .body("{ \"name\": \"\", \"email\": \"invalid\" }")
        .when()
            .post("https://api.example.com/v1/users")
        .then()
            .statusCode(200); // Ожидаем 200, но получим 400, логи будут записаны
    }
}
```

**Примечания**:
- Логирование включается автоматически при любом сбое валидации (статус-код, тело, заголовки).
- Можно настроить уровень логирования (например, только заголовки):
```java
RestAssured.enableLoggingOfRequestAndResponseIfValidationFails(LogDetail.HEADERS);
```

## Интеграция с Allure: вложения curl, JSON, headers

Allure улучшает отчётность, добавляя вложения с информацией о запросах и ответах, включая curl-команды, JSON-данные и заголовки. Для этого используется фильтр `AllureRestAssured`.

### Настройка Allure
Добавьте зависимости в `pom.xml`:
```xml
<dependency>
    <groupId>io.qameta.allure</groupId>
    <artifactId>allure-rest-assured</artifactId>
    <version>2.24.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>io.qameta.allure</groupId>
    <artifactId>allure-junit5</artifactId>
    <version>2.24.0</version>
    <scope>test</scope>
</dependency>
```

### Пример интеграции с Allure (JUnit 5)
```java
import io.qameta.allure.restassured.AllureRestAssured;
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class AllureLoggingTest {
    @Test
    void testAllureLogging() {
        given()
            .filter(new AllureRestAssured()) // Включаем Allure-логирование
            .contentType(ContentType.JSON)
            .body("{ \"name\": \"Jane Smith\", \"email\": \"jane@example.com\" }")
        .when()
            .post("https://api.example.com/v1/users")
        .then()
            .statusCode(201)
            .body("email", equalTo("jane@example.com"));
    }
}
```

### Пример интеграции с Allure (TestNG)
```java
import io.qameta.allure.restassured.AllureRestAssured;
import io.restassured.http.ContentType;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class AllureLoggingTest {
    @Test
    public void testAllureLogging() {
        given()
            .filter(new AllureRestAssured())
            .contentType(ContentType.JSON)
            .body("{ \"name\": \"Jane Smith\", \"email\": \"jane@example.com\" }")
        .when()
            .post("https://api.example.com/v1/users")
        .then()
            .statusCode(201)
            .body("email", equalTo("jane@example.com"));
    }
}
```

**Примечания**:
- `AllureRestAssured` автоматически добавляет в отчёт Allure:
  - curl-команду для воспроизведения запроса.
  - JSON-тело запроса и ответа.
  - Заголовки запроса и ответа.
- Для настройки отчётов создайте файл `allure.properties`:
```properties
allure.results.directory=target/allure-results
```

## Связь с клиент-серверной моделью

Логирование запросов и ответов моделирует поведение клиента, который фиксирует взаимодействие с сервером. Это помогает анализировать, какие данные отправляются (запросы) и какие возвращаются (ответы). Интеграция с Allure делает эту информацию доступной в отчётах, упрощая анализ ошибок на стороне клиента или сервера.

## Лучшие практики и отладка

- **Стабильность**: Используйте `log().ifError()` или `enableLoggingOfRequestAndResponseIfValidationFails()` для минимизации объёма логов.
- **Логирование**: Настройте SLF4J с Logback для дополнительного контроля:
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

- **Отладка**: Используйте IntelliJ IDEA для просмотра логов в консоли или Allure-отчётов. Для проверки curl-команд копируйте их из Allure и тестируйте в терминале.
- **Распространённые ошибки**:
  - Перегрузка логов из-за `log().all()` (используйте условное логирование).
  - Отсутствие Allure-фильтра, из-за чего отчёты не содержат данных о запросах.
  - Неправильная настройка `allure-results` (проверяйте путь в `allure.properties`).

## Источники
- [RestAssured Official Documentation](http://rest-assured.io/)
- [Allure Framework Documentation](https://docs.qameta.io/allure/)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [TestNG Documentation](https://testng.org/doc/)
- [SLF4J Documentation](https://www.slf4j.org/)

---
[**← Назад к оглавлению**](../README.md)