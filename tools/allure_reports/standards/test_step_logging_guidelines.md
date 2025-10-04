# Логирование шагов теста в Allure

Логирование шагов теста с использованием аннотации `@Step` или метода `Allure.step(...)` обеспечивает прозрачность и читаемость тестов, отражая пользовательскую логику. Каждый шаг должен описывать конкретное действие, такое как «Ввод email» или «Нажатие кнопки входа», и быть понятным для QA, разработчиков и бизнеса. В статье описываются рекомендации по логированию шагов в UI и API тестах. Используется технологический стек: Java 17, Maven, SLF4J с Logback, JUnit 5, TestNG, Selenium WebDriver, RestAssured, Allure и Jenkins.

## Рекомендации по логированию шагов

### Использовать @Step или Allure.step(...)

- **@Step**: Аннотация для методов, описывающих шаги теста. Используется для структурированных и переиспользуемых действий.
- **Allure.step(...)**: Ручное логирование шагов внутри теста, подходит для динамических или одноразовых действий.
- **Когда использовать**:
  - `@Step` — для повторяющихся действий, которые можно вынести в отдельные методы.
  - `Allure.step(...)` — для уникальных шагов или динамической генерации описаний.

### Каждый шаг должен описывать действие

- Шаги должны быть конкретными и отражать действия пользователя или системы, например:
  - «Ввод email в поле формы».
  - «Нажатие кнопки входа».
  - «Отправка GET-запроса для получения данных».
- Избегайте общих фраз, таких как «Проверка» или «Тест», без указания действия.

### Шаги должны быть читаемыми и отражать пользовательскую логику

- **Читаемость**: Используйте понятные описания, ориентированные на бизнес-логику, а не технические детали.
- **Пользовательская логика**: Шаги должны соответствовать сценарию, который выполняет пользователь или API, например, последовательность действий в интерфейсе или запросах.
- **Пример**: Вместо «Валидация поля email» используйте «Проверка сообщения об ошибке при неверном формате email».

## Примеры реализации

### UI-тест с логированием шагов (JUnit 5)

```java
import io.qameta.allure.Allure;
import io.qameta.allure.Attachment;
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
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.By;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import static org.assertj.core.api.Assertions.assertThat;

@Epic("Авторизация")
@Feature("Модальное окно входа")
@Story("Вход в систему")
@Owner("qa_team")
@Tag("ui")
@Tag("smoke")
@Severity(SeverityLevel.BLOCKER)
public class UiStepLoggingTest {
    private static final Logger logger = LoggerFactory.getLogger(UiStepLoggingTest.class);
    private WebDriver driver;

    @BeforeEach
    void setUp() {
        logger.info("Инициализация UI-теста");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
    }

    @Test
    @DisplayName("Проверка успешного входа с корректными данными")
    @Description("Тест проверяет вход в систему с корректными учетными данными")
    void testSuccessfulLogin() {
        openLoginPage();
        enterCredentials("user@example.com", "password");
        submitLoginForm();
        verifyDashboardPage();
    }

    @Step("Открытие страницы логина")
    private void openLoginPage() {
        logger.info("Открытие страницы: https://example.com/login");
        driver.get("https://example.com/login");
    }

    @Step("Ввод учетных данных: email={email}, password={password}")
    private void enterCredentials(String email, String password) {
        logger.info("Ввод email: {}, password: ****", email);
        Allure.step("Ввод email: " + email);
        // driver.findElement(By.id("email")).sendKeys(email);
        // driver.findElement(By.id("password")).sendKeys(password);
    }

    @Step("Нажатие кнопки входа")
    private void submitLoginForm() {
        logger.info("Нажатие кнопки входа");
        // driver.findElement(By.id("loginButton")).click();
    }

    @Step("Проверка перехода на страницу дашборда")
    private void verifyDashboardPage() {
        logger.info("Проверка URL после входа");
        Allure.step("Проверка URL: ожидается dashboard");
        assertThat(driver.getCurrentUrl()).contains("dashboard");
    }

    @Attachment(value = "Скриншот ошибки", type = "image/png")
    private byte[] attachScreenshot() {
        return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
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

### API-тест с логированием шагов (TestNG)

```java
import io.qameta.allure.Allure;
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

@Epic("API-тестирование")
@Feature("API пользователей")
@Story("Получение данных пользователя")
@Owner("api_team")
@Tag("api")
@Tag("regression")
@Severity(SeverityLevel.CRITICAL)
public class ApiStepLoggingTest {
    private static final Logger logger = LoggerFactory.getLogger(ApiStepLoggingTest.class);

    @Test
    @Description("Тест проверяет получение данных пользователя по ID")
    @DisplayName("Проверка получения данных пользователя по ID")
    void testGetUserData() {
        sendGetRequest();
        verifyResponseStatus();
        verifyUserId();
    }

    @Step("Отправка GET-запроса для получения данных пользователя")
    private Response sendGetRequest() {
        logger.info("Отправка GET-запроса: /users/1");
        RestAssured.baseURI = "https://api.example.com";
        Response response = given()
                .when()
                .get("/users/1")
                .then()
                .extract().response();
        Allure.step("Получен ответ с кодом: " + response.getStatusCode());
        return response;
    }

    @Step("Проверка статуса ответа")
    private void verifyResponseStatus() {
        Response response = sendGetRequest();
        logger.info("Проверка статуса ответа: {}", response.getStatusCode());
        assertEquals(response.getStatusCode(), 200, "Ожидался статус 200");
    }

