# RequestSpecification и ResponseSpecification

`RequestSpecification` и `ResponseSpecification` в RestAssured позволяют создавать переиспользуемые шаблоны для HTTP-запросов и ожидаемых ответов. Это упрощает настройку тестов, унифицирует проверки и уменьшает дублирование кода. В этой статье рассматриваются создание шаблонов, настройка заголовков, таймаутов, логирования, а также проверка статус-кодов, схем ответов и времени ответа.

## Переиспользуемые шаблоны запросов и ответов

`RequestSpecification` задаёт параметры запросов, такие как базовый URL, заголовки, параметры запроса и тело. `ResponseSpecification` определяет ожидаемые параметры ответа, включая статус-коды, формат тела и заголовки. Использование спецификаций позволяет централизовать конфигурацию и применять её к множеству тестов.

### Пример RequestSpecification и ResponseSpecification
```java
import io.restassured.builder.RequestSpecBuilder;
import io.restassured.builder.ResponseSpecBuilder;
import io.restassured.http.ContentType;
import io.restassured.specification.RequestSpecification;
import io.restassured.specification.ResponseSpecification;

public class ApiSpecifications {
    public static RequestSpecification defaultRequestSpec() {
        return new RequestSpecBuilder()
                .setBaseUri("https://api.example.com")
                .setBasePath("/v1")
                .setContentType(ContentType.JSON)
                .addHeader("Authorization", "Bearer token123")
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

### Использование в тесте (JUnit 5)
```java
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class UserApiTest {
    @Test
    void testGetUser() {
        given()
            .spec(ApiSpecifications.defaultRequestSpec())
        .when()
            .get("/users/123")
        .then()
            .spec(ApiSpecifications.defaultResponseSpec())
            .body("name", equalTo("John Doe"));
    }
}
```

### Использование в тесте (TestNG)
```java
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class UserApiTest {
    @Test
    public void testGetUser() {
        given()
            .spec(ApiSpecifications.defaultRequestSpec())
        .when()
            .get("/users/123")
        .then()
            .spec(ApiSpecifications.defaultResponseSpec())
            .body("name", equalTo("John Doe"));
    }
}
```

## Установка заголовков, таймаутов, логирования

`RequestSpecification` позволяет настраивать заголовки, таймауты и логирование для всех запросов. Это особенно полезно для добавления аутентификации, управления временем ожидания и отладки.

### Настройка заголовков
Заголовки, такие как `Authorization` или `Accept`, задаются в `RequestSpecBuilder`:
```java
import io.restassured.builder.RequestSpecBuilder;
import io.restassured.specification.RequestSpecification;

public class ApiSpecifications {
    public static RequestSpecification authRequestSpec() {
        return new RequestSpecBuilder()
                .setBaseUri("https://api.example.com")
                .setBasePath("/v1")
                .addHeader("Authorization", "Bearer token123")
                .addHeader("Accept", "application/json")
                .setContentType(ContentType.JSON)
                .build();
    }
}
```

### Настройка таймаутов
Таймауты задаются через `RestAssuredConfig`:
```java
import io.restassured.RestAssured;
import io.restassured.config.HttpClientConfig;
import io.restassured.config.RestAssuredConfig;
import io.restassured.builder.RequestSpecBuilder;
import io.restassured.specification.RequestSpecification;

public class ApiSpecifications {
    public static RequestSpecification timeoutRequestSpec() {
        return new RequestSpecBuilder()
                .setBaseUri("https://api.example.com")
                .setBasePath("/v1")
                .setConfig(RestAssuredConfig.config()
                        .httpClient(HttpClientConfig.httpClientConfig()
                                .setParam("http.connection.timeout", 5000)
                                .setParam("http.socket.timeout", 5000)))
                .build();
    }
}
```

### Настройка логирования
Логирование запросов и ответов включается через `log().all()`:
```java
import io.restassured.builder.RequestSpecBuilder;
import io.restassured.specification.RequestSpecification;

