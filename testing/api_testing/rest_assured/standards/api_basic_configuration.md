# Базовая конфигурация API-тестов

Базовая конфигурация API-тестов с использованием RestAssured обеспечивает унифицированный подход к настройке запросов, упрощает поддержку тестов и позволяет адаптировать их под разные окружения (qa, staging, prod). В этой статье рассматриваются настройка `baseURI`, `basePath`, `port` через `RequestSpecBuilder`, централизация конфигурации в классах `ApiConfig` или `SpecFactory`, а также поддержка переменных окружения.

## Настройка baseURI, basePath, port в RequestSpecBuilder

`RequestSpecBuilder` в RestAssured позволяет задавать общие параметры HTTP-запросов, такие как `baseURI`, `basePath` и `port`, которые используются во всех тестах. Это уменьшает дублирование кода и упрощает изменение настроек.

### Пример настройки RequestSpecBuilder
```java
import io.restassured.builder.RequestSpecBuilder;
import io.restassured.http.ContentType;
import io.restassured.specification.RequestSpecification;

public class ApiConfig {
    public static RequestSpecification defaultSpec() {
        return new RequestSpecBuilder()
                .setBaseUri("https://api.example.com")
                .setBasePath("/v1")
                .setPort(443)
                .setContentType(ContentType.JSON)
                .addHeader("Accept", "application/json")
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
            .spec(ApiConfig.defaultSpec())
        .when()
            .get("/users/123")
        .then()
            .statusCode(200)
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
            .spec(ApiConfig.defaultSpec())
        .when()
            .get("/users/123")
        .then()
            .statusCode(200)
            .body("name", equalTo("John Doe"));
    }
}
```

**Примечания**:
- `baseURI`: Полный адрес сервера (например, `https://api.example.com`).
- `basePath`: Общая часть пути API (например, `/v1`).
- `port`: Указывается явно для нестандартных портов (по умолчанию 80 для HTTP, 443 для HTTPS).
- Дополнительные параметры, такие как заголовки или тип контента, также задаются в `RequestSpecBuilder`.

## Централизация конфигурации в ApiConfig или SpecFactory

Централизация конфигурации в одном классе (`ApiConfig` или `SpecFactory`) позволяет управлять настройками из единой точки. Это особенно полезно при работе с большим количеством тестов или сложной структурой API.

### Пример ApiConfig с несколькими спецификациями
```java
import io.restassured.builder.RequestSpecBuilder;
import io.restassured.builder.ResponseSpecBuilder;
import io.restassured.http.ContentType;
import io.restassured.specification.RequestSpecification;
import io.restassured.specification.ResponseSpecification;

public class ApiConfig {
    public static RequestSpecification authSpec() {
        return new RequestSpecBuilder()
                .setBaseUri("https://api.example.com")
                .setBasePath("/auth")
                .setContentType(ContentType.JSON)
                .addHeader("Authorization", "Bearer token123")
                .build();
    }

    public static RequestSpecification userSpec() {
        return new RequestSpecBuilder()
                .setBaseUri("https://api.example.com")
                .setBasePath("/v1/users")
                .setContentType(ContentType.JSON)
                .build();
    }

    public static ResponseSpecification okResponseSpec() {
        return new ResponseSpecBuilder()
                .expectStatusCode(200)
                .expectContentType(ContentType.JSON)
                .build();
    }
}
```

### Пример SpecFactory
```java
import io.restassured.builder.RequestSpecBuilder;
import io.restassured.http.ContentType;
import io.restassured.specification.RequestSpecification;

public class SpecFactory {
    public static RequestSpecification getSpec(String basePath) {
        return new RequestSpecBuilder()
                .setBaseUri("https://api.example.com")
                .setBasePath(basePath)
                .setContentType(ContentType.JSON)
                .build();
    }
}
```

### Использование SpecFactory в тесте
```java
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class UserApiTest {
    @Test
    void testGetUser() {
        given()
            .spec(SpecFactory.getSpec("/v1/users"))
        .when()
            .get("/123")
        .then()
            .spec(ApiConfig.okResponseSpec())
            .body("name", equalTo("John Doe"));
    }
}
```

