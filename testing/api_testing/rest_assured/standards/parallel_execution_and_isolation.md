# Параллельный запуск и изоляция

Параллельный запуск тестов ускоряет их выполнение, а изоляция окружения предотвращает влияние тестов друг на друга. В этой статье рассматриваются настройка параллельного запуска через TestNG, обеспечение изоляции с помощью уникальных данных и очистки, а также использование mock-сервисов и Testcontainers для создания независимых тестовых окружений.

## Поддержка параллельного запуска через TestNG

TestNG поддерживает параллельный запуск тестов на уровне классов, методов или групп, что позволяет сократить время выполнения большого набора тестов. Для API-тестов с RestAssured это особенно полезно, так как запросы часто независимы.

### Настройка параллельного запуска в TestNG
Создайте файл `testng.xml` для настройки параллельного выполнения:

```xml
<!DOCTYPE suite SYSTEM "http://testng.org/testng-1.0.dtd">
<suite name="ApiTestSuite" parallel="tests" thread-count="4">
    <test name="UserTests">
        <classes>
            <class name="com.example.UserApiTest"/>
        </classes>
    </test>
    <test name="AuthTests">
        <classes>
            <class name="com.example.AuthApiTest"/>
        </classes>
    </test>
</suite>
```

- **parallel="tests"**: Запускает тесты (`<test>`) параллельно.
- **thread-count="4"**: Указывает максимальное количество потоков (4 в данном случае).

### Пример тестового класса
```java
import io.restassured.http.ContentType;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class UserApiTest {
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

### Запуск тестов
Запустите тесты через Maven:
```bash
mvn test -DsuiteXmlFile=testng.xml
```

**Примечания**:
- Используйте `parallel="methods"` для параллельного запуска методов внутри одного класса.
- Убедитесь, что тесты не зависят друг от друга, чтобы избежать конфликтов.
- Настройте `thread-count` в зависимости от мощности CI/CD-сервера.

## Изоляция окружения: уникальные данные, cleanup

Изоляция тестов предотвращает их взаимное влияние, обеспечивая независимость выполнения. Это достигается с помощью уникальных данных и очистки состояния после тестов.

### Уникальные данные
Генерируйте уникальные данные для каждого теста, например, с помощью UUID или временных меток.

#### Пример генерации уникальных данных
```java
import java.util.UUID;
import lombok.Data;

@Data
class UserDto {
    private String id;
    private String name;
    private String email;
}

public class UserFactory {
    public static UserDto createUniqueUser() {
        String uniqueId = UUID.randomUUID().toString();
        UserDto user = new UserDto();
        user.setId(uniqueId);
        user.setName("User_" + uniqueId);
        user.setEmail("user_" + uniqueId + "@example.com");
        return user;
    }
}
```

#### Использование в тесте
```java
import io.restassured.http.ContentType;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class UniqueDataTest {
    @Test
    public void testCreateUniqueUser() {
        UserDto user = UserFactory.createUniqueUser();

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

### Очистка данных (cleanup)
Для очистки данных после теста используйте аннотации `@AfterMethod` или вызовите API для удаления созданных ресурсов.

#### Пример очистки
```java
import io.restassured.http.ContentType;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class CleanupTest {
    private String createdUserId;

    @Test
    public void testCreateAndDeleteUser() {
        UserDto user = UserFactory.createUniqueUser();

        createdUserId = given()
            .contentType(ContentType.JSON)
            .body(user)
        .when()
            .post("https://api.example.com/v1/users")
        .then()
            .statusCode(201)
            .body("id", equalTo(user.getId()))
            .extract().path("id");
    }

    @AfterMethod
    public void cleanup() {
        if (createdUserId != null) {
            given()
                .contentType(ContentType.JSON)
                .header("Authorization", "Bearer token123")
            .when()
                .delete("https://api.example.com/v1/users/{id}", createdUserId)
            .then()
                .statusCode(204);
        }
    }
}
```

**Примечания**:
- Используйте уникальные идентификаторы (UUID) для данных, чтобы избежать коллизий.
- Проверяйте, что API для очистки доступен и работает корректно.
- Логируйте действия очистки для отладки:
```java
given().log().all().delete("/users/{id}", createdUserId);
```

## Использование mock-сервисов или Testcontainers

Mock-сервисы и Testcontainers обеспечивают изолированные окружения для тестов, минимизируя зависимость от внешних серверов.

### Mock-сервисы
WireMock используется для создания заглушек API.

#### Настройка WireMock
Добавьте зависимость в `pom.xml`:
```xml
<dependency>
    <groupId>com.github.tomakehurst</groupId>
    <artifactId>wiremock-jre8</artifactId>
    <version>2.35.0</version>
    <scope>test</scope>
</dependency>
```

#### Пример теста с WireMock
```java
import com.github.tomakehurst.wiremock.WireMockServer;
import io.restassured.http.ContentType;
import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;
import static com.github.tomakehurst.wiremock.client.WireMock.*;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class WireMockTest {
    private static WireMockServer wireMockServer;

    @BeforeAll
    static void setUp() {
        wireMockServer = new WireMockServer(8080);
        wireMockServer.start();
        stubFor(get(urlEqualTo("/v1/users/123"))
            .willReturn(aResponse()
                .withStatus(200)
                .withHeader("Content-Type", "application/json")
                .withBody("{\"id\": \"123\", \"name\": \"John Doe\", \"email\": \"john@example.com\"}")));
    }

    @AfterAll
    static void tearDown() {
        wireMockServer.stop();
    }

    @Test
    void testMockedApi() {
        given()
            .contentType(ContentType.JSON)
        .when()
            .get("http://localhost:8080/v1/users/123")
        .then()
            .statusCode(200)
            .body("name", equalTo("John Doe"));
    }
}
```

### Использование Testcontainers
Testcontainers запускает API в Docker-контейнере, обеспечивая полную изоляцию.

#### Настройка Testcontainers
Добавьте зависимость в `pom.xml`:
```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers</artifactId>
    <version>1.20.1</version>
    <scope>test</scope>
</dependency>
```

#### Пример теста с Testcontainers
```java
import org.testcontainers.containers.GenericContainer;
import org.testcontainers.utility.DockerImageName;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class TestcontainersTest {
    @Test
    void testApiInContainer() {
        try (GenericContainer<?> container = new GenericContainer<>(DockerImageName.parse("my-api-image:latest"))
                .withExposedPorts(8080)) {
            container.start();
            String baseUri = "http://" + container.getHost() + ":" + container.getFirstMappedPort();

            given()
                .baseUri(baseUri)
                .contentType("application/json")
            .when()
                .get("/v1/users/123")
            .then()
                .statusCode(200)
                .body("name", equalTo("John Doe"));
        }
    }
}
```

**Примечания**:
- WireMock подходит для быстрого создания заглушек без запуска реального сервера.
- Testcontainers обеспечивает полноценное окружение, но требует Docker и больше ресурсов.
- Логируйте действия контейнера для отладки: `container.withLogConsumer(output -> System.out.println(output.getUtf8String()))`.

## Связь с клиент-серверной моделью

Параллельный запуск и изоляция моделируют независимые клиентские запросы к серверу, минимизируя конфликты. Уникальные данные и очистка гарантируют, что каждый клиент (тест) работает с чистым состоянием, а mock-сервисы и Testcontainers эмулируют серверное поведение в изолированной среде.

## Лучшие практики и отладка

- **Стабильность**: Используйте уникальные данные и проверяйте их уникальность перед отправкой. Настройте таймауты для параллельных тестов:
```java
RestAssured.config().httpClient(HttpClientConfig.httpClientConfig().setParam("http.connection.timeout", 5000));
```

- **Логирование**: Настройте SLF4J с Logback для записи действий:
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

- **Отладка**: Используйте IntelliJ IDEA для анализа логов параллельных тестов. Для Testcontainers проверяйте логи контейнера.
- **Распространённые ошибки**:
  - Конфликты данных при параллельном запуске (используйте UUID).
  - Отсутствие очистки, приводящее к загрязнению базы данных.
  - Ошибки настройки WireMock/Testcontainers (проверяйте порты и образы).

## Источники
- [TestNG Documentation](https://testng.org/doc/)
- [RestAssured Official Documentation](http://rest-assured.io/)
- [WireMock Documentation](http://wiremock.org/docs/)
- [Testcontainers Documentation](https://testcontainers.org/)
- [SLF4J Documentation](https://www.slf4j.org/)

---
[**← Назад к оглавлению**](../README.md)