# Параметризация запросов

Параметризация запросов в RestAssured позволяет гибко настраивать HTTP-запросы, включая параметры пути, запроса, заголовки, куки и тело. Использование DTO/POJO для формирования тела запроса упрощает работу с данными, а поддержка сериализации через Jackson или Gson обеспечивает преобразование объектов в JSON/XML. В этой статье рассматриваются способы параметризации, создание DTO, их сериализация и лучшие практики.

## pathParam, queryParam, header, cookie, body

RestAssured предоставляет методы для добавления различных типов параметров в запросы:
- **pathParam**: Для динамических частей URL (например, `/users/{id}`).
- **queryParam**: Для параметров запроса в строке URL (например, `?key=value`).
- **header**: Для HTTP-заголовков (например, `Authorization`, `Content-Type`).
- **cookie**: Для добавления cookies в запрос.
- **body**: Для передачи данных в теле запроса (JSON, XML и др.).

### Пример параметризации (JUnit 5)
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class ParametrizedApiTest {
    @Test
    void testParametrizedRequest() {
        String requestBody = "{ \"name\": \"Jane Smith\", \"email\": \"jane@example.com\" }";

        given()
            .pathParam("userId", "123")
            .queryParam("filter", "active")
            .header("Authorization", "Bearer token123")
            .cookie("session", "abc123")
            .contentType(ContentType.JSON)
            .body(requestBody)
        .when()
            .post("/users/{userId}")
        .then()
            .statusCode(201)
            .body("email", equalTo("jane@example.com"));
    }
}
```

### Пример параметризации (TestNG)
```java
import io.restassured.http.ContentType;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class ParametrizedApiTest {
    @Test
    public void testParametrizedRequest() {
        String requestBody = "{ \"name\": \"Jane Smith\", \"email\": \"jane@example.com\" }";

        given()
            .pathParam("userId", "123")
            .queryParam("filter", "active")
            .header("Authorization", "Bearer token123")
            .cookie("session", "abc123")
            .contentType(ContentType.JSON)
            .body(requestBody)
        .when()
            .post("/users/{userId}")
        .then()
            .statusCode(201)
            .body("email", equalTo("jane@example.com"));
    }
}
```

## Использование DTO/POJO для тела запроса

DTO (Data Transfer Object) или POJO (Plain Old Java Object) используются для представления структуры данных в теле запроса. Это упрощает создание и валидацию данных, а также улучшает читаемость кода. Для работы с DTO рекомендуется использовать библиотеку Lombok для сокращения шаблонного кода.

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

### Использование DTO в тесте
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class UserApiTest {
    @Test
    void testCreateUserWithDto() {
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

## Поддержка сериализации через Jackson/Gson

RestAssured автоматически использует Jackson или Gson для сериализации и десериализации объектов в JSON. Jackson включён по умолчанию, но можно настроить Gson, если требуется.

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

### Пример сериализации с Jackson
```java
import com.fasterxml.jackson.databind.ObjectMapper;
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class JacksonApiTest {
    @Test
    void testSerializeUser() throws Exception {
        UserDto user = new UserDto();
        user.setName("Jane Smith");
        user.setEmail("jane@example.com");

        ObjectMapper mapper = new ObjectMapper();
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
}
```

### Настройка Gson
Добавьте зависимость в `pom.xml`:
```xml
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.11.0</version>
    <scope>test</scope>
</dependency>
```

### Пример сериализации с Gson
```java
import com.google.code.gson.Gson;
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class GsonApiTest {
    @Test
    void testSerializeUser() {
        UserDto user = new UserDto();
        user.setName("Jane Smith");
        user.setEmail("jane@example.com");

        Gson gson = new Gson();
        String jsonBody = gson.toJson(user);

        given()
            .contentType(ContentType.JSON)
            .body(jsonBody)
        .when()
            .post("https://api.example.com/v1/users")
        .then()
            .statusCode(201)
            .body("email", equalTo("jane@example.com"));
    }
}
```

### Десериализация ответа
Для получения ответа в виде объекта используйте `extract().as()`:
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.assertj.core.api.Assertions.assertThat;

public class DeserializeApiTest {
    @Test
    void testDeserializeUser() {
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

## Связь с клиент-серверной моделью

Параметризация запросов моделирует поведение клиента, отправляющего запросы к серверу. `pathParam` и `queryParam` задают параметры URL, `header` и `cookie` обеспечивают аутентификацию и сессии, а `body` передаёт данные для обработки сервером. DTO/POJO упрощают работу с данными на стороне клиента, а сериализация через Jackson/Gson обеспечивает корректный формат обмена данными (JSON/XML).

## Лучшие практики и отладка

- **Стабильность**: Используйте стабильные `pathParam` (например, ID вместо имени) и проверяйте обязательные заголовки в `RequestSpecification`.
- **Логирование**: Добавьте `log().all()` для отладки параметров:
```java
given()
    .pathParam("userId", "123")
    .log().all()
.when()
    .get("/users/{userId}")
.then()
    .log().all();
```

- **Отладка**: В IntelliJ IDEA используйте точки останова для проверки значений DTO перед сериализацией. Для JSON-путей используйте JSONPath Finder.
- **Распространённые ошибки**:
  - Неправильные `pathParam` или `queryParam` (проверяйте документацию API).
  - Ошибки в сериализации (например, отсутствие геттеров/сеттеров в DTO).
  - Неправильный формат тела запроса (проверяйте `Content-Type`).

## Источники
- [RestAssured Official Documentation](http://rest-assured.io/)
- [Jackson Documentation](https://github.com/FasterXML/jackson)
- [Gson Documentation](https://github.com/google/gson)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [TestNG Documentation](https://testng.org/doc/)

---
[**← Назад к оглавлению**](../README.md)