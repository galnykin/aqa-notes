# Структура API-тестов

Организованная структура API-тестов упрощает их поддержку, масштабирование и чтение. Использование слоёв (спецификации, модели, тесты) и базового класса для конфигурации позволяет централизовать настройки и унифицировать тесты. В этой статье описывается, как структурировать API-тесты с использованием RestAssured, применяя шаблон `given-when-then`, и лучшие практики для обеспечения стабильности и отладки.

## Разделение на слои

Чёткое разделение API-тестов на слои улучшает читаемость и повторное использование кода. Основные слои:

- **Спецификации** (Request/Response Specifications): Определяют общие параметры запросов (заголовки, базовый URL, таймауты) и ожидаемые ответы (статус-коды, форматы). Используются для унификации тестов.
- **Модели** (Data Models): POJO-классы, представляющие структуру JSON/XML-ответов. Упрощают десериализацию и валидацию данных.
- **Тесты**: Содержат бизнес-логику тестов, используют спецификации и модели для выполнения запросов и проверок.

### Пример спецификации
```java
import io.restassured.builder.RequestSpecBuilder;
import io.restassured.builder.ResponseSpecBuilder;
import io.restassured.http.ContentType;
import io.restassured.specification.RequestSpecification;
import io.restassured.specification.ResponseSpecification;

public class ApiSpecifications {
    public static RequestSpecification requestSpec() {
        return new RequestSpecBuilder()
                .setBaseUri("https://api.example.com")
                .setContentType(ContentType.JSON)
                .addHeader("Authorization", "Bearer token123")
                .build();
    }

    public static ResponseSpecification responseSpec() {
        return new ResponseSpecBuilder()
                .expectStatusCode(200)
                .expectContentType(ContentType.JSON)
                .build();
    }
}
```

### Пример модели
```java
import lombok.Data;

@Data
public class User {
    private String id;
    private String name;
    private String email;
}
```

### Пример теста с использованием спецификаций и моделей
```java
import io.restassured.RestAssured;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class UserApiTest {
    @Test
    void testGetUser() {
        given()
            .spec(ApiSpecifications.requestSpec())
        .when()
            .get("/users/123")
        .then()
            .spec(ApiSpecifications.responseSpec())
            .body("name", equalTo("John Doe"));
    }
}
```

## Использование BaseApiTest для конфигурации

Базовый класс `BaseApiTest` централизует настройку RestAssured (например, `baseURI`, фильтры для логирования или Allure). Это уменьшает дублирование кода и упрощает изменение конфигурации.

### Пример BaseApiTest (JUnit 5)
```java
import io.qameta.allure.restassured.AllureRestAssured;
import io.restassured.RestAssured;
import org.junit.jupiter.api.BeforeEach;

public abstract class BaseApiTest {
    @BeforeEach
    void setUp() {
        RestAssured.baseURI = "https://api.example.com";
        RestAssured.filters(new AllureRestAssured());
        RestAssured.useRelaxedHTTPSValidation(); // Для HTTPS без проверки сертификатов
    }
}
```

### Пример теста, использующего BaseApiTest
```java
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class UserApiTest extends BaseApiTest {
    @Test
    void testGetUser() {
        given()
            .spec(ApiSpecifications.requestSpec())
        .when()
            .get("/users/123")
        .then()
            .spec(ApiSpecifications.responseSpec())
            .body("name", equalTo("John Doe"));
    }
}
```

### Пример BaseApiTest (TestNG)
```java
import io.qameta.allure.restassured.AllureRestAssured;
import io.restassured.RestAssured;
import org.testng.annotations.BeforeClass;

public abstract class BaseApiTest {
    @BeforeClass
    public void setUp() {
        RestAssured.baseURI = "https://api.example.com";
        RestAssured.filters(new AllureRestAssured());
        RestAssured.useRelaxedHTTPSValidation();
    }
}
```

### Пример теста с TestNG
```java
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class UserApiTest extends BaseApiTest {
    @Test
    public void testGetUser() {
        given()
            .spec(ApiSpecifications.requestSpec())
        .when()
            .get("/users/123")
        .then()
            .spec(ApiSpecifications.responseSpec())
            .body("name", equalTo("John Doe"));
    }
}
```

## Единый шаблон: given() → when() → then()

Шаблон `given-when-then` (часть DSL RestAssured) соответствует подходу AAA (Arrange-Act-Assert) или Given-When-Then, делая тесты последовательными и понятными:
- **given()**: Настройка запроса (параметры, заголовки, тело).
- **when()**: Выполнение HTTP-запроса (GET, POST и т.д.).
- **then()**: Проверка ответа (статус-код, тело, заголовки).

