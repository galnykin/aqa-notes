# Обработка ошибок и отображение причин падения в Allure

Для упрощения анализа падений тестов в отчётах Allure необходимо явно отображать причину сбоя (например, `AssertionError`, `TimeoutException`, `NoSuchElementException`), стек вызовов, шаги и вложения. Ошибки не должны скрываться, а их описание должно быть читаемым и информативным для QA, разработчиков и бизнеса. В статье описываются рекомендации по обработке ошибок и отображению их причин. Используется технологический стек: Java 17, Maven, SLF4J с Logback, JUnit 5, TestNG, Selenium WebDriver, RestAssured, Allure и Jenkins.

## Рекомендации по обработке ошибок

### Отображение причины падения

- **Типы ошибок**: Указывайте конкретную причину сбоя, такую как `AssertionError` (невыполненное утверждение), `TimeoutException` (превышение времени ожидания), `NoSuchElementException` (отсутствие элемента в DOM).
- **Как**: Логируйте исключение через SLF4J и прикрепляйте его в Allure через `@Attachment` или `Allure.addAttachment(...)`.
- **Где**: В `try-catch` блоках, обрабатывающих проверки (`assert`) или действия Selenium/API.

### Отображение стека, шагов, вложений

- **Стек вызовов**: Прикрепляйте полный стек ошибки для диагностики.
- **Шаги**: Убедитесь, что шаги теста, выполненные до сбоя, видны в Allure (через `@Step` или `Allure.step(...)`).
- **Вложения**: Добавляйте скриншоты, HTML, JSON, логи или curl-запросы при сбое для контекста.
- **Как**: Используйте `@Attachment` для вложений и `Allure.getLifecycle().addAttachment(...)` для динамических данных.

### Не скрывать ошибки

- **Читаемость**: Описывайте ошибки в терминах, понятных не только разработчикам (например, «Элемент не найден» вместо «NoSuchElementException»).
- **Подробности**: Указывайте, какой шаг привёл к ошибке, какие данные использовались и что ожидалось.
- **Логирование**: Используйте SLF4J для записи ошибок в `logs/tests.log`, синхронизируя с Allure.

## Примеры реализации

### UI-тест с обработкой ошибок (JUnit 5)

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
import org.openqa.selenium.NoSuchElementException;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.TimeoutException;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.time.Duration;

import static org.assertj.core.api.Assertions.assertThat;

@Epic("Авторизация")
@Feature("Модальное окно входа")
@Story("Обработка ошибок")
@Owner("qa_team")
@Tag("ui")
@Tag("regression")
@Severity(SeverityLevel.CRITICAL)
public class UiErrorHandlingTest {
    private static final Logger logger = LoggerFactory.getLogger(UiErrorHandlingTest.class);
    private WebDriver driver;
    private WebDriverWait wait;

