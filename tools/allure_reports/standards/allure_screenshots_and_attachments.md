# Скриншоты, вложения и артефакты в Allure

Вложения в Allure, такие как скриншоты, логи, HTML, JSON и curl-запросы, значительно упрощают анализ падений тестов и обеспечивают дополнительный контекст. В статье описываются рекомендации по автоматическому прикреплению скриншотов при падении тестов, добавлению различных типов вложений по необходимости и использованию метода `Allure.addAttachment(...)`. Используется технологический стек: Java 17, Maven, SLF4J с Logback, JUnit 5, TestNG, Selenium WebDriver, RestAssured, Allure и Jenkins.

## Рекомендации по работе с вложениями

### Автоматическое прикрепление скриншотов при падении теста

- **Цель**: Обеспечить визуальный контекст ошибки для быстрого анализа.
- **Как реализовать**: Используйте `try-catch` для обработки исключений и прикрепляйте скриншот через `@Attachment` или `Allure.addAttachment(...)` при сбое.
- **Инструменты**: Для UI-тестов используйте Selenium WebDriver (`TakesScreenshot`).

### Прикрепление логов, HTML, JSON, curl-запросов по необходимости

- **Логи**: Прикрепляйте выдержки из логов (например, из Logback) для анализа ошибок.
- **HTML**: Сохраняйте исходный код страницы для UI-тестов.
- **JSON**: Прикрепляйте запросы и ответы API для API-тестов.
- **curl-запросы**: Полезны для воспроизведения API-запросов вручную.
- **Когда использовать**: Добавляйте вложения, если они помогают в анализе ошибки или предоставляют важный контекст.

### Использование Allure.addAttachment(...)

- **Метод**: `Allure.addAttachment(name, type, content)` используется для прикрепления данных в отчёт.
- **Параметры**:
  - `name`: Название вложения (например, «Скриншот ошибки»).
  - `type`: MIME-тип (например, `image/png`, `text/plain`, `application/json`).
  - `content`: Данные вложения (байты, строка или InputStream).

## Примеры реализации

### UI-тест с автоматическим скриншотом (JUnit 5)

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
public class UiAttachmentTest {
    private static final Logger logger = LoggerFactory.getLogger(UiAttachmentTest.class);
    private WebDriver driver;

    @BeforeEach
    void setUp() {
        logger.info("Инициализация UI-теста");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
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
- При сбое теста (`AssertionError`) автоматически прикрепляется скриншот (`image/png`) и HTML-код страницы (`text/html`).
- Логи в `logs/tests.log` дополняют контекст ошибки.

### API-тест с вложениями JSON и curl (TestNG)

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
public class ApiAttachmentTest {
    private static final Logger logger = LoggerFactory.getLogger(ApiAttachmentTest.class);

    @Test
    @Description("Тест проверяет получение данных пользователя по ID")
    @DisplayName("Проверка получения данных пользователя по ID")
    void testGetUserData() {
        Response response = sendGetRequest();
        verifyResponseStatus(response);
        attachResponseJson(response);
        attachCurlRequest();
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
    private void verifyResponseStatus(Response response) {
        try {
            logger.info("Проверка статуса ответа: {}", response.getStatusCode());
            assertEquals(response.getStatusCode(), 200, "Ожидался статус 200");
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
    private String attachCurlRequest() {
        logger.info("Прикрепление curl-запроса");
        return "curl -X GET https://api.example.com/users/1";
    }
}
```

**Что происходит**:
- При сбое теста прикрепляется JSON-ответ API (`application/json`).
- Всегда прикрепляется curl-запрос (`text/plain`) для воспроизведения.

## Интеграция с CI/CD

Allure интегрируется с Jenkins для отображения вложений в отчётах.

- **Вложения в Allure**: Скриншоты, JSON, HTML и curl-запросы отображаются в отчёте для каждого теста.
- **Логи**: Архивируются в `logs/*.log` для корреляции с вложениями.

## Лучшие практики и отладка

### Автоматическое прикрепление скриншотов

- **Где использовать**: В UI-тестах, особенно при сбоях (`AssertionError` или другие исключения).
- **Как**: Используйте `@Attachment` для методов, возвращающих `byte[]` (скриншоты) или `String` (HTML, текст).
- **Пример**: Прикрепляйте скриншот только при ошибке, чтобы не перегружать отчёт.

### Прикрепление логов, JSON, curl

- **Логи**: Извлекайте релевантные строки из `logs/tests.log` и прикрепляйте через `Allure.addAttachment("Логи", "text/plain", logContent)`.
- **JSON**: Для API-тестов прикрепляйте запрос и ответ в формате `application/json`.
- **curl**: Формируйте curl-команду для воспроизведения запроса (например, `curl -X GET ...`).
- **Когда**: Прикрепляйте только при ошибках или если вложение критично для анализа.

### Использование Allure.addAttachment(...)

- **Гибкость**: Подходит для любых данных (файлы, текст, байты).
- **MIME-типы**:
  - Скриншоты: `image/png`.
  - Логи/HTML: `text/plain`, `text/html`.
  - JSON: `application/json`.
- **Пример**: `Allure.addAttachment("JSON ответа", "application/json", response.asPrettyString())`.

### Отладка

- **Локальная проверка**: Используйте `mvn allure:serve` для просмотра отчётов и проверки вложений.
- **Логи**: Анализируйте `logs/tests.log` в IntelliJ IDEA для корреляции с вложениями.
- **CI/CD**: Убедитесь, что Jenkins отображает вложения (скриншоты, JSON) в Allure-отчётах.

### Распространённые ошибки

- **Отсутствие скриншотов**: Пропуск прикрепления скриншотов при сбоях UI-тестов.
- **Перегрузка вложениями**: Прикрепление всех логов или JSON без необходимости.
- **Неправильный MIME-тип**: Например, использование `text/plain` для JSON вместо `application/json`.

## Дополнительно

### Пример с обработкой ошибок и несколькими вложениями

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
public class UiFullAttachmentTest {
    private static final Logger logger = LoggerFactory.getLogger(UiFullAttachmentTest.class);
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
public class AllureAttachmentContainerTest {
    private static final Logger logger = LoggerFactory.getLogger(AllureAttachmentContainerTest.class);

    @Container
    private static final GenericContainer<?> logstash = new GenericContainer<>("logstash:8.15.2")
            .withExposedPorts(5000);

    @Test
    @DisplayName("Проверка отправки логов в Logstash")
    void testLogstashIntegration() {
        startLogstashContainer();
        sendLogToLogstash();
    }

    @Step Euros
    @Step("Запуск контейнера Logstash")
    private void startLogstashContainer() {
        logger.info("Запуск контейнера Logstash на порту: {}", logstash.getFirstMappedPort());
        Allure.step("Контейнер Logstash запущен на " + logstash.getHost() + ":" + logstash.getFirstMappedPort());
    }

    @Step("Отправка лога в Logstash")
    private void sendLogToLogstash() {
        logger.info("Отправка лога в Logstash на {}:{}", logstash.getHost(), logstash.getFirstMappedPort());
        Allure.step("Отправка тестового лога в Logstash");
        Allure.addAttachment("Лог сообщения", "text/plain", "Тестовое сообщение для Logstash");
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