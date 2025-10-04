# Тестовые данные и фикстуры

Эффективное управление тестовыми данными и фикстурами в API-тестах упрощает их поддержку и повышает воспроизводимость. Хранение данных в JSON, YAML или Excel позволяет централизовать их, а использование фабрик данных (например, `UserFactory`) обеспечивает гибкость в создании тестовых сценариев. Переиспользуемые шаблоны запросов минимизируют дублирование кода. В этой статье рассматриваются подходы к хранению данных, генерации фикстур и созданию шаблонов для типовых запросов.

## Хранение данных в JSON/YAML/Excel

Тестовые данные можно хранить в файлах JSON, YAML или Excel для удобного доступа и управления. Это позволяет отделить данные от кода тестов, упрощая их обновление и переиспользование.

### Хранение данных в JSON
JSON-файлы удобны для хранения структурированных данных, таких как объекты или списки.

#### Пример JSON-файла (test-data/users.json)
```json
{
  "users": [
    {
      "id": "123",
      "name": "John Doe",
      "email": "john@example.com"
    },
    {
      "id": "456",
      "name": "Jane Smith",
      "email": "jane@example.com"
    }
  ]
}
```

#### Чтение JSON в тесте
Добавьте зависимость Jackson в `pom.xml`:
```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.17.2</version>
    <scope>test</scope>
</dependency>
```

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.Data;
import org.junit.jupiter.api.Test;
import java.io.IOException;
import java.util.List;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

@Data
class UserDto {
    private String id;
    private String name;
    private String email;
}

@Data
class UsersWrapper {
    private List<UserDto> users;
}

public class JsonDataTest {
    private final ObjectMapper mapper = new ObjectMapper();

    @Test
    void testWithJsonData() throws IOException {
        UsersWrapper users = mapper.readValue(
            getClass().getClassLoader().getResourceAsStream("test-data/users.json"),
            UsersWrapper.class
        );

        UserDto user = users.getUsers().get(0);

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

### Хранение данных в YAML
YAML более читаем для сложных структур. Используйте библиотеку SnakeYAML.

#### Пример YAML-файла (test-data/users.yaml)
```yaml
users:
  - id: "123"
    name: "John Doe"
    email: "john@example.com"
  - id: "456"
    name: "Jane Smith"
    email: "jane@example.com"
```

#### Настройка SnakeYAML
```xml
<dependency>
    <groupId>org.yaml</groupId>
    <artifactId>snakeyaml</artifactId>
    <version>2.2</version>
    <scope>test</scope>
</dependency>
```

#### Чтение YAML в тесте
```java
import org.yaml.snakeyaml.Yaml;
import lombok.Data;
import org.junit.jupiter.api.Test;
import java.io.InputStream;
import java.util.List;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

@Data
class UserDto {
    private String id;
    private String name;
    private String email;
}

@Data
class UsersWrapper {
    private List<UserDto> users;
}

public class YamlDataTest {
    @Test
    void testWithYamlData() {
        Yaml yaml = new Yaml();
        InputStream inputStream = getClass().getClassLoader().getResourceAsStream("test-data/users.yaml");
        UsersWrapper users = yaml.loadAs(inputStream, UsersWrapper.class);

        UserDto user = users.getUsers().get(0);

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

### Хранение данных в Excel
Excel подходит для табличных данных. Используйте Apache POI для чтения.

#### Настройка Apache POI
```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>5.2.5</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.2.5</version>
    <scope>test</scope>
</dependency>
```

#### Пример Excel-файла (test-data/users.xlsx)
| id  | name      | email              |
|-----|-----------|--------------------|
| 123 | John Doe  | john@example.com   |
| 456 | Jane Smith| jane@example.com   |

#### Чтение Excel в тесте
```java
import org.apache.poi.ss.usermodel.*;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class ExcelDataTest {
    @Test
    void testWithExcelData() throws Exception {
        Workbook workbook = WorkbookFactory.create(
            getClass().getClassLoader().getResourceAsStream("test-data/users.xlsx")
        );
        Sheet sheet = workbook.getSheetAt(0);
        Row row = sheet.getRow(1); // Первая строка после заголовков
        UserDto user = new UserDto();
        user.setId(row.getCell(0).getStringCellValue());
        user.setName(row.getCell(1).getStringCellValue());
        user.setEmail(row.getCell(2).getStringCellValue());

        given()
            .contentType("application/json")
            .body(user)
        .when()
            .post("https://api.example.com/v1/users")
        .then()
            .statusCode(201)
            .body("email", equalTo(user.getEmail()));

        workbook.close();
    }
}
```

## Генерация данных через фабрики

Фабрики данных (например, `UserFactory`) позволяют генерировать тестовые данные программно, что полезно для создания валидных и невалидных сценариев. Это уменьшает зависимость от статических файлов.

### Пример UserFactory
```java
import lombok.Data;

@Data
public class UserDto {
    private String id;
    private String name;
    private String email;
}

public class UserFactory {
    public static UserDto createValidUser() {
        UserDto user = new UserDto();
        user.setId("123");
        user.setName("John Doe");
        user.setEmail("john@example.com");
        return user;
    }

    public static UserDto createInvalidUser() {
        UserDto user = new UserDto();
        user.setId("999");
        user.setName(""); // Пустое имя
        user.setEmail("invalid"); // Некорректный email
        return user;
    }
}
```

### Использование фабрики в тесте (JUnit 5)
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class UserFactoryTest {
    @Test
    void testValidUser() {
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

    @Test
    void testInvalidUser() {
        UserDto user = UserFactory.createInvalidUser();

        given()
            .contentType(ContentType.JSON)
            .body(user)
        .when()
            .post("https://api.example.com/v1/users")
        .then()
            .statusCode(400)
            .body("error.message", equalTo("Invalid email format"));
    }
}
```

### Использование фабрики в тесте (TestNG)
```java
import io.restassured.http.ContentType;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class UserFactoryTest {
    @Test
    public void testValidUser() {
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

    @Test
    public void testInvalidUser() {
        UserDto user = UserFactory.createInvalidUser();

        given()
            .contentType(ContentType.JSON)
            .body(user)
        .when()
            .post("https://api.example.com/v1/users")
        .then()
            .statusCode(400)
            .body("error.message", equalTo("Invalid email format"));
    }
}
```

## Переиспользуемые шаблоны для типовых запросов

Шаблоны запросов, реализованные через `RequestSpecification`, позволяют унифицировать типовые запросы, такие как создание пользователя или получение данных.

### Пример шаблона запроса
```java
import io.restassured.builder.RequestSpecBuilder;
import io.restassured.http.ContentType;
import io.restassured.specification.RequestSpecification;

public class RequestTemplates {
    public static RequestSpecification createUserSpec(UserDto user) {
        return new RequestSpecBuilder()
                .setBaseUri("https://api.example.com")
                .setBasePath("/v1/users")
                .setContentType(ContentType.JSON)
                .setBody(user)
                .build();
    }

    public static RequestSpecification getUserSpec(String userId) {
        return new RequestSpecBuilder()
                .setBaseUri("https://api.example.com")
                .setBasePath("/v1/users")
                .setContentType(ContentType.JSON)
                .addPathParam("userId", userId)
                .build();
    }
}
```

### Использование шаблона в тесте
```java
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class TemplateTest {
    @Test
    void testCreateUserTemplate() {
        UserDto user = UserFactory.createValidUser();

        given()
            .spec(RequestTemplates.createUserSpec(user))
        .when()
            .post()
        .then()
            .statusCode(201)
            .body("email", equalTo(user.getEmail()));
    }

    @Test
    void testGetUserTemplate() {
        given()
            .spec(RequestTemplates.getUserSpec("123"))
        .when()
            .get("/{userId}")
        .then()
            .statusCode(200)
            .body("name", equalTo("John Doe"));
    }
}
```

## Связь с клиент-серверной моделью

Тестовые данные и фикстуры моделируют данные, отправляемые клиентом (тестом) на сервер, и ответы, которые клиент ожидает. JSON/YAML/Excel обеспечивают структурированное хранение данных, фабрики упрощают их генерацию, а шаблоны запросов унифицируют взаимодействие с сервером.

## Лучшие практики и отладка

- **Стабильность**: Храните тестовые данные в `src/test/resources` и используйте относительные пути. Проверяйте корректность данных в файлах.
- **Логирование**: Логируйте данные перед отправкой:
```java
given()
    .log().all()
    .spec(RequestTemplates.createUserSpec(user))
.when()
    .post();
```

- **Отладка**: Используйте IntelliJ IDEA для проверки значений DTO или данных из файлов. Для JSON/YAML используйте онлайн-валидаторы.
- **Распространённые ошибки**:
  - Несоответствие структуры данных в файлах и DTO (проверяйте JSON/YAML).
  - Ошибки чтения файлов (убедитесь, что файлы находятся в `src/test/resources`).
  - Неправильные шаблоны запросов (проверяйте параметры в `RequestSpecification`).

## Источники
- [RestAssured Official Documentation](http://rest-assured.io/)
- [Jackson Documentation](https://github.com/FasterXML/jackson)
- [SnakeYAML Documentation](https://bitbucket.org/snakeyaml/snakeyaml/wiki/Home)
- [Apache POI Documentation](https://poi.apache.org/)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [TestNG Documentation](https://testng.org/doc/)

---
[**← Назад к оглавлению**](../README.md)