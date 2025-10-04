# Использование DTO и POJO

DTO (Data Transfer Object) и POJO (Plain Old Java Object) широко используются в API-тестах для представления данных запросов и ответов. Они упрощают работу с JSON/XML, обеспечивают типизацию и улучшают читаемость тестов. В этой статье рассматриваются создание моделей для тела запроса и ответа, использование `ObjectMapper` для сериализации/десериализации и упрощение тестов с помощью типизированных объектов.

## Создание моделей для тела запроса и ответа

DTO/POJO представляют структуру данных API в виде Java-классов. Использование библиотеки Lombok сокращает шаблонный код (геттеры, сеттеры, toString). Модели создаются для запросов (например, отправка данных в POST) и ответов (десериализация ответа сервера).

### Пример DTO с Lombok
```java
import lombok.Data;

@Data
public class UserDto {
    private String id;
    private String name;
    private String email;
}
```

### Использование DTO в запросе (JUnit 5)
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class UserApiTest {
    @Test
    void testCreateUser() {
        UserDto user = new UserDto();
        user.setName("Jane Smith");
        user.setEmail("jane@example.com");

        given()
            .contentType(ContentType.JSON)
            .body(user)
        .when()
            .post("https://api.example.com/v1/users")
        .then()
            .statusCode(201)
            .body("email", equalTo("jane@example.com"));
    }
}
```

### Использование DTO для ответа
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.assertj.core.api.Assertions.assertThat;

public class UserResponseTest {
    @Test
    void testGetUser() {
        UserDto user = given()
            .contentType(ContentType.JSON)
        .when()
            .get("https://api.example.com/v1/users/123")
        .then()
            .statusCode(200)
            .extract().as(UserDto.class);

        assertThat(user.getName()).isEqualTo("Jane Smith");
        assertThat(user.getEmail()).isEqualTo("jane@example.com");
    }
}
```

### Пример для TestNG
```java
import io.restassured.http.ContentType;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static org.testng.Assert.assertEquals;

public class UserApiTest {
    @Test
    public void testCreateUser() {
        UserDto user = new UserDto();
        user.setName("Jane Smith");
        user.setEmail("jane@example.com");

        given()
            .contentType(ContentType.JSON)
            .body(user)
        .when()
            .post("https://api.example.com/v1/users")
        .then()
            .statusCode(201)
            .body("email", equalTo("jane@example.com"));
    }

    @Test
    public void testGetUser() {
        UserDto user = given()
            .contentType(ContentType.JSON)
        .when()
            .get("https://api.example.com/v1/users/123")
        .then()
            .statusCode(200)
            .extract().as(UserDto.class);

        assertEquals(user.getName(), "Jane Smith");
        assertEquals(user.getEmail(), "jane@example.com");
    }
}
```

**Примечания**:
- Поля DTO должны соответствовать структуре JSON (имя и тип данных).
- Используйте аннотации `@JsonProperty` для нестандартных имён полей:
```java
import com.fasterxml.jackson.annotation.JsonProperty;
import lombok.Data;

@Data
public class UserDto {
    @JsonProperty("user_id")
    private String id;
    private String name;
    private String email;
}
```

## Использование ObjectMapper для сериализации

`ObjectMapper` из библиотеки Jackson используется для явной сериализации (объект → JSON) и десериализации (JSON → объект). Это полезно для сложных сценариев, когда нужно обработать данные перед отправкой или после получения.

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

### Пример сериализации и десериализации
```java
import com.fasterxml.jackson.databind.ObjectMapper;
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.assertj.core.api.Assertions.assertThat;

public class ObjectMapperTest {
    private final ObjectMapper mapper = new ObjectMapper();

    @Test
    void testSerializeAndSend() throws Exception {
        UserDto user = new UserDto();
        user.setName("Jane Smith");
        user.setEmail("jane@example.com");

        String jsonBody = mapper.writeValueAsString(user);

        given()
            .contentType(ContentType.JSON)
            .body(jsonBody)
        .when()
            .post("https://api.example.com/v1/users")
        .then()
            .statusCode(201)
            .body("email", equalTo("jane@example.com"));
    }

    @Test
    void testDeserializeResponse() throws Exception {
        String response = given()
            .contentType(ContentType.JSON)
        .when()
            .get("https://api.example.com/v1/users/123")
        .then()
            .statusCode(200)
            .extract().asString();

        UserDto user = mapper.readValue(response, UserDto.class);
        assertThat(user.getName()).isEqualTo("Jane Smith");
    }
}
```

**Примечания**:
- `ObjectMapper` автоматически используется RestAssured для `body(dto)` и `extract().as(dto.class)`.
- Для обработки ошибок сериализации добавляйте try-catch:
```java
try {
    String jsonBody = mapper.writeValueAsString(user);
} catch (JsonProcessingException e) {
    throw new RuntimeException("Serialization failed", e);
}
```

## Упрощение тестов за счёт typed-объектов

Типизированные объекты (DTO/POJO) упрощают тесты, предоставляя следующие преимущества:
- **Читаемость**: Код становится понятнее, так как данные представлены в виде объектов, а не строк JSON.
- **Типобезопасность**: Компилятор проверяет корректность типов и полей.
- **Повторное использование**: DTO можно использовать в разных тестах для запросов и ответов.
- **Простота валидации**: AssertJ или TestNG-утверждения работают с объектами напрямую.

### Пример упрощённого теста
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.assertj.core.api.Assertions.assertThat;

public class TypedObjectTest {
    @Test
    void testTypedRequestAndResponse() {
        UserDto request = new UserDto();
        request.setName("Jane Smith");
        request.setEmail("jane@example.com");

        UserDto response = given()
            .contentType(ContentType.JSON)
            .body(request)
        .when()
            .post("https://api.example.com/v1/users")
        .then()
            .statusCode(201)
            .extract().as(UserDto.class);

        assertThat(response.getName()).isEqualTo(request.getName());
        assertThat(response.getEmail()).isEqualTo(request.getEmail());
    }
}
```

## Связь с клиент-серверной моделью

DTO/POJO моделируют данные, которыми обмениваются клиент (тест) и сервер. Они упрощают отправку запросов (сериализация) и обработку ответов (десериализация), обеспечивая структурированное взаимодействие в клиент-серверной архитектуре.

## Лучшие практики и отладка

- **Стабильность**: Убедитесь, что DTO полностью соответствует JSON-структуре. Используйте аннотации `@JsonProperty` для нестандартных имён.
- **Логирование**: Логируйте JSON для отладки:
```java
given()
    .log().all()
    .body(dto)
.when()
    .post("/users")
.then()
    .log().all();
```

- **Отладка**: Используйте IntelliJ IDEA для проверки значений DTO перед сериализацией. Для анализа JSON используйте JSONPath Finder или онлайн-валидаторы.
- **Распространённые ошибки**:
  - Несоответствие полей DTO и JSON (проверяйте документацию API).
  - Отсутствие геттеров/сеттеров (используйте Lombok или добавляйте вручную).
  - Ошибки сериализации из-за неверных типов данных (например, `String` вместо `Integer`).

## Источники
- [RestAssured Official Documentation](http://rest-assured.io/)
- [Jackson Documentation](https://github.com/FasterXML/jackson)
- [Lombok Documentation](https://projectlombok.org/)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [TestNG Documentation](https://testng.org/doc/)

---
[**← Назад к оглавлению**](../README.md)