### Пример шаблона
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class PostApiTest {
    @Test
    void testCreateUser() {
        String requestBody = "{ \"name\": \"Jane Smith\", \"email\": \"jane@example.com\" }";

        given()
            .contentType(ContentType.JSON)
            .body(requestBody)
        .when()
            .post("https://api.example.com/users")
        .then()
            .statusCode(201)
            .body("email", equalTo("jane@example.com"));
    }
}
```

## Связь с клиент-серверной моделью

API-тесты проверяют взаимодействие клиента (теста) с сервером через HTTP. RestAssured позволяет моделировать запросы клиента, включая аутентификацию, параметры и тело, а также проверять ответы сервера. Для сложных сценариев (например, OAuth) можно настроить токены в спецификациях.

### Пример с OAuth
```java
import io.restassured.specification.RequestSpecification;
import static io.restassured.RestAssured.given;

public class OAuthApiTest {
    public static RequestSpecification oauthSpec() {
        return new RequestSpecBuilder()
                .setBaseUri("https://api.example.com")
                .addHeader("Authorization", "Bearer " + getAccessToken())
                .build();
    }

    private static String getAccessToken() {
        return given()
                .formParam("client_id", "client123")
                .formParam("client_secret", "secret")
                .formParam("grant_type", "client_credentials")
            .when()
                .post("https://auth.example.com/token")
            .then()
                .extract().path("access_token");
    }
}
```

## Лучшие практики и отладка

- **Стабильность тестов**: Используйте `await()` для асинхронных API. Проверяйте статус-коды и JSON-пути с помощью AssertJ для мягких утверждений:
```java
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.assertj.core.api.Assertions.assertThat;

public class SoftAssertionsTest {
    @Test
    void testSoftAssertions() {
        var response = given().get("https://api.example.com/users/123").then().extract().response();
        assertThat(response.statusCode()).isEqualTo(200);
        assertThat(response.jsonPath().getString("name")).isEqualTo("John Doe");
    }
}
```

- **Логирование**: Настройте SLF4J с Logback для записи запросов/ответов:
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

- **Отладка**: Используйте `log().all()` для вывода запросов и ответов. В IntelliJ IDEA настройте точки останова для проверки JSON-путей или ответов.
- **Распространённые ошибки**:
  - Неправильная базовая конфигурация (`baseURI`, заголовки).
  - Ошибки в JSONPath (используйте инструменты вроде JSONPath Finder).
  - Игнорирование асинхронных ответов (добавляйте ожидания).

## Дополнительно

### Использование Testcontainers
Testcontainers позволяет запускать API в Docker-контейнерах для изолированных тестов:
```java
import org.testcontainers.containers.GenericContainer;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;

public class TestcontainersApiTest {
    @Test
    void testApiInContainer() {
        try (GenericContainer<?> container = new GenericContainer<>("my-api-image:latest")
                .withExposedPorts(8080)) {
            container.start();
            String baseUri = "http://" + container.getHost() + ":" + container.getFirstMappedPort();

            given()
                .baseUri(baseUri)
                .get("/api/health")
            .then()
                .statusCode(200);
        }
    }
}
```

### Настройка CI/CD
Пример `GitHub Actions` для запуска тестов:
```yaml
name: API Tests
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
      - name: Run tests
        run: mvn clean test
      - name: Publish Allure Report
        uses: simple-elf/allure-report-action@v1.7
        with:
          allure_results: target/allure-results
```

### Настройка Maven
Пример `pom.xml`:
```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>api-tests</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <java.version>17</java.version>
        <rest-assured.version>5.4.0</rest-assured.version>
        <junit-jupiter.version>5.10.2</junit-jupiter.version>
        <allure.version>2.24.0</allure.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>io.rest-assured</groupId>
            <artifactId>rest-assured</artifactId>
            <version>${rest-assured.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>${junit-jupiter.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>io.qameta.allure</groupId>
            <artifactId>allure-rest-assured</artifactId>
            <version>${allure.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>testcontainers</artifactId>
            <version>1.20.1</version>
            <scope>test</scope>
        </dependency>
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
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.2.5</version>
            </plugin>
            <plugin>
                <groupId>io.qameta.allure</groupId>
                <artifactId>allure-maven</artifactId>
                <version>2.12.0</version>
                <configuration>
                    <reportVersion>${allure.version}</reportVersion>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

## Источники
- [RestAssured Official Documentation](http://rest-assured.io/)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [TestNG Documentation](https://testng.org/doc/)
- [Allure Framework Documentation](https://docs.qameta.io/allure/)
- [Testcontainers Documentation](https://testcontainers.org/)

---
[**← Назад к оглавлению**](../README.md)