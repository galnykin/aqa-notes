# Работа с JSON

Работа с JSON в RestAssured упрощает тестирование API, предоставляя инструменты для проверки значений ответа, валидации схем и десериализации данных в Java-объекты. В этой статье рассматриваются методы проверки JSON через `body()` и `jsonPath()`, использование `JsonSchemaValidator` для проверки структуры ответа, а также десериализация ответа в объекты с помощью `response.as()`.

## Проверка значений через body(...), jsonPath(...)

RestAssured предоставляет два основных метода для проверки значений в JSON-ответах:
- **`body()`**: Используется для проверки значений в ответе с помощью Hamcrest-матчеров. Подходит для простых проверок.
- **`jsonPath()`**: Позволяет извлекать данные из JSON для более сложных проверок или десериализации.

### Пример проверки с body() (JUnit 5)
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;
import static org.hamcrest.Matchers.hasItems;

public class JsonValidationTest {
    @Test
    void testBodyValidation() {
        given()
            .contentType(ContentType.JSON)
        .when()
            .get("https://api.example.com/v1/users")
        .then()
            .statusCode(200)
            .body("users[0].name", equalTo("John Doe"))
            .body("users.name", hasItems("John Doe", "Jane Smith"));
    }
}
```

### Пример проверки с body() (TestNG)
```java
import io.restassured.http.ContentType;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;
import static org.hamcrest.Matchers.hasItems;

public class JsonValidationTest {
    @Test
    public void testBodyValidation() {
        given()
            .contentType(ContentType.JSON)
        .when()
            .get("https://api.example.com/v1/users")
        .then()
            .statusCode(200)
            .body("users[0].name", equalTo("John Doe"))
            .body("users.name", hasItems("John Doe", "Jane Smith"));
    }
}
```

### Пример проверки с jsonPath()
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.assertj.core.api.Assertions.assertThat;

public class JsonPathTest {
    @Test
    void testJsonPathValidation() {
        String name = given()
            .contentType(ContentType.JSON)
        .when()
            .get("https://api.example.com/v1/users/123")
        .then()
            .statusCode(200)
            .extract().jsonPath().getString("name");

        assertThat(name).isEqualTo("John Doe");
    }
}
```

**Примечания**:
- `body()` удобно для быстрого написания тестов, но ограничено Hamcrest-матчерами.
- `jsonPath()` даёт больше гибкости, позволяя извлекать данные для последующей обработки.
- Для мягких утверждений используйте AssertJ с `jsonPath()`.

## Использование JsonSchemaValidator для валидации схем

`JsonSchemaValidator` проверяет, соответствует ли JSON-ответ заданной схеме, что гарантирует правильность структуры данных. Это особенно полезно для API с динамическими данными.

### Настройка зависимости
Добавьте в `pom.xml`:
```xml
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>json-schema-validator</artifactId>
    <version>5.4.0</version>
    <scope>test</scope>
</dependency>
```

### Пример JSON-схемы (user-schema.json)
Создайте файл `src/test/resources/user-schema.json`:
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

### Пример проверки схемы (JUnit 5)
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

### Пример проверки схемы (TestNG)
```java
import io.restassured.http.ContentType;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static io.restassured.module.jsv.JsonSchemaValidator.matchesJsonSchemaInClasspath;

public class SchemaValidationTest {
    @Test
    public void testJsonSchema() {
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

**Примечания**:
- Храните схемы в `src/test/resources` для удобного доступа.
- Используйте онлайн-валидаторы JSON-схем для проверки корректности схемы перед тестами.
- Для сложных API создавайте отдельные схемы для разных эндпоинтов.

## Десериализация ответа в объекты (response.as(MyDto.class))

RestAssured поддерживает автоматическую десериализацию JSON-ответов в Java-объекты с использованием Jackson или Gson. Это упрощает работу с данными и позволяет использовать их в последующих проверках.

### Пример DTO
```java
import lombok.Data;

@Data
public class UserDto {
    private String id;
    private String name;
    private String email;
}
```

### Пример десериализации (JUnit 5)
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.assertj.core.api.Assertions.assertThat;

public class DeserializeTest {
    @Test
    void testDeserializeUser() {
        UserDto user = given()
            .contentType(ContentType.JSON)
        .when()
            .get("https://api.example.com/v1/users/123")
        .then()
            .statusCode(200)
            .extract().as(UserDto.class);

        assertThat(user.getName()).isEqualTo("John Doe");
        assertThat(user.getEmail()).isEqualTo("john@example.com");
    }
}
```

### Пример десериализации (TestNG)
```java
import io.restassured.http.ContentType;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static org.testng.Assert.assertEquals;

public class DeserializeTest {
    @Test
    public void testDeserializeUser() {
        UserDto user = given()
            .contentType(ContentType.JSON)
        .when()
            .get("https://api.example.com/v1/users/123")
        .then()
            .statusCode(200)
            .extract().as(UserDto.class);

        assertEquals(user.getName(), "John Doe");
        assertEquals(user.getEmail(), "john@example.com");
    }
}
```

### Настройка Jackson
Добавьте зависимость в `pom.xml`:
```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.17.2</version>
    <scope>test</scope>
</dependency>
```

**Примечания**:
- Убедитесь, что поля DTO совпадают с JSON-структурой. Используйте аннотации `@JsonProperty` для нестандартных имён.
- Для сложных JSON (например, вложенных объектов) создавайте соответствующие DTO.
- Проверяйте корректность десериализации с помощью AssertJ для мягких утверждений.

## Связь с клиент-серверной моделью

Работа с JSON напрямую связана с клиент-серверной архитектурой, где клиент (тест) отправляет запросы и получает JSON-ответы от сервера. `body()` и `jsonPath()` проверяют данные на стороне клиента, `JsonSchemaValidator` гарантирует соответствие структуры, а десериализация позволяет использовать данные в тестах, моделируя реальные сценарии работы приложения.

## Лучшие практики и отладка

- **Стабильность**: Используйте стабильные JSON-пути (например, `users[0].name` вместо индексов, если порядок элементов может меняться).
- **Логирование**: Добавьте `log().all()` для отладки JSON-ответов:
```java
given()
    .log().all()
.when()
    .get("/users/123")
.then()
    .log().all();
```

- **Отладка**: Используйте точки останова в IntelliJ IDEA для проверки JSON-путей или десериализованных объектов. Для валидации схем используйте онлайн-инструменты, такие как JSON Schema Validator.
- **Распространённые ошибки**:
  - Неправильные JSON-пути (проверяйте с помощью JSONPath Finder).
  - Несоответствие DTO и JSON (проверяйте наличие всех полей и их типов).
  - Ошибки в схемах (например, отсутствие `required` для обязательных полей).

## Источники
- [RestAssured Official Documentation](http://rest-assured.io/)
- [JSON Schema Validator Documentation](https://github.com/rest-assured/rest-assured/wiki/Usage#json-schema-validation)
- [Jackson Documentation](https://github.com/FasterXML/jackson)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [TestNG Documentation](https://testng.org/doc/)

---
[**← Назад к оглавлению**](../README.md)