public class ApiSpecifications {
    public static RequestSpecification loggingRequestSpec() {
        return new RequestSpecBuilder()
                .setBaseUri("https://api.example.com")
                .setBasePath("/v1")
                .log().all() // Логирует запросы и ответы
                .build();
    }
}
```

Для более детального логирования используйте SLF4J с Logback:
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

## Проверка статус-кодов, схем, времени ответа

`ResponseSpecification` позволяет проверять статус-коды, формат ответа (JSON, XML) и время ответа. Это помогает унифицировать проверки и выявлять проблемы с производительностью.

### Проверка статус-кодов
```java
import io.restassured.builder.ResponseSpecBuilder;
import io.restassured.specification.ResponseSpecification;

public class ApiSpecifications {
    public static ResponseSpecification createdResponseSpec() {
        return new ResponseSpecBuilder()
                .expectStatusCode(201)
                .expectContentType(ContentType.JSON)
                .build();
    }
}
```

### Проверка схем JSON
RestAssured поддерживает валидацию JSON-схем через библиотеку `json-schema-validator`:
```xml
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>json-schema-validator</artifactId>
    <version>5.4.0</version>
    <scope>test</scope>
</dependency>
```

Пример JSON-схемы (`user-schema.json`):
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "id": { "type": "string" },
    "name": { "type": "string" },
    "email": { "type": "string" }
  },
  "required": ["id", "name", "email"]
}
```

Проверка схемы в тесте:
```java
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static io.restassured.module.jsv.JsonSchemaValidator.matchesJsonSchemaInClasspath;

public class SchemaValidationTest {
    @Test
    void testUserSchema() {
        given()
            .spec(ApiSpecifications.defaultRequestSpec())
        .when()
            .get("/users/123")
        .then()
            .spec(ApiSpecifications.defaultResponseSpec())
            .body(matchesJsonSchemaInClasspath("user-schema.json"));
    }
}
```

### Проверка времени ответа
Время ответа проверяется через `time()`:
```java
import io.restassured.builder.ResponseSpecBuilder;
import io.restassured.specification.ResponseSpecification;
import java.util.concurrent.TimeUnit;

public class ApiSpecifications {
    public static ResponseSpecification fastResponseSpec() {
        return new ResponseSpecBuilder()
                .expectStatusCode(200)
                .expectContentType(ContentType.JSON)
                .expectResponseTime(lessThan(2000L), TimeUnit.MILLISECONDS)
                .build();
    }
}
```

### Использование в тесте
```java
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;

public class PerformanceTest {
    @Test
    void testResponseTime() {
        given()
            .spec(ApiSpecifications.defaultRequestSpec())
        .when()
            .get("/users/123")
        .then()
            .spec(ApiSpecifications.fastResponseSpec());
    }
}
```

## Связь с клиент-серверной моделью

`RequestSpecification` моделирует клиентскую часть, задавая параметры запросов (заголовки, тело, параметры URL). `ResponseSpecification` проверяет серверные ответы, включая их структуру и производительность. Это обеспечивает тестирование полного цикла взаимодействия между клиентом и сервером.

## Лучшие практики и отладка

- **Стабильность**: Используйте `ResponseSpecification` для проверки обязательных заголовков (например, `Content-Type`) и статус-кодов.
- **Отладка**: Добавьте `log().all()` в `ResponseSpecification` для вывода ответов:
```java
public static ResponseSpecification debugResponseSpec() {
    return new ResponseSpecBuilder()
            .expectStatusCode(200)
            .log().all()
            .build();
}
```

- **Распространённые ошибки**:
  - Неправильные заголовки в `RequestSpecification` (например, отсутствие `Content-Type`).
  - Слишком строгие проверки в `ResponseSpecification` (например, ожидание точного времени ответа).
  - Ошибки в JSON-схемах (проверяйте схемы с помощью онлайн-валидаторов).

## Источники
- [RestAssured Official Documentation](http://rest-assured.io/)
- [JSON Schema Validator Documentation](https://github.com/rest-assured/rest-assured/wiki/Usage#json-schema-validation)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [TestNG Documentation](https://testng.org/doc/)

---
[**← Назад к оглавлению**](../README.md)