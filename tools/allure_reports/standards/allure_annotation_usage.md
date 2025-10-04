# Использование аннотаций Allure

Аннотации Allure структурируют тесты и отчёты, обеспечивая прозрачность, удобство анализа и фильтрацию. В статье описывается использование аннотаций `@Epic`, `@Feature`, `@Story`, `@Owner`, `@Tag` и `@Severity` для UI и API тестов. Аннотации `@Epic`, `@Feature` и `@Story` обязательны для всех тестов, чтобы обеспечить единообразную структуру. Используется технологический стек: Java 17, Maven, SLF4J с Logback, JUnit 5, TestNG, Selenium WebDriver, RestAssured, Allure и Jenkins.

## Использование аннотаций

### @Epic, @Feature, @Story

Эти аннотации формируют иерархию тестов, делая отчёты понятными для всех участников (QA, разработчики, бизнес).

- **@Epic**: Крупный бизнес-блок, например, «Авторизация» или «Оплата». Указывает общий контекст модуля.
- **@Feature**: Функциональный раздел внутри эпика, например, «Модальное окно входа» или «API получения данных».
- **@Story**: Конкретный пользовательский сценарий, например, «Успешный вход» или «Неверный формат email».

**Требование**: Все UI и API тесты должны использовать `@Epic`, `@Feature` и `@Story` для единообразной структуры.

### @Owner

Аннотация `@Owner` указывает ответственного за тест, что упрощает коммуникацию и распределение задач.

- **Назначение**: Определение ответственного (QA-инженер, разработчик).
- **Применение**: Указывается как строка, например, `@Owner("qa_team")`.

### @Tag

Аннотация `@Tag` используется для группировки тестов по типу, например, `ui`, `api`, `regression`, `smoke`.

- **Назначение**: Упрощает фильтрацию тестов в отчётах.
- **Применение**: Можно задавать несколько тегов, например, `@Tag("ui")` и `@Tag("regression")`.

### @Severity

Аннотация `@Severity` определяет критичность теста: `BLOCKER`, `CRITICAL`, `NORMAL`, `MINOR`, `TRIVIAL`.

- **Назначение**: Помогает выделить важные тесты в отчётах.
- **Применение**: Используется для фильтрации, например, запуск только `CRITICAL` тестов.

**Уровни критичности**:
- `BLOCKER`: Блокирует основную функциональность.
- `CRITICAL`: Затрагивает ключевые функции, но есть обходной путь.
- `NORMAL`: Стандартный тест.
- `MINOR`: Незначительные проблемы.
- `TRIVIAL`: Косметические дефекты.

## Примеры реализации

### UI-тест с аннотациями (JUnit 5)

```java
import io.qameta.allure.Description;
import io.qameta.allure.Epic;
import io.qameta.allure.Feature;
import io.qameta.allure.Owner;
import io.qameta.allure.Severity;
import io.qameta.allure.SeverityLevel;
import io.qameta.allure.Step;
import io.qameta.allure.Story;
import io.qameta.allure.Tag;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import static org.assertj.core.api.Assertions.assertThat;

public class UiAllureTest {
    private static final Logger logger = LoggerFactory.getLogger(UiAllureTest.class);
    private WebDriver driver;

    @BeforeEach
    void setUp() {
        logger.info("Инициализация UI-теста");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
    }

    @Test
    @Epic("Авторизация")
    @Feature("Модальное окно входа")
    @Story("Успешный вход")
    @Owner("qa_team")
    @Tag("ui")
    @Tag("smoke")
    @Severity(SeverityLevel.CRITICAL)
    @Description("Проверка успешного входа через модальное окно")
    void testSuccessfulLogin() {
        openLoginPage();
        enterCredentials("user", "password");
        verifyLoginSuccess();
    }

    @Step("Открытие страницы логина")
    private void openLoginPage() {
        logger.info("Открытие страницы: https://example.com/login");
        driver.get("https://example.com/login");
    }

    @Step("Ввод учетных данных: username={username}, password={password}")
    private void enterCredentials(String username, String password) {
        logger.info("Ввод учетных данных: username={}, password=****", username);
        // driver.findElement(By.id("username")).sendKeys(username);
        // driver.findElement(By.id("password")).sendKeys(password);
        // driver.findElement(By.id("loginButton")).click();
    }

    @Step("Проверка успешного входа")
    private void verifyLoginSuccess() {
        logger.info("Проверка URL после входа");
        assertThat(driver.getCurrentUrl()).contains("dashboard");
    }

    @AfterEach
    void tearDown() {
        logger.info("Завершение UI-теста");
        if (driver != null) {
            driver.quit();
        }
    }
}
```

### API-тест с аннотациями (TestNG)

```java
import io.qameta.allure.Description;
import io.qameta.allure.Epic;
import io.qameta.allure.Feature;
import io.qameta.allure.Owner;
import io.qameta.allure.Severity;
import io.qameta.allure.SeverityLevel;
import io.qameta.allure.Step;
import io.qameta.allure.Story;
import io.qameta.allure.Tag;
import io.restassured.RestAssured;
import io.restassured.response.Response;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testng.annotations.Test;

import static io.restassured.RestAssured.given;
import static org.testng.Assert.assertEquals;

public class ApiAllureTest {
    private static final Logger logger = LoggerFactory.getLogger(ApiAllureTest.class);

    @Test
    @Epic("API-тестирование")
    @Feature("Получение данных пользователя")
    @Story("Проверка ответа API")
    @Owner("api_team")
    @Tag("api")
    @Tag("regression")
    @Severity(SeverityLevel.NORMAL)
    @Description("Проверка получения данных пользователя через API")
    void testApiUserData() {
        sendGetRequest();
        verifyResponseStatus();
    }

    @Step("Отправка GET-запроса к /users/1")
    private Response sendGetRequest() {
        logger.info("Отправка GET-запроса: /users/1");
        RestAssured.baseURI = "https://api.example.com";
        return given()
                .when()
                .get("/users/1")
                .then()
                .extract().response();
    }

    @Step("Проверка статуса ответа")
    private void verifyResponseStatus() {
        Response response = sendGetRequest();
        logger.info("Статус ответа: {}", response.getStatusCode());
        assertEquals(response.getStatusCode(), 200, "Ожидался статус 200");
    }
}
```

## Зависимости Maven

```xml
<dependency>
    <groupId>io.qameta.allure</groupId>
    <artifactId>allure-junit5</artifactId>
    <version>2.29.0</version>
</dependency>
<dependency>
    <groupId>io.qameta.allure</groupId>
    <artifactId>allure-testng</artifactId>
    <version>2.29.0</version>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.5.8</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>2.0.16</version>
</dependency>
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.11.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testng</groupId>
    <artifactId>testng</artifactId>
    <version>7.10.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>io.github.bonigarcia</groupId>
    <artifactId>webdrivermanager</artifactId>
    <version>5.9.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>rest-assured</artifactId>
    <version>5.5.0</version>
    <scope>test</scope>
</dependency>
```

## Интеграция с CI/CD

Allure интегрируется с Jenkins для автоматической генерации отчётов, где аннотации `@Tag` и `@Severity` упрощают фильтрацию.

- **Фильтрация в Jenkins**: Используйте плагин Allure для фильтрации по `@Tag` (например, только `smoke`) или `@Severity` (например, только `CRITICAL`).
- **Логи**: Архивируются в `logs/*.log` для корреляции с Allure-отчётами.

## Лучшие практики и отладка

### Использование аннотаций

- **@Epic, @Feature, @Story**: Обязательны для всех UI и API тестов. Убедитесь, что они отражают бизнес-логику и пользовательские сценарии.
- **@Owner**: Указывайте команду или конкретного инженера (например, `@Owner("john.doe")`).
- **@Tag**: Используйте теги для категоризации (`ui`, `api`, `regression`, `smoke`) и фильтрации в CI.
- **@Severity**: Присваивайте критичность в зависимости от влияния теста на функциональность.

### Отладка

- **Локальная проверка**: Используйте `mvn allure:serve` для просмотра отчётов Allure.
- **Анализ логов**: Проверяйте `logs/tests.log` в IntelliJ IDEA для корреляции с шагами Allure.
- **CI/CD**: Убедитесь, что Jenkins корректно отображает иерархию (`Epic > Feature > Story`) и фильтрует по `@Tag` и `@Severity`.

### Распространённые ошибки

- **Пропуск обязательных аннотаций**: Отсутствие `@Epic`, `@Feature` или `@Story` нарушает структуру отчёта.
- **Неправильный `@Owner`**: Указание некорректного ответственного затрудняет коммуникацию.
- **Избыточные теги**: Слишком много `@Tag` усложняет фильтрацию.
- **Неправильная критичность**: Завышение или занижение `@Severity` искажает приоритеты.

## Дополнительно

### Пример с обработкой ошибок и скриншотами

```java
import io.qameta.allure.Allure;
import io.qameta.allure.Attachment;
import io.qameta.allure.Epic;
import io.qameta.allure.Feature;
import io.qameta.allure.Owner;
import io.qameta.allure.Severity;
import io.qameta.allure.SeverityLevel;
import io.qameta.allure.Story;
import io.qameta.allure.Tag;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import static org.assertj.core.api.Assertions.assertThat;

public class UiAllureErrorTest {
    private static final Logger logger = LoggerFactory.getLogger(UiAllureErrorTest.class);
    private WebDriver driver;

    @BeforeEach
    void setUp() {
        logger.info("Инициализация UI-теста");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
    }

    @Test
    @Epic("Авторизация")
    @Feature("Модальное окно входа")
    @Story("Неверный формат email")
    @Owner("qa_team")
    @Tag("ui")
    @Tag("regression")
    @Severity(SeverityLevel.NORMAL)
    void testInvalidEmail() {
        openLoginPage();
        enterInvalidEmail();
        verifyErrorMessage();
    }

    @Attachment(value = "Скриншот ошибки", type = "image/png")
    private byte[] attachScreenshot() {
        return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
    }

    @Step("Открытие страницы логина")
    private void openLoginPage() {
        logger.info("Открытие страницы: https://example.com/login");
        driver.get("https://example.com/login");
    }

    @Step("Ввод неверного email: {email}")
    private void enterInvalidEmail() {
        logger.info("Ввод email: invalid-email");
        // driver.findElement(By.id("email")).sendKeys("invalid-email");
    }

    @Step("Проверка сообщения об ошибке")
    private void verifyErrorMessage() {
        try {
            logger.info("Проверка наличия сообщения об ошибке");
            assertThat(driver.getPageSource()).contains("Invalid email");
        } catch (AssertionError e) {
            logger.error("Ошибка в тесте: {}", e.getMessage(), e);
            attachScreenshot();
            throw e;
        }
    }

    @AfterEach
    void tearDown() {
        logger.info("Завершение UI-теста");
        if (driver != null) {
            driver.quit();
        }
    }
}
```

### Интеграция с Testcontainers

Testcontainers помогает тестировать интеграцию с системами логирования, с поддержкой Allure-аннотаций.

```java
import io.qameta.allure.Epic;
import io.qameta.allure.Feature;
import io.qameta.allure.Owner;
import io.qameta.allure.Severity;
import io.qameta.allure.SeverityLevel;
import io.qameta.allure.Step;
import io.qameta.allure.Story;
import io.qameta.allure.Tag;
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testcontainers.containers.GenericContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

@Testcontainers
public class AllureContainerTest {
    private static final Logger logger = LoggerFactory.getLogger(AllureContainerTest.class);

    @Container
    private static final GenericContainer<?> logstash = new GenericContainer<>("logstash:8.15.2")
            .withExposedPorts(5000);

    @Test
    @Epic("Интеграционное тестирование")
    @Feature("Логирование в Logstash")
    @Story("Проверка отправки логов")
    @Owner("devops_team")
    @Tag("integration")
    @Severity(SeverityLevel.MINOR)
    void testLogstashIntegration() {
        logContainerStep();
    }

    @Step("Отправка лога в Logstash")
    private void logContainerStep() {
        logger.info("Отправка лога в Logstash на {}:{}", logstash.getHost(), logstash.getFirstMappedPort());
    }
}
```

## Источники

- [Allure Framework](https://allurereport.org/docs/)
- [SLF4J User Manual](http://www.slf4j.org/manual.html)
- [Logback Documentation](https://logback.qos.ch/documentation.html)
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/webdriver/)
- [RestAssured Documentation](https://rest-assured.io/)
- [Testcontainers Documentation](https://testcontainers.org/)
- [Jenkins Pipeline Documentation](https://www.jenkins.io/doc/book/pipeline/)

---
[**← Назад к оглавлению**](../README.md)