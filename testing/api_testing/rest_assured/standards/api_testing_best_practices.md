# Best Practices для API-тестов

Применение лучших практик в API-тестировании с использованием RestAssured повышает читаемость, стабильность и поддерживаемость тестов. В этой статье рассматриваются ключевые принципы: один тест — одна логическая проверка, приоритет читаемости, избегание `Thread.sleep`, централизация конфигурации и минимизация дублирования.

## Один тест — одна логическая проверка

Каждый тест должен проверять ровно одну функциональность или сценарий. Это упрощает отладку и делает тесты независимыми.

### Пример (JUnit 5)
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class SingleResponsibilityTest {
    @Test
    void testCreateUser() { // Проверяет только создание пользователя
        given()
            .contentType(ContentType.JSON)
            .body("{ \"name\": \"Jane Smith\", \"email\": \"jane@example.com\" }")
        .when()
            .post("https://api.example.com/v1/users")
        .then()
            .statusCode(201)
            .body("email", equalTo("jane@example.com"));
    }

    @Test
    void testGetUser() { // Проверяет только получение пользователя
        given()
            .contentType(ContentType.JSON)
        .when()
            .get("https://api.example.com/v1/users/123")
        .then()
            .statusCode(200)
            .body("name", equalTo("Jane Smith"));
    }
}
```

### Пример (TestNG)
```java
import io.restassured.http.ContentType;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class SingleResponsibilityTest {
    @Test
    public void testCreateUser() {
        given()
            .contentType(ContentType.JSON)
            .body("{ \"name\": \"Jane Smith\", \"email\": \"jane@example.com\" }")
        .when()
            .post("https://api.example.com/v1/users")
        .then()
            .statusCode(201)
            .body("email", equalTo("jane@example.com"));
    }

    @Test
    public void testGetUser() {
        given()
            .contentType(ContentType.JSON)
        .when()
            .get("https://api.example.com/v1/users/123")
        .then()
            .statusCode(200)
            .body("name", equalTo("Jane Smith"));
    }
}
```

**Почему это важно**:
- Упрощает поиск причины сбоя.
- Делает тесты независимыми, что критично для параллельного запуска.
- Улучшает читаемость и поддержку.

## Читаемость важнее компактности

Читаемые тесты легче поддерживать и понимать. Используйте понятные имена тестов, структурируйте код с помощью `given-when-then` и добавляйте комментарии при необходимости.

### Пример читаемого теста
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class ReadableTest {
    @Test
    void shouldCreateUserWithValidData() {
        // Подготовка данных для создания пользователя
        String requestBody = "{ \"name\": \"Jane Smith\", \"email\": \"jane@example.com\" }";

        given()
            .contentType(ContentType.JSON)
            .body(requestBody)
            .log().all() // Логируем запрос для отладки
        .when()
            .post("https://api.example.com/v1/users")
        .then()
            .statusCode(201)
            .body("email", equalTo("jane@example.com"))
            .log().ifError(); // Логируем только при ошибке
    }
}
```

**Почему это важно**:
- Понятные имена тестов (например, `shouldCreateUserWithValidData`) описывают цель теста.
- Структурированный код улучшает восприятие.
- Логирование делает отладку проще.

## Не использовать Thread.sleep, только Awaitility, retry или polling

`Thread.sleep` приводит к нестабильным тестам из-за фиксированных задержек. Вместо этого используйте библиотеку Awaitility для асинхронных проверок или механизмы retry/polling.

### Настройка Awaitility
Добавьте зависимость в `pom.xml`:
```xml
<dependency>
    <groupId>org.awaitility</groupId>
    <artifactId>awaitility</artifactId>
    <version>4.2.1</version>
    <scope>test</scope>
</dependency>
```

### Пример с Awaitility
```java
import io.restassured.http.ContentType;
import org.awaitility.Awaitility;
import org.junit.jupiter.api.Test;
import java.time.Duration;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class AwaitilityTest {
    @Test
    void testAsyncResponse() {
        given()
            .contentType(ContentType.JSON)
            .body("{ \"name\": \"Jane Smith\", \"email\": \"jane@example.com\" }")
        .when()
            .post("https://api.example.com/v1/users/async")
        .then()
            .statusCode(202); // Запрос принят, но обработка асинхронная

        Awaitility.await()
            .atMost(Duration.ofSeconds(10))
            .pollInterval(Duration.ofSeconds(1))
            .until(() -> 
                given()
                    .contentType(ContentType.JSON)
                .when()
                    .get("https://api.example.com/v1/users/123")
                .then()
                    .extract().statusCode() == 200
            );

        given()
            .contentType(ContentType.JSON)
        .when()
            .get("https://api.example.com/v1/users/123")
        .then()
            .statusCode(200)
            .body("name", equalTo("Jane Smith"));
    }
}
```

**Почему это важно**:
- Awaitility адаптируется к реальному времени обработки, избегая лишних задержек.
- Retry/polling позволяет повторять запросы до получения ожидаемого результата.
- Уменьшает ложные сбои из-за асинхронных операций.