**Преимущества**:
- Централизованное управление настройками.
- Лёгкое переключение между API (например, `/auth` и `/users`).
- Упрощение добавления новых спецификаций.

## Поддержка переменных окружения (qa, staging, prod)

Для поддержки разных окружений (qa, staging, prod) конфигурацию можно вынести в переменные окружения или файлы свойств. Это позволяет запускать тесты без изменения кода.

### Пример чтения переменных окружения
```java
import io.restassured.builder.RequestSpecBuilder;
import io.restassured.http.ContentType;
import io.restassured.specification.RequestSpecification;

public class ApiConfig {
    public static RequestSpecification envSpec() {
        String env = System.getProperty("env", "qa");
        String baseUri;
        switch (env.toLowerCase()) {
            case "prod":
                baseUri = "https://api.prod.example.com";
                break;
            case "staging":
                baseUri = "https://api.staging.example.com";
                break;
            default:
                baseUri = "https://api.qa.example.com";
        }

        return new RequestSpecBuilder()
                .setBaseUri(baseUri)
                .setBasePath("/v1")
                .setContentType(ContentType.JSON)
                .build();
    }
}
```

### Запуск тестов с переменной окружения
В Maven можно передать переменную через команду:
```bash
mvn test -Denv=prod
```

### Использование файла свойств
Создайте файл `config.properties`:
```properties
qa.baseUri=https://api.qa.example.com
staging.baseUri=https://api.staging.example.com
prod.baseUri=https://api.prod.example.com
```

Чтение файла в `ApiConfig`:
```java
import io.restassured.builder.RequestSpecBuilder;
import io.restassured.http.ContentType;
import io.restassured.specification.RequestSpecification;

import java.io.IOException;
import java.util.Properties;

public class ApiConfig {
    public static RequestSpecification envSpec() {
        Properties props = new Properties();
        try {
            props.load(ApiConfig.class.getClassLoader().getResourceAsStream("config.properties"));
        } catch (IOException e) {
            throw new RuntimeException("Failed to load config.properties", e);
        }

        String env = System.getProperty("env", "qa");
        String baseUri = props.getProperty(env + ".baseUri");

        return new RequestSpecBuilder()
                .setBaseUri(baseUri)
                .setBasePath("/v1")
                .setContentType(ContentType.JSON)
                .build();
    }
}
```

## Связь с клиент-серверной моделью

Базовая конфигурация API-тестов напрямую связана с клиент-серверной архитектурой. `baseURI` и `basePath` задают точку входа для клиентских запросов к серверу, а `port` и заголовки обеспечивают корректное взаимодействие. Поддержка окружений позволяет тестировать API в разных средах, имитируя реальные сценарии использования.

## Лучшие практики и отладка

- **Стабильность**: Проверяйте доступность `baseURI` перед запуском тестов, используя, например, `RestAssured.get(baseUri + "/health").then().statusCode(200)`.
- **Логирование**: Используйте SLF4J с Logback для записи конфигурации и запросов:
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

- **Отладка**: Добавьте `log().all()` в `RequestSpecBuilder` для вывода запросов:
```java
public static RequestSpecification defaultSpec() {
    return new RequestSpecBuilder()
            .setBaseUri("https://api.example.com")
            .setBasePath("/v1")
            .setContentType(ContentType.JSON)
            .log().all()
            .build();
}
```

- **Распространённые ошибки**:
  - Неправильный `baseURI` или `basePath` (проверяйте в документации API).
  - Отсутствие заголовков (например, `Authorization`).
  - Игнорирование HTTPS-сертификатов (используйте `useRelaxedHTTPSValidation()`).

## Источники
- [RestAssured Official Documentation](http://rest-assured.io/)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [TestNG Documentation](https://testng.org/doc/)

---
[**← Назад к оглавлению**](../README.md)