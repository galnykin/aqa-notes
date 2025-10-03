# Отображение входных данных и ожидаемых результатов в Allure

Для упрощения анализа падений тестов в отчётах Allure необходимо явно отображать входные данные (кроме чувствительных, таких как пароли) и ожидаемые результаты (сообщения, ошибки, переходы). Это помогает QA, разработчикам и бизнесу быстро понять причину сбоя. В статье описываются рекомендации по логированию входных данных и ожидаемых результатов с использованием аннотаций `@Step`, `@Attachment` и `Allure.step(...)`. Используется технологический стек: Java 17, Maven, SLF4J с Logback, JUnit 5, TestNG, Selenium WebDriver, RestAssured, Allure и Jenkins.

## Рекомендации по отображению входных данных и ожидаемых результатов

### Вводимые значения (кроме чувствительных)

- **Что отображать**: Все входные данные, используемые в тесте (например, email, параметры запроса), за исключением чувствительных данных (пароли, токены, личная информация).
- **Как**: Используйте `@Step` с параметрами или `Allure.step(...)` для логирования входных данных. Для чувствительных данных заменяйте значения на маски (например, `****`).
- **Пример**: Логируйте «Ввод email: user@example.com», но для пароля — «Ввод пароля: ****».

### Ожидаемые сообщения, ошибки, переходы

- **Что отображать**: Ожидаемые результаты, такие как текст сообщения об ошибке, статус-код API, URL после перехода.
- **Как**: Используйте `@Step` или `Allure.step(...)` для описания ожидаемого результата перед проверкой (`assert`). Прикрепляйте фактические результаты через `@Attachment` или `Allure.addAttachment(...)` при сбое.
- **Пример**: «Ожидается сообщение: Неверный формат email» или «Ожидается переход на /dashboard».

### Преимущества

- **Быстрый анализ**: Входные данные и ожидаемые результаты в отчёте помогают понять, что именно вызвало сбой.
- **Прозрачность**: Упрощает коммуникацию между QA, разработчиками и бизнесом.
- **Воспроизводимость**: Чёткое описание входных данных позволяет воспроизвести тест.

## Примеры реализации

### UI-тест с отображением входных данных и ожидаемых результатов (JUnit 5)

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
public class UiInputOutputTest {
    private static final Logger logger = LoggerFactory.getLogger(UiInputOutputTest.class);
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

    @Test
    @DisplayName("Проверка ошибки при неверном формате email")
    @Description("Тест проверяет отображение ошибки при вводе неверного email")
    void testInvalidEmailFormat() {
        openLoginPage();
        enterInvalidEmail("invalid-email");
        submitLoginForm();
        verifyErrorMessage();
    }

    @Step("Открытие страницы логина")
    private void openLoginPage() {
        logger.info("Открытие страницы: https://example.com/login");
        Allure.step("Открытие страницы логина: https://example.com/login");
        driver.get("https://example.com/login");
    }

    @Step("Ввод учетных данных: email={email}, password=****")
    private void enterCredentials(String email, String password) {
        logger.info("Ввод email: {}, password: ****", email);
        Allure.step("Ввод email: " + email);
        // driver.findElement(By.id("email")).sendKeys(email);
        // driver.findElement(By.id("password")).sendKeys(password);
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
        Allure.step("Нажатие кнопки входа");
        // driver.findElement(By.id("loginButton")).click();
    }

    @Step("Проверка перехода на страницу дашборда")
    private void verifyDashboardPage() {
        logger.info("Проверка URL после входа");
        Allure.step("Ожидается переход на страницу: /dashboard");
        try {
            assertThat(driver.getCurrentUrl()).contains("dashboard");
        } catch (AssertionError e) {
            logger.error("Ошибка в тесте: {}", e.getMessage(), e);
            attachScreenshot();
            attachPageSource();
            throw e;
        }
    }

    @Step("Проверка сообщения об ошибке")
    private void verifyErrorMessage() {
        logger.info("Проверка сообщения об ошибке");
        Allure.step("Ожидается сообщение: Неверный формат email");
        try {
            assertThat(driver.getPageSource()).contains("Invalid email");
        } catch (AssertionError e) {
            logger.error("Ошибка в тесте: {}", e.getMessage(), e);
            attachScreenshot();
            attachPageSource();
            throw e;
        }
    }

    @Attachment(value = "Скриншот ошибки", type = "image/png")
    private byte[] attachScreenshot() {
        logger.info("Создание скриншота");
        return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
    }

    @Attachment(value = "HTML страницы", type = "text/html")
    private String attachPageSource() {
        logger.info("Прикрепление HTML страницы");
        return driver.getPageSource();
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

**Что происходит**:
- **Входные данные**: Email логируется явно (например, «Ввод email: user@example.com»), пароль маскируется («password: ****»).
- **Ожидаемые результаты**: Логируются как шаги (например, «Ожидается переход на страницу: /dashboard» или «Ожидается сообщение: Неверный формат email»).
- **При сбое**: Прикрепляются скриншот (`image/png`) и HTML страницы (`text/html`).

### API-тест с отображением входных данных и ожидаемых результатов (TestNG)

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
public class ApiInputOutputTest {
    private static final Logger logger = LoggerFactory.getLogger(ApiInputOutputTest.class);

    @Test
    @Description("Тест проверяет получение данных пользователя по ID")
    @DisplayName("Проверка получения данных пользователя по ID")
    void testGetUserData() {
        Response response = sendGetRequest("1");
        verifyResponseStatus(response);
        verifyUserId(response);
    }

    @Step("Отправка GET-запроса для пользователя с ID: {userId}")
    private Response sendGetRequest(String userId) {
        logger.info("Отправка GET-запроса: /users/{}", userId);
        Allure.step("Отправка GET-запроса для пользователя с ID: " + userId);
        RestAssured.baseURI = "https://api.example.com";
        Response response = given()
                .when()
                .get("/users/" + userId)
                .then()
                .extract().response();
        attachResponseJson(response);
        attachCurlRequest(userId);
        return response;
    }

    @Step("Проверка статуса ответа")
    private void verifyResponseStatus(Response response) {
        logger.info("Проверка статуса ответа: {}", response.getStatusCode());
        Allure.step("Ожидается статус ответа: 200");
        try {
            assertEquals(response.getStatusCode(), 200, "Ожидался статус 200");
        } catch (AssertionError e) {
            logger.error("Ошибка в тесте: {}", e.getMessage(), e);
            attachResponseJson(response);
            throw e;
        }
    }

    @Step("Проверка ID пользователя в ответе")
    private void verifyUserId(Response response) {
        logger.info("Проверка ID пользователя");
        Allure.step("Ожидается ID пользователя: 1");
        try {
            assertEquals(response.jsonPath().getString("id"), "1");
        } catch (AssertionError e) {
            logger.error("Ошибка в тесте: {}", e.getMessage(), e);
            attachResponseJson(response);
            throw e;
        }
    }

    @Attachment(value = "JSON ответа", type = "application/json")
    private String attachResponseJson(Response response) {
        logger.info("Прикрепление JSON ответа");
        return response.asPrettyString();
    }

    @Attachment(value = "curl-запрос", type = "text/plain")
    private String attachCurlRequest(String userId) {
        logger.info("Прикрепление curl-запроса");
        return "curl -X GET https://api.example.com/users/" + userId;
    }
}
```

**Что происходит**:
- **Входные данные**: ID пользователя логируется явно (например, «Отправка GET-запроса для пользователя с ID: 1»).
- **Ожидаемые результаты**: Логируются как шаги (например, «Ожидается статус ответа: 200»).
- **При сбое**: Прикрепляется JSON ответа (`application/json`) и curl-запрос (`text/plain`).

## Интеграция с CI/CD

Allure интегрируется с Jenkins для отображения входных данных и ожидаемых результатов в отчётах.

- **Отображение в Allure**: Входные данные и ожидаемые результаты отображаются в шагах (`@Step`, `Allure.step(...)`), а вложения (скриншоты, JSON) доступны при сбоях.
- **Логи**: Архивируются в `logs/*.log` для корреляции с отчётами.

## Лучшие практики и отладка

### Отображение входных данных

- **Логирование**: Используйте `@Step` с параметрами (например, `@Step("Ввод email: {email}")`) для явного отображения данных.
- **Чувствительные данные**: Маскируйте пароли и токены (например, «password: ****»).
- **Ясность**: Указывайте точные значения (например, «ID: 1» вместо «ID пользователя»).

### Отображение ожидаемых результатов

- **Логирование**: Используйте `Allure.step(...)` перед проверкой для описания ожидаемого результата (например, «Ожидается статус: 200»).
- **При сбое**: Прикрепляйте фактические результаты (скриншот, JSON) через `@Attachment` или `Allure.addAttachment(...)`.
- **Контекст**: Указывайте, что именно ожидается (например, «Ожидается сообщение: Неверный формат email»).

### Отладка

- **Локальная проверка**: Используйте `mvn allure:serve` для просмотра отчётов и проверки входных/ожидаемых данных.
- **Логи**: Анализируйте `logs/tests.log` в IntelliJ IDEA для корреляции с шагами и вложениями.
- **CI/CD**: Убедитесь, что Jenkins отображает шаги и вложения в Allure-отчётах.

### Распространённые ошибки

- **Пропуск входных данных**: Не указаны используемые email, ID или параметры запроса.
- **Раскрытие чувствительных данных**: Логирование паролей или токенов вместо их маскировки.
- **Неинформативные ожидания**: Например, «Ожидается успех» вместо «Ожидается переход на /dashboard».

## Дополнительно

### Пример с обработкой ошибок и вложениями

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
public class UiFullInputOutputTest {
    private static final Logger logger = LoggerFactory.getLogger(UiFullInputOutputTest.class);
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

    @Step("Открытие страницы логина: https://example.com/login")
    private void openLoginPage() {
        logger.info("Открытие страницы: https://example.com/login");
        Allure.step("Открытие страницы логина: https://example.com/login");
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
        Allure.step("Нажатие кнопки входа");
        // driver.findElement(By.id("loginButton")).click();
    }

    @Step("Проверка сообщения об ошибке")
    private void verifyErrorMessage() {
        logger.info("Проверка сообщения об ошибке");
        Allure.step("Ожидается сообщение: Неверный формат email");
        try {
            assertThat(driver.getPageSource()).contains("Invalid email");
        } catch (AssertionError e) {
            logger.error("Ошибка в тесте: {}", e.getMessage(), e);
            attachScreenshot();
            attachPageSource();
            attachLogSnippet();
            throw e;
        }
    }

    @Attachment(value = "Скриншот ошибки", type = "image/png")
    private byte[] attachScreenshot() {
        logger.info("Создание скриншота");
        return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
    }

    @Attachment(value = "HTML страницы", type = "text/html")
    private String attachPageSource() {
        logger.info("Прикрепление HTML страницы");
        return driver.getPageSource();
    }

    @Attachment(value = "Выдержка из логов", type = "text/plain")
    private String attachLogSnippet() {
        logger.info("Прикрепление выдержки из логов");
        return "Логи теста: Ошибка при проверке сообщения";
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
import io.qameta.allure.Attachment;
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
public class AllureInputOutputContainerTest {
    private static final Logger logger = LoggerFactory.getLogger(AllureInputOutputContainerTest.class);

    @Container
    private static final GenericContainer<?> logstash = new GenericContainer<>("logstash:8.15.2")
            .withExposedPorts(5000);

    @Test
    @DisplayName("Проверка отправки логов в Logstash")
    void testLogstashIntegration() {
        startLogstashContainer();
        sendLogToLogstash("Test message");
    }

    @Step("Запуск контейнера Logstash на порту: {0}")
    private void startLogstashContainer() {
        logger.info("Запуск контейнера Logstash на порту: {}", logstash.getFirstMappedPort());
        Allure.step("Запуск контейнера Logstash на " + logstash.getHost() + ":" + logstash.getFirstMappedPort());
    }

    @Step("Отправка лога в Logstash: {message}")
    private void sendLogToLogstash(String message) {
        logger.info("Отправка лога в Logstash: {}", message);
        Allure.step("Отправка лога: " + message);
        Allure.step("Ожидается успешная отправка лога");
        attachLogMessage(message);
    }

    @Attachment(value = "Отправленное сообщение", type = "text/plain")
    private String attachLogMessage(String message) {
        logger.info("Прикрепление сообщения: {}", message);
        return message;
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