## Централизация конфигурации и спецификаций

Централизация настроек через `RequestSpecification` и `ResponseSpecification` уменьшает дублирование и упрощает обновление конфигурации.

### Пример централизации
```java
import io.restassured.builder.RequestSpecBuilder;
import io.restassured.builder.ResponseSpecBuilder;
import io.restassured.http.ContentType;
import io.restassured.specification.RequestSpecification;
import io.restassured.specification.ResponseSpecification;

public class ApiConfig {
    public static RequestSpecification defaultRequestSpec() {
        return new RequestSpecBuilder()
                .setBaseUri("https://api.example.com")
                .setBasePath("/v1")
                .setContentType(ContentType.JSON)
                .addHeader("Authorization", "Bearer token123")
                .log().all()
                .build();
    }

    public static ResponseSpecification defaultResponseSpec() {
        return new ResponseSpecBuilder()
                .expectStatusCode(200)
                .expectContentType(ContentType.JSON)
                .build();
    }
}
```

### Использование в тесте
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class CentralizedConfigTest {
    @Test
    void testCreateUser() {
        given()
            .spec(ApiConfig.defaultRequestSpec())
            .body("{ \"name\": \"Jane Smith\", \"email\": \"jane@example.com\" }")
        .when()
            .post("/users")
        .then()
            .spec(ApiConfig.defaultResponseSpec())
            .body("email", equalTo("jane@example.com"));
    }
}
```

**Почему это важно**:
- Изменения в конфигурации вносятся в одном месте.
- Унифицирует настройки для всех тестов.
- Упрощает поддержку при изменении API.

## Минимизация дублирования и жёстко заданных значений

Дублирование кода и хардкод значений усложняют поддержку тестов. Используйте фабрики данных, шаблоны запросов и внешние файлы (JSON/YAML) для управления данными.

### Пример фабрики данных
```java
import lombok.Data;
import java.util.UUID;

@Data
public class UserDto {
    private String id;
    private String name;
    private String email;
}

public class UserFactory {
    public static UserDto createValidUser() {
        UserDto user = new UserDto();
        user.setId(UUID.randomUUID().toString());
        user.setName("Jane Smith");
        user.setEmail("jane_" + UUID.randomUUID() + "@example.com");
        return user;
    }
}
```

### Пример теста с фабрикой
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class NoHardcodingTest {
    @Test
    void testCreateUser() {
        UserDto user = UserFactory.createValidUser();

        given()
            .contentType(ContentType.JSON)
            .body(user)
        .when()
            .post("https://api.example.com/v1/users")
        .then()
            .statusCode(201)
            .body("email", equalTo(user.getEmail()));
    }
}
```

### Хранение данных в JSON
Создайте файл `test-data/users.json`:
```json
{
  "validUser": {
    "name": "Jane Smith",
    "email": "jane@example.com"
  }
}
```

#### Чтение JSON
```java
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.Data;
import org.junit.jupiter.api.Test;
import java.io.IOException;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

@Data
class UserDto {
    private String name;
    private String email;
}

public class JsonDataTest {
    private final ObjectMapper mapper = new ObjectMapper();

    @Test
    void testWithJsonData() throws IOException {
        UserDto user = mapper.readValue(
            getClass().getClassLoader().getResourceAsStream("test-data/users.json"),
            UserDto.class
        );

        given()
            .contentType("application/json")
            .body(user)
        .when()
            .post("https://api.example.com/v1/users")
        .then()
            .statusCode(201)
            .body("email", equalTo(user.getEmail()));
    }
}
```

**Почему это важно**:
- Уменьшает дублирование кода за счёт фабрик и шаблонов.
- Устраняет хардкод, делая тесты гибкими.
- Упрощает обновление данных через внешние файлы.

## Связь с клиент-серверной моделью

Эти практики обеспечивают стабильное и воспроизводимое взаимодействие клиента (теста) с сервером. Централизация конфигурации унифицирует запросы, фабрики данных моделируют реальные клиентские сценарии, а избегание `Thread.sleep` гарантирует надёжную работу с асинхронными ответами сервера.

## Лучшие практики и отладка

- **Стабильность**: Проверяйте уникальность данных в фабриках (например, с UUID). Используйте Awaitility для асинхронных API.
- **Логирование**: Настройте SLF4J с Logback для детальных логов:
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

- **Отладка**: Используйте IntelliJ IDEA для проверки данных из фабрик или JSON. Логируйте запросы/ответы с помощью `log().all()`.
- **Распространённые ошибки**:
  - Множественные проверки в одном тесте (разделяйте тесты).
  - Хардкод значений (используйте фабрики или файлы).
  - Использование `Thread.sleep` (заменяйте на Awaitility).

## Источники
- [RestAssured Official Documentation](http://rest-assured.io/)
- [Awaitility Documentation](https://github.com/awaitility/awaitility)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [TestNG Documentation](https://testng.org/doc/)
- [SLF4J Documentation](https://www.slf4j.org/)

---
[**← Назад к оглавлению**](../README.md)