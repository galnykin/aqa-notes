# Интеграция с Allure

Интеграция API-тестов с Allure улучшает отчётность, делая результаты тестов наглядными и информативными. Allure позволяет добавлять шаги, прикреплять тела запросов и ответов, а также генерировать curl-команды для воспроизведения запросов. В этой статье рассматриваются настройка Allure, использование `Allure.step()`, вложения через `Allure.addAttachment()`, и автоматическая генерация curl-команд с помощью `AllureRestAssured`.

## Настройка Allure

Для интеграции Allure с RestAssured и тестовыми фреймворками (JUnit 5 или TestNG) добавьте зависимости в `pom.xml`:

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
<dependency>
    <groupId>io.qameta.allure</groupId>
    <artifactId>allure-testng</artifactId>
    <version>2.24.0</version>
    <scope>test</scope>
</dependency>
```

Создайте файл `allure.properties` в `src/test/resources` для настройки директории результатов:
```properties
allure.results.directory=target/allure-results
```

Для генерации отчётов добавьте плагин в `pom.xml`:
```xml
<plugin>
    <groupId>io.qameta.allure</groupId>
    <artifactId>allure-maven</artifactId>
    <version>2.12.0</version>
    <configuration>
        <reportVersion>2.24.0</reportVersion>
    </configuration>
</plugin>
```

## Добавление шагов через Allure.step(...)

Метод `Allure.step()` позволяет структурировать тесты, добавляя описательные шаги в отчёт. Это улучшает читаемость и помогает отслеживать этапы выполнения теста.

### Пример добавления шагов (JUnit 5)
```java
import io.qameta.allure.Allure;
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class AllureStepsTest {
    @Test
    void testWithSteps() {
        Allure.step("Подготовка запроса", () -> {
            given()
                .contentType(ContentType.JSON)
                .body("{ \"name\": \"Jane Smith\", \"email\": \"jane@example.com\" }");
        });

        Allure.step("Отправка POST-запроса", () -> {
            given()
                .contentType(ContentType.JSON)
                .body("{ \"name\": \"Jane Smith\", \"email\": \"jane@example.com\" }")
            .when()
                .post("https://api.example.com/v1/users");
        });

        Allure.step("Проверка ответа", () -> {
            given()
                .contentType(ContentType.JSON)
                .body("{ \"name\": \"Jane Smith\", \"email\": \"jane@example.com\" }")
            .when()
                .post("https://api.example.com/v1/users")
            .then()
                .statusCode(201)
                .body("email", equalTo("jane@example.com"));
        });
    }
}
```

### Пример добавления шагов (TestNG)
```java
import io.qameta.allure.Allure;
import io.restassured.http.ContentType;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class AllureStepsTest {
    @Test
    public void testWithSteps() {
        Allure.step("Подготовка запроса", () -> {
            given()
                .contentType(ContentType.JSON)
                .body("{ \"name\": \"Jane Smith\", \"email\": \"jane@example.com\" }");
        });

        Allure.step("Отправка POST-запроса", () -> {
            given()
                .contentType(ContentType.JSON)
                .body("{ \"name\": \"Jane Smith\", \"email\": \"jane@example.com\" }")
            .when()
                .post("https://api.example.com/v1/users");
        });

        Allure.step("Проверка ответа", () -> {
            given()
                .contentType(ContentType.JSON)
                .body("{ \"name\": \"Jane Smith\", \"email\": \"jane@example.com\" }")
            .when()
                .post("https://api.example.com/v1/users")
            .then()
                .statusCode(201)
                .body("email", equalTo("jane@example.com"));
        });
    }
}
```

**Примечания**:
- Каждый шаг отображается в отчёте Allure с названием и статусом.
- Используйте `Allure.step()` для логически связанных частей теста (например, настройка, запрос, проверка).

## Вложение тела запроса и ответа (Allure.addAttachment)

Метод `Allure.addAttachment()` позволяет прикреплять к отчёту произвольные данные, такие как тело запроса, ответа или другие детали. Это полезно для анализа ошибок.

### Пример добавления вложений (JUnit 5)
```java
import io.qameta.allure.Allure;
import io.restassured.http.ContentType;
import io.restassured.response.Response;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;