    @BeforeEach
    void setUp() {
        logger.info("Инициализация UI-теста");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        wait = new WebDriverWait(driver, Duration.ofSeconds(10));
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

    @Test
    @DisplayName("Проверка ошибки при отсутствии элемента")
    @Description("Тест проверяет обработку отсутствия кнопки входа")
    void testNoSuchElement() {
        openLoginPage();
        submitMissingLoginForm();
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
        try {
            driver.findElement(By.id("email")).sendKeys(email);
        } catch (NoSuchElementException e) {
            logger.error("Элемент email не найден: {}", e.getMessage(), e);
            attachScreenshot();
            attachPageSource();
            attachStackTrace(e);
            throw e;
        }
    }

    @Step("Нажатие кнопки входа")
    private void submitLoginForm() {
        logger.info("Нажатие кнопки входа");
        Allure.step("Нажатие кнопки входа");
        try {
            driver.findElement(By.id("loginButton")).click();
        } catch (NoSuchElementException e) {
            logger.error("Кнопка входа не найдена: {}", e.getMessage(), e);
            attachScreenshot();
            attachPageSource();
            attachStackTrace(e);
            throw e;
        }
    }

    @Step("Нажатие отсутствующей кнопки входа")
    private void submitMissingLoginForm() {
        logger.info("Попытка нажатия отсутствующей кнопки входа");
        Allure.step("Попытка нажатия отсутствующей кнопки входа");
        try {
            wait.until(ExpectedConditions.elementToBeClickable(By.id("missingButton")));
        } catch (TimeoutException e) {
            logger.error("Превышено время ожидания элемента: {}", e.getMessage(), e);
            attachScreenshot();
            attachPageSource();
            attachStackTrace(e);
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
            logger.error("Ошибка утверждения: {}", e.getMessage(), e);
            attachScreenshot();
            attachPageSource();
            attachStackTrace(e);
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

    @Attachment(value = "Стек ошибки", type = "text/plain")
    private String attachStackTrace(Throwable throwable) {
        logger.info("Прикрепление стека ошибки");
        StringBuilder stackTrace = new StringBuilder(throwable.toString()).append("\n");
        for (StackTraceElement element : throwable.getStackTrace()) {
            stackTrace.append(element.toString()).append("\n");
        }
        return stackTrace.toString();
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
- **Ошибки**: Обрабатываются `AssertionError`, `NoSuchElementException`, `TimeoutException`.
- **Логирование**: Причина сбоя и стек вызовов записываются в `logs/tests.log` и прикрепляются в Allure (`text/plain`).
- **Вложения**: При сбое добавляются скриншот (`image/png`), HTML страницы (`text/html`) и стек ошибки.
- **Шаги**: Все шаги до сбоя отображаются в Allure через `@Step`.

### API-тест с обработкой ошибок (TestNG)

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

import java.io.IOException;

import static io.restassured.RestAssured.given;
import static org.testng.Assert.assertEquals;

@Epic("API-тестирование")
@Feature("API пользователей")
@Story("Получение данных пользователя")
@Owner("api_team")
@Tag("api")
@Tag("regression")
@Severity(SeverityLevel.CRITICAL)
public class ApiErrorHandlingTest {
    private static final Logger logger = LoggerFactory.getLogger(ApiErrorHandlingTest.class);

    @Test
    @Description("Тест проверяет получение данных пользователя по ID")
    @DisplayName("Проверка получения данных пользователя по ID")
    void testGetUserData() {
        Response response = sendGetRequest("1");
        verifyResponseStatus(response);
        verifyUserId(response);
    }

    @Test
    @Description("Тест проверяет ошибку при недоступном сервере")
    @DisplayName("Проверка ошибки при недоступном сервере")
    void testServerUnavailable() {
        Response response = sendGetRequestToInvalidServer("1");
        verifyResponseStatus(response);
    }

    @Step("Отправка GET-запроса для пользователя с ID: {userId}")
    private Response sendGetRequest(String userId) {
        logger.info("Отправка GET-запроса: /users/{}", userId);
        Allure.step("Отправка GET-запроса для пользователя с ID: " + userId);
        RestAssured.baseURI = "https://api.example.com";
        Response response;
        try {
            response = given()
                    .when()
                    .get("/users/" + userId)
                    .then()
                    .extract().response();
        } catch (Exception e) {
            logger.error("Ошибка при отправке запроса: {}", e.getMessage(), e);
            attachStackTrace(e);
            throw e;
        }
        attachResponseJson(response);
        attachCurlRequest(userId);
        return response;
    }

    @Step("Отправка GET-запроса к недоступному серверу для пользователя с ID: {userId}")
    private Response sendGetRequestToInvalidServer(String userId) {
        logger.info("Отправка GET-запроса к недоступному серверу: /users/{}", userId);
        Allure.step("Отправка GET-запроса к недоступному серверу с ID: " + userId);
        RestAssured.baseURI = "https://invalid-api.example.com";
        Response response;
        try {
            response = given()
                    .when()
                    .get("/users/" + userId)
                    .then()
                    .extract().response();
        } catch (Exception e) {
            logger.error("Ошибка при отправке запроса: {}", e.getMessage(), e);
            attachStackTrace(e);
            throw e;
        }
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
            logger.error("Ошибка утверждения: {}", e.getMessage(), e);
            attachResponseJson(response);
            attachStackTrace(e);
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
            logger.error("Ошибка утверждения: {}", e.getMessage(), e);
            attachResponseJson(response);
            attachStackTrace(e);
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

    @Attachment(value = "Стек ошибки", type = "text/plain")
    private String attachStackTrace(Throwable throwable) {
        logger.info("Прикрепление стека ошибки");
        StringBuilder stackTrace = new StringBuilder(throwable.toString()).append("\n");
        for (StackTraceElement element : throwable.getStackTrace()) {
            stackTrace.append(element.toString()).append("\n");
        }
        return stackTrace.toString();
    }
}
```

**Что происходит**:
- **Ошибки**: Обрабатываются `AssertionError` и исключения при запросах (например, `IOException`).
- **Логирование**: Причина сбоя и стек вызовов записываются в лог и Allure.
- **Вложения**: При сбое добавляются JSON ответа (`application/json`), curl-запрос (`text/plain`) и стек ошибки (`text/plain`).
- **Шаги**: Все шаги до сбоя отображаются в Allure.

## Конфигурация Logback

Логи фиксируют ошибки и дополняют отчёты Allure.

**`logback.xml`** (в `src/main/resources`):

```xml
<configuration>
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/tests.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>logs/tests.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>10MB</maxFileSize>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="info">
        <appender-ref ref="FILE"/>
    </root>
</configuration>
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

Allure интегрируется с Jenkins для отображения ошибок и вложений.

**Пример `Jenkinsfile`**:

```groovy
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh 'mvn clean test'
            }
        }
        stage('Allure Report') {
            steps {
                allure([
                    includeProperties: false,
                    jdk: '',
                    results: [[path: 'target/allure-results']]
                ])
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'target/allure-results/**, logs/*.log', allowEmptyArchive: true
        }
    }
}
```

- **Ошибки в Allure**: Причина сбоя (`AssertionError`, `TimeoutException` и т.д.) отображается в отчёте вместе со стеком и вложениями.
- **Логи**: Архивируются в `logs/*.log` для корреляции.

## Лучшие практики и отладка

### Отображение причины падения

- **Конкретность**: Указывайте тип исключения (`AssertionError`, `NoSuchElementException`) и его сообщение.
- **Логирование**: Используйте `logger.error` с полным стеком (`e.getMessage(), e`).
- **Allure**: Прикрепляйте стек через `@Attachment` с типом `text/plain`.

### Отображение стека, шагов, вложений

- **Стек**: Форматируйте стек вызовов для читаемости (строка с `toString()` и элементами `StackTraceElement`).
- **Шаги**: Убедитесь, что все шаги до сбоя логируются через `@Step` или `Allure.step(...)`.
- **Вложения**: Добавляйте только релевантные данные (скриншот, JSON, HTML) при сбое.

### Не скрывать ошибки

- **Читаемость**: Переводите технические сообщения в понятные, например, «Элемент не найден» вместо `NoSuchElementException`.
- **Подробности**: Указывайте входные данные и ожидаемые результаты в шагах.
- **Логи**: Синхронизируйте SLF4J и Allure для полного контекста.

### Отладка

- **Локальная проверка**: Используйте `mvn allure:serve` для просмотра отчётов и проверки ошибок.
- **Логи**: Анализируйте `logs/tests.log` в IntelliJ IDEA для корреляции с Allure.
- **CI/CD**: Убедитесь, что Jenkins отображает ошибки и вложения в отчётах.

### Распространённые ошибки

- **Скрытие исключений**: Поглощение ошибок без логирования или прикрепления.
- **Недостаток контекста**: Отсутствие скриншотов, JSON или стека при сбое.
- **Неинформативные сообщения**: Например, «Тест упал» вместо «Элемент не найден».

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
import org.openqa.selenium.NoSuchElementException;
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
public class UiFullErrorHandlingTest {
    private static final Logger logger = LoggerFactory.getLogger(UiFullErrorHandlingTest.class);
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
        try {
            driver.findElement(By.id("email")).sendKeys(email);
        } catch (NoSuchElementException e) {
            logger.error("Элемент email не найден: {}", e.getMessage(), e);
            attachScreenshot();
            attachPageSource();
            attachStackTrace(e);
            throw e;
        }
    }

    @Step("Нажатие кнопки входа")
    private void submitLoginForm() {
        logger.info("Нажатие кнопки входа");
        Allure.step("Нажатие кнопки входа");
        try {
            driver.findElement(By.id("loginButton")).click();
        } catch (NoSuchElementException e) {
            logger.error("Кнопка входа не найдена: {}", e.getMessage(), e);
            attachScreenshot();
            attachPageSource();
            attachStackTrace(e);
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
            logger.error("Ошибка утверждения: {}", e.getMessage(), e);
            attachScreenshot();
            attachPageSource();
            attachStackTrace(e);
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

    @Attachment(value = "Стек ошибки", type = "text/plain")
    private String attachStackTrace(Throwable throwable) {
        logger.info("Прикрепление стека ошибки");
        StringBuilder stackTrace = new StringBuilder(throwable.toString()).append("\n");
        for (StackTraceElement element : throwable.getStackTrace()) {
            stackTrace.append(element.toString()).append("\n");
        }
        return stackTrace.toString();
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
public class AllureErrorContainerTest {
    private static final Logger logger = LoggerFactory.getLogger(AllureErrorContainerTest.class);

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
        try {
            logstash.start();
        } catch (Exception e) {
            logger.error("Ошибка запуска контейнера: {}", e.getMessage(), e);
            attachStackTrace(e);
            throw e;
        }
    }

    @Step("Отправка лога в Logstash: {message}")
    private void sendLogToLogstash(String message) {
        logger.info("Отправка лога в Logstash: {}", message);
        Allure.step("Отправка лога: " + message);
        Allure.step("Ожидается успешная отправка лога");
        try {
            // Симуляция отправки лога
            if (!logstash.isRunning()) {
                throw new IllegalStateException("Logstash не запущен");
            }
            attachLogMessage(message);
        } catch (Exception e) {
            logger.error("Ошибка отправки лога: {}", e.getMessage(), e);
            attachStackTrace(e);
            throw e;
        }
    }

    @Attachment(value = "Отправленное сообщение", type = "text/plain")
    private String attachLogMessage(String message) {
        logger.info("Прикрепление сообщения: {}", message);
        return message;
    }

    @Attachment(value = "Стек ошибки", type = "text/plain")
    private String attachStackTrace(Throwable throwable) {
        logger.info("Прикрепление стека ошибки");
        StringBuilder stackTrace = new StringBuilder(throwable.toString()).append("\n");
        for (StackTraceElement element : throwable.getStackTrace()) {
            stackTrace.append(element.toString()).append("\n");
        }
        return stackTrace.toString();
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