    @Step("Проверка ID пользователя в ответе")
    private void verifyUserId() {
        Response response = sendGetRequest();
        logger.info("Проверка ID пользователя");
        Allure.step("Проверка: ID пользователя должен быть 1");
        assertEquals(response.jsonPath().getString("id"), "1");
    }
}
```

## Интеграция с CI/CD

Allure интегрируется с Jenkins для отображения шагов теста в отчётах, что упрощает анализ.

- **Шаги в Allure**: Аннотации `@Step` и вызовы `Allure.step(...)` отображаются как отдельные шаги в отчёте, с указанием статуса (успех/ошибка).
- **Логи**: Архивируются в `logs/*.log` для корреляции с шагами.

## Лучшие практики и отладка

### Логирование шагов

- **@Step**: Используйте для методов, которые переиспользуются в нескольких тестах (например, `openLoginPage()`).
- **Allure.step(...)**: Применяйте для динамических шагов или уточнения деталей (например, логирование конкретного email).
- **Читаемость**: Описывайте шаги с точки зрения пользователя, например, «Ввод email» вместо «Заполнение поля email».
- **Пользовательская логика**: Убедитесь, что шаги отражают последовательность действий пользователя или API.

### Отладка

- **Локальная проверка**: Используйте `mvn allure:serve` для просмотра отчётов и проверки шагов.
- **Логи**: Анализируйте `logs/tests.log` в IntelliJ IDEA для корреляции с шагами Allure.
- **CI/CD**: Убедитесь, что Jenkins корректно отображает шаги в Allure-отчётах.

### Распространённые ошибки

- **Неинформативные шаги**: Например, `@Step("Проверка")` не описывает конкретное действие.
- **Слишком много шагов**: Избыточное дробление теста на шаги усложняет отчёт.
- **Технические термины**: Например, `@Step("Валидация ответа API")` вместо «Проверка данных пользователя».

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
import io.qameta.allure.Step;
import io.qameta.allure.Story;
import io.qameta.allure.Tag;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.By;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import static org.assertj.core.api.Assertions.assertThat;

@Epic("Авторизация")
@Feature("Модальное окно входа")
@Story("Обработка ошибок")
@Owner("qa_team")
@Tag("ui")
@Tag("regression")
@Severity(SeverityLevel.CRITICAL)
public class UiStepErrorTest {
    private static final Logger logger = LoggerFactory.getLogger(UiStepErrorTest.class);
    private WebDriver driver;

    @BeforeEach
    void setUp() {
        logger.info("Инициализация UI-теста");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
    }

    @Test
    @DisplayName("Проверка ошибки при неверном формате email")
    void testInvalidEmailFormat() {
        openLoginPage();
        enterInvalidEmail("invalid-email");
        submitLoginForm();
        verifyErrorMessage();
    }

    @Step("Открытие страницы логина")
    private void openLoginPage() {
        logger.info("Открытие страницы: https://example.com/login");
        driver.get("https://example.com/login");
    }

    @Step("Ввод email: {email}")
    private void enterInvalidEmail(String email) {
        logger.info("Ввод email: {}", email);
        Allure.step("Ввод некорректного email: " + email);
        // driver.findElement(By.id("email")).sendKeys(email);
    }

    @Step("Нажатие кнопки входа")
    private void submitLoginForm() {
        logger.info("Нажатие кнопки входа");
        // driver.findElement(By.id("loginButton")).click();
    }

    @Step("Проверка сообщения об ошибке")
    private void verifyErrorMessage() {
        try {
            logger.info("Проверка сообщения об ошибке");
            Allure.step("Ожидается сообщение: Неверный формат email");
            assertThat(driver.getPageSource()).contains("Invalid email");
        } catch (AssertionError e) {
            logger.error("Ошибка в тесте: {}", e.getMessage(), e);
            attachScreenshot();
            throw e;
        }
    }

    @Attachment(value = "Скриншот ошибки", type = "image/png")
    private byte[] attachScreenshot() {
        return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
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

```java
import io.qameta.allure.Allure;
import io.qameta.allure.Epic;
import io.qameta.allure.Feature;
import io.qameta.allure.Owner;
import io.qameta.allure.Severity;
import io.qameta.allure.SeverityLevel;
import io.qameta.allure.Step;
import io.qameta.allure.Story;
import io.qameta.allure.Tag;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testcontainers.containers.GenericContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

@Testcontainers
@Epic("Интеграционное тестирование")
@Feature("Логирование в Logstash")
@Story("Отправка логов")
@Owner("devops_team")
@Tag("integration")
@Severity(SeverityLevel.MINOR)
public class AllureStepContainerTest {
    private static final Logger logger = LoggerFactory.getLogger(AllureStepContainerTest.class);

    @Container
    private static final GenericContainer<?> logstash = new GenericContainer<>("logstash:8.15.2")
            .withExposedPorts(5000);

    @Test
    @DisplayName("Проверка отправки логов в Logstash")
    void testLogstashIntegration() {
        startLogstashContainer();
        sendLogToLogstash();
    }

    @Step("Запуск контейнера Logstash")
    private void startLogstashContainer() {
        logger.info("Запуск контейнера Logstash на порту: {}", logstash.getFirstMappedPort());
        Allure.step("Контейнер Logstash запущен на " + logstash.getHost() + ":" + logstash.getFirstMappedPort());
    }

    @Step("Отправка лога в Logstash")
    private void sendLogToLogstash() {
        logger.info("Отправка лога в Logstash на {}:{}", logstash.getHost(), logstash.getFirstMappedPort());
        Allure.step("Отправка тестового лога в Logstash");
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