public class AllureAttachmentTest {
    @Test
    void testWithAttachments() {
        String requestBody = "{ \"name\": \"Jane Smith\", \"email\": \"jane@example.com\" }";
        Allure.addAttachment("Request Body", "application/json", requestBody, ".json");

        Response response = given()
            .contentType(ContentType.JSON)
            .body(requestBody)
        .when()
            .post("https://api.example.com/v1/users")
        .then()
            .statusCode(201)
            .extract().response();

        Allure.addAttachment("Response Body", "application/json", response.asString(), ".json");
        Allure.addAttachment("Response Headers", response.getHeaders().toString());
    }
}
```

### Пример добавления вложений (TestNG)
```java
import io.qameta.allure.Allure;
import io.restassured.http.ContentType;
import io.restassured.response.Response;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;

public class AllureAttachmentTest {
    @Test
    public void testWithAttachments() {
        String requestBody = "{ \"name\": \"Jane Smith\", \"email\": \"jane@example.com\" }";
        Allure.addAttachment("Request Body", "application/json", requestBody, ".json");

        Response response = given()
            .contentType(ContentType.JSON)
            .body(requestBody)
        .when()
            .post("https://api.example.com/v1/users")
        .then()
            .statusCode(201)
            .extract().response();

        Allure.addAttachment("Response Body", "application/json", response.asString(), ".json");
        Allure.addAttachment("Response Headers", response.getHeaders().toString());
    }
}
```

**Примечания**:
- Указывайте тип контента (`application/json`) и расширение (`.json`) для корректного отображения в Allure.
- Вложения полезны для анализа ошибок, особенно при тестировании негативных сценариев.

## Генерация curl-команд для отчёта

Фильтр `AllureRestAssured` автоматически добавляет curl-команды, JSON-данные и заголовки в отчёт Allure. Это позволяет воспроизвести запросы для отладки.

### Пример интеграции с AllureRestAssured
```java
import io.qameta.allure.restassured.AllureRestAssured;
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class AllureCurlTest {
    @Test
    void testCurlGeneration() {
        given()
            .filter(new AllureRestAssured()) // Включаем генерацию curl и логов
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
- `AllureRestAssured` автоматически добавляет в отчёт:
  - curl-команду для воспроизведения запроса.
  - Тело запроса и ответа (JSON/XML).
  - Заголовки запроса и ответа.
- Для проверки curl-команд скопируйте их из отчёта Allure и выполните в терминале.

## Связь с клиент-серверной моделью

Allure фиксирует взаимодействие клиента (теста) с сервером, добавляя в отчёты запросы, ответы и шаги. Это помогает анализировать, какие данные отправляются и получаются, а curl-команды позволяют воспроизвести запросы для проверки серверной логики.

## Лучшие практики и отладка

- **Стабильность**: Используйте `Allure.step()` для разделения теста на логические блоки, чтобы упростить анализ отчётов.
- **Логирование**: Комбинируйте `AllureRestAssured` с `log().all()` для детального логирования при отладке.
- **Отладка**: Просматривайте отчёты Allure в браузере (`mvn allure:serve`) для анализа шагов, вложений и curl-команд. Используйте IntelliJ IDEA для проверки JSON-данных перед добавлением в вложения.
- **Распространённые ошибки**:
  - Отсутствие фильтра `AllureRestAssured`, из-за чего данные запросов не попадают в отчёт.
  - Неправильная настройка `allure.results.directory` (проверяйте путь).
  - Слишком большие вложения (ограничивайте размер JSON для оптимизации).

## Источники
- [Allure Framework Documentation](https://docs.qameta.io/allure/)
- [RestAssured Official Documentation](http://rest-assured.io/)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [TestNG Documentation](https://testng.org/doc/)

---
[**← Назад к оглавлению**](../README.md)