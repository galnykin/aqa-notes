# Обработка исключений в тестах

Обработка исключений в автоматизированных тестах с использованием TestNG позволяет проверять поведение системы в ошибочных сценариях, таких как некорректные данные или недоступность ресурсов. Аннотация `expectedExceptions` явно указывает ожидаемые исключения, а правильное логирование и проверка сообщений исключений повышают информативность тестов. Эта статья дополняет предыдущие материалы, фокусируясь на использовании `expectedExceptions`, логировании причин падения и проверке сообщений исключений.

## Использование `expectedExceptions`

Аннотация `expectedExceptions` в TestNG позволяет указать, что тест ожидает определённое исключение. Если указанное исключение не возникает, тест считается проваленным. Это полезно для проверки негативных сценариев, таких как ввод некорректных данных или недоступность сервиса.

### Пример кода (TestNG с `expectedExceptions`)

```java
import io.qameta.allure.Description;
import org.testng.annotations.Test;
import static org.testng.Assert.assertTrue;

public class ExceptionTest extends BaseTest {
    @Test(expectedExceptions = IllegalArgumentException.class)
    @Description("Проверка обработки некорректного имени пользователя")
    public void testInvalidUsernameThrowsException() {
        // Arrange
        UserService userService = new UserService();

        // Act
        userService.createUserWithInvalidUsername("");
    }

    @Test(expectedExceptions = IllegalStateException.class)
    @Description("Проверка недоступности сервиса")
    public void testServiceUnavailableThrowsException() {
        // Arrange
        ServiceClient serviceClient = new ServiceClient();

        // Act
        serviceClient.callUnavailableService();
    }
}
```

### Пример реализации сервиса

```java
public class UserService {
    public void createUserWithInvalidUsername(String username) {
        if (username == null || username.isEmpty()) {
            throw new IllegalArgumentException("Username cannot be empty");
        }
        // Логика создания пользователя
    }
}

public class ServiceClient {
    public void callUnavailableService() {
        // Симуляция недоступности сервиса
        throw new IllegalStateException("Service is unavailable");
    }
}
```

### Пояснения
- **`expectedExceptions`**: Указывает, что тест ожидает `IllegalArgumentException` или `IllegalStateException`. Если исключение не возникает, тест проваливается.
- **Назначение**: Проверяет, что система корректно обрабатывает ошибочные сценарии.
- **Allure**: Аннотация `@Description` добавляет контекст для отчётов.

### Лучшие практики
- **Конкретные исключения**: Указывайте конкретные классы исключений (например, `IllegalArgumentException`) вместо общего `Exception`.
- **Минимальная область**: Применяйте `expectedExceptions` только для негативных сценариев, где исключение ожидаемо.
- **Комбинирование с другими проверками**: Если нужно проверить сообщение исключения, используйте дополнительные механизмы (см. ниже).

## Логирование причины падения

Логирование с использованием SLF4J и Logback помогает фиксировать причины исключений, упрощая отладку.

### Пример кода (TestNG с логированием)

```java
import io.qameta.allure.Description;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testng.annotations.Test;

public class ExceptionLoggingTest extends BaseTest {
    private static final Logger logger = LoggerFactory.getLogger(ExceptionLoggingTest.class);

    @Test(expectedExceptions = IllegalArgumentException.class)
    @Description("Проверка и логирование некорректного имени пользователя")
    public void testInvalidUsernameWithLogging() {
        // Arrange
        UserService userService = new UserService();

        // Act
        try {
            userService.createUserWithInvalidUsername(null);
        } catch (IllegalArgumentException e) {
            logger.error("Expected exception caught: {}", e.getMessage());
            throw e; // Повторно выбрасываем, чтобы TestNG обработал
        }
    }
}
```

### Пояснения
- **Логирование**: Используется `logger.error` для записи сообщения исключения.
- **Повторное выбрасывание**: Исключение перебрасывается, чтобы TestNG зарегистрировал его как ожидаемое.
- **Зависимости в `pom.xml`**:

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>2.0.16</version>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.5.12</version>
</dependency>
```

## Проверка сообщений исключений

Если важно проверить не только тип исключения, но и его сообщение, используйте блок `try-catch` с AssertJ для мягких утверждений.

### Пример кода (TestNG с проверкой сообщения)

```java
import io.qameta.allure.Description;
import org.testng.annotations.Test;
import static org.assertj.core.api.Assertions.assertThatThrownBy;

public class ExceptionMessageTest extends BaseTest {
    private static final Logger logger = LoggerFactory.getLogger(ExceptionMessageTest.class);

    @Test
    @Description("Проверка сообщения об ошибке для некорректного имени пользователя")
    public void testInvalidUsernameExceptionMessage() {
        // Arrange
        UserService userService = new UserService();

        // Act & Assert
        assertThatThrownBy(() -> userService.createUserWithInvalidUsername(""))
                .isInstanceOf(IllegalArgumentException.class)
                .hasMessage("Username cannot be empty");

        // Логирование
        try {
            userService.createUserWithInvalidUsername("");
        } catch (IllegalArgumentException e) {
            logger.error("Caught exception with message: {}", e.getMessage());
        }
    }
}
```

### Пояснения
- **AssertJ**: Используется `assertThatThrownBy` для проверки типа исключения и его сообщения.
- **Логирование**: Дополнительный блок `try-catch` фиксирует сообщение исключения.
- **Зависимость в `pom.xml`**:

```xml
<dependency>
    <groupId>org.assertj</groupId>
    <artifactId>assertj-core</artifactId>
    <version>3.26.3</version>
    <scope>test</scope>
</dependency>
```

### Лучшие практики
- **Проверка сообщений**: Используйте AssertJ для точной проверки текста исключения, особенно если он важен для теста.
- **Логирование**: Всегда логируйте исключения, чтобы упростить отладку.
- **Избегайте дублирования**: Если сообщение не критично, достаточно `expectedExceptions`.

## Интеграция с CI/CD

Для запуска тестов с обработкой исключений настройте Maven и TestNG в CI/CD.

### Пример настройки GitHub Actions

```yaml
name: Run Tests with Exception Handling
on:
  push:
    branches:
      - main
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
      - name: Run tests
        run: mvn test -Dtestng.dtd.http=true
      - name: Generate Allure report
        run: mvn allure:report
      - name: Publish Allure report
        uses: actions/upload-artifact@v4
        with:
          name: allure-report
          path: target/site/allure-maven-plugin
```

### Пример `testng.xml`

```xml
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="ExceptionTestSuite">
    <test name="ExceptionTests">
        <classes>
            <class name="ExceptionTest"/>
            <class name="ExceptionLoggingTest"/>
            <class name="ExceptionMessageTest"/>
        </classes>
    </test>
</suite>
```

## Дополнительно

### Интеграция с Allure

Allure отображает информацию об ожидаемых исключениях в отчётах.

```java
import io.qameta.allure.Description;
import io.qameta.allure.Step;
import org.testng.annotations.Test;

public class AllureExceptionTest extends BaseTest {
    private static final Logger logger = LoggerFactory.getLogger(AllureExceptionTest.class);

    @Test(expectedExceptions = IllegalArgumentException.class)
    @Description("Проверка обработки некорректного ввода")
    public void testInvalidInputThrowsException() {
        // Arrange
        UserService userService = new UserService();

        // Act
        validateInput(userService, null);
    }

    @Step("Валидация ввода: {username}")
    private void validateInput(UserService userService, String username) {
        try {
            userService.createUserWithInvalidUsername(username);
        } catch (IllegalArgumentException e) {
            logger.error("Caught exception: {}", e.getMessage());
            throw e;
        }
    }
}
```

### Использование Testcontainers

Testcontainers можно использовать для проверки исключений, связанных с недоступностью сервисов.

```java
import org.testcontainers.containers.GenericContainer;
import org.testng.annotations.BeforeClass;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;

public class ContainerExceptionTest extends BaseTest {
    private GenericContainer<?> apiContainer;

    @BeforeClass
    public void setupContainer() {
        apiContainer = new GenericContainer<>("my-api-image:latest")
                .withExposedPorts(8080);
        apiContainer.start();
        RestAssured.baseURI = "http://" + apiContainer.getHost() + ":" + apiContainer.getMappedPort(8080);
    }

    @Test(expectedExceptions = RuntimeException.class)
    @Description("Проверка ошибки при недоступном API")
    public void testApiFailureThrowsException() {
        // Arrange
        apiContainer.stop(); // Симуляция сбоя

        // Act
        given()
                .when()
                .get("/api/health")
                .then()
                .statusCode(200); // Ожидаем сбой
    }
}
```

### Проверка качества кода

Используйте PMD для анализа обработки исключений. Пример конфигурации в `pom.xml`:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-pmd-plugin</artifactId>
    <version>3.23.0</version>
    <configuration>
        <rulesets>
            <ruleset>rulesets/java/errorprone.xml</ruleset>
        </rulesets>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

Запуск:
```bash
mvn pmd:check
```

### Отладка

Для отладки в IntelliJ IDEA:
1. Установите точки останова в блоках `try-catch` или в методах, выбрасывающих исключения.
2. Используйте TestNG Run/Debug Configuration для запуска тестов.
3. Проверяйте логи SLF4J и сообщения исключений в Variables.

## Распространённые ошибки

- **Слишком общие исключения**: Указывайте конкретные исключения в `expectedExceptions`, а не `Exception`.
- **Отсутствие логирования**: Всегда логируйте причину исключения для упрощения отладки.
- **Игнорирование сообщений**: Если сообщение исключения важно, проверяйте его с помощью AssertJ.

## Источники
- [TestNG Documentation](https://testng.org/doc/documentation-main.html#annotations)
- [AssertJ Documentation](https://assertj.github.io/doc/)
- [Allure Framework](https://allurereport.org/docs/)
- [Testcontainers Documentation](https://testcontainers.org/)
- [PMD Documentation](https://pmd.github.io/)

---
[**← Назад к оглавлению**](../README.md)