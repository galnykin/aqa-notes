# Цели использования RestAssured

RestAssured — это библиотека на Java, предназначенная для упрощения тестирования API через HTTP-запросы. Она предоставляет удобный DSL (Domain-Specific Language), который делает тесты читаемыми, поддерживает интеграцию с популярными тестовыми фреймворками и CI/CD-инструментами, а также подходит для работы с различными форматами данных. В этой статье рассматриваются ключевые цели использования RestAssured, его преимущества, примеры применения и лучшие практики.

## Упрощение написания HTTP-запросов

RestAssured упрощает отправку HTTP-запросов (GET, POST, PUT, DELETE и др.) и проверку ответов. Вместо использования низкоуровневых библиотек, таких как Apache HttpClient, разработчик может сосредоточиться на логике теста. Библиотека абстрагирует работу с HTTP, предоставляя методы для задания заголовков, параметров, тела запроса и проверки статус-кодов, JSON/XML-ответов.

### Пример кода (JUnit 5):
```java
import io.restassured.RestAssured;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class ApiTests {
    @BeforeEach
    void setUp() {
        RestAssured.baseURI = "https://api.example.com";
    }

    @Test
    void testGetUser() {
        // Arrange
        String userId = "123";

        // Act & Assert
        given()
            .pathParam("id", userId)
        .when()
            .get("/users/{id}")
        .then()
            .statusCode(200)
            .body("id", equalTo(userId))
            .body("name", equalTo("John Doe"));
    }
}
```

### Пример кода (TestNG):
```java
import io.restassured.RestAssured;
import org.testng.annotations.BeforeClass;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class ApiTests {
    @BeforeClass
    public void setUp() {
        RestAssured.baseURI = "https://api.example.com";
    }

    @Test
    public void testGetUser() {
        // Arrange
        String userId = "123";

        // Act & Assert
        given()
            .pathParam("id", userId)
        .when()
            .get("/users/{id}")
        .then()
            .statusCode(200)
            .body("id", equalTo(userId))
            .body("name", equalTo("John Doe"));
    }
}
```

## Читаемость и декларативность DSL-стиля

RestAssured использует декларативный стиль, основанный на цепочках методов (`given-when-then`), что делает тесты понятными даже для тех, кто не знаком с деталями HTTP. Это соответствует шаблону AAA (Arrange-Act-Assert) или Given-When-Then, улучшая восприятие кода. Например, проверка ответа API становится компактной и выразительной.

### Связь с клиент-серверной моделью
RestAssured ориентирован на тестирование серверной части в клиент-серверной архитектуре. Клиент (тест) отправляет запросы к серверу (API), а RestAssured проверяет корректность ответов, включая статус-коды, заголовки и тело ответа. Это особенно полезно для RESTful API, где важны проверки JSON или XML-структур.

### Пример проверки JSON:
```java
import io.restassured.RestAssured;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.hasItems;

public class JsonValidationTest {
    @Test
    void testJsonResponse() {
        given()
            .get("https://api.example.com/users")
        .then()
            .statusCode(200)
            .body("users.name", hasItems("John Doe", "Jane Smith"));
    }
}
```

## Интеграция с JUnit/TestNG, Allure и CI/CD

RestAssured легко интегрируется с JUnit 5 и TestNG, что позволяет встраивать API-тесты в существующие тестовые фреймворки. Для генерации отчётов можно использовать Allure, который поддерживает RestAssured через аспект AllureRestAssured. Это позволяет создавать наглядные отчёты с деталями запросов и ответов.

### Настройка Allure с RestAssured:
Добавьте зависимость в `pom.xml`:
```xml
<dependency>
    <groupId>io.qameta.allure</groupId>
    <artifactId>allure-rest-assured</artifactId>
    <version>2.24.0</version>
    <scope>test</scope>
</dependency>
```

Пример теста с Allure:
```java
import io.qameta.allure.restassured.AllureRestAssured;
import io.restassured.RestAssured;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;

public class AllureApiTest {
    @Test
    void testWithAllure() {
        given()
            .filter(new AllureRestAssured())
            .get("https://api.example.com/users")
        .then()
            .statusCode(200);
    }
}
```

Для CI/CD (например, Jenkins или GitHub Actions) тесты с RestAssured запускаются через Maven. Пример настройки `Jenkinsfile`:
```groovy
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh 'mvn clean test'
            }
        }
        stage('Allure Report') {
            steps {
                allure([
                    includeProperties: false,
                    jdk: '',
                    resultsDirectory: 'target/allure-results'
                ])
            }
        }
    }
}
```

## Поддержка различных форматов данных

RestAssured поддерживает работу с JSON, XML, multipart/form-data, OAuth, cookies и другие форматы. Это делает библиотеку универсальной для тестирования API с различными типами данных.

### Пример отправки POST с JSON:
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;

public class PostRequestTest {
    @Test
    void testPostUser() {
        String requestBody = "{ \"name\": \"John Doe\", \"email\": \"john@example.com\" }";

        given()
            .contentType(ContentType.JSON)
            .body(requestBody)
        .when()
            .post("https://api.example.com/users")
        .then()
            .statusCode(201)
            .body("email", equalTo("john@example.com"));
    }
}
```

## Лучшие практики и отладка

- **Стабильность тестов**: Используйте явные ожидания для асинхронных API (например, `RestAssured.given().await()` для асинхронных ответов). Проверяйте статус-коды и используйте стабильные пути в JSON/XML.
- **Логирование**: Настройте SLF4J с Logback для логирования запросов и ответов. Пример настройки в `logback-test.xml`:
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

- **Отладка**: Для отладки запросов используйте `log().all()` в RestAssured:
```java
given().log().all().get("https://api.example.com/users").then().log().all();
```

- **Распространённые ошибки**:
  - Неправильная настройка `baseURI` или заголовков.
  - Игнорирование проверки сертификатов для HTTPS (решайте через `RestAssured.useRelaxedHTTPSValidation()`).
  - Неправильные JSON-пути (используйте JSONPath или GPath для проверки структуры).

## Дополнительно

### Использование Testcontainers
Testcontainers позволяет запускать API в Docker-контейнерах для изолированного тестирования. Пример:
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
- [Allure Framework Documentation](https://docs.qameta.io/allure/)
- [Testcontainers Documentation](https://testcontainers.org/)

---
[**← Назад к оглавлению**](../README.md)