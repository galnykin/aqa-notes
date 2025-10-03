# Поддержка мультиязычности и читаемость в Allure

Для обеспечения единообразия и удобства анализа тестовых отчётов Allure необходимо выбрать единый язык для всех аннотаций и сообщений, избегать смешивания языков (например, русского и английского) и следить за читаемостью текстов. Названия и описания должны быть короткими, понятными и без технического жаргона, чтобы их могли легко понять QA, разработчики и бизнес. В статье описываются рекомендации по поддержке мультиязычности и улучшению читаемости. Используется технологический стек: Java 17, Maven, SLF4J с Logback, JUnit 5, TestNG, Selenium WebDriver, RestAssured, Allure и Jenkins.

## Рекомендации по мультиязычности и читаемости

### Выбор единого языка для аннотаций и сообщений

- **Единый язык**: Выберите один язык (например, русский или английский) для всех аннотаций (`@Epic`, `@Feature`, `@Story`, `@Step`), сообщений в логах и отчётах Allure.
- **Рекомендация**: Используйте язык, наиболее понятный команде и бизнесу. Например, если бизнес-аналитики и QA работают на русском, выбирайте русский. Если проект международный, предпочтителен английский.
- **Пример (русский)**:
  - `@Epic("Авторизация")`
  - `@Feature("Модальное окно входа")`
  - `@Step("Ввод email: {email}")`
- **Пример (английский)**:
  - `@Epic("Authorization")`
  - `@Feature("Login Modal")`
  - `@Step("Enter email: {email}")`

### Избегать смешивания языков

- **Проблема**: Смешивание русского и английского (например, `@Feature("Login")` и `@Step("Ввод пароля")`) затрудняет чтение отчётов и создаёт путаницу.
- **Как избежать**:
  - Установите стандарт языка в документации проекта.
  - Проверяйте аннотации и сообщения в коде на этапе ревью.
  - Используйте линтеры (например, SonarQube) для выявления смешанных языков.
- **Пример ошибки**: 
  - `@Feature("Login Modal")` + `@Step("Ввод email в поле")` — смешивание языков.
- **Исправление**: 
  - `@Feature("Модальное окно входа")` + `@Step("Ввод email в поле")` — всё на русском.

### Читаемость: короткие, понятные названия, без технического жаргона

- **Короткие названия**: Используйте лаконичные описания для `@Epic`, `@Feature`, `@Story`, `@Step` и сообщений в логах.
  - Пример: Вместо `@Step("Проверка валидации корректности введённого адреса электронной почты")` используйте `@Step("Проверка email")`.
- **Понятность**: Пишите так, чтобы текст был понятен без технических знаний.
  - Пример: Вместо `@Step("Валидация ответа API")` — `@Step("Проверка ответа сервера")`.
- **Без жаргона**: Избегайте терминов, таких как «ассерты», «бэкенд», «DOM», в аннотациях и сообщениях Allure.
  - Пример: Вместо `@Step("Ассерт на наличие элемента")` — `@Step("Проверка отображения элемента")`.

## Примеры реализации

### UI-тест с единым языком и читаемыми аннотациями (русский, JUnit 5)

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
@Story("Вход в систему")
@Owner("qa_team")
@Tag("ui")
@Tag("smoke")
@Severity(SeverityLevel.BLOCKER)
public class UiMultilanguageTest {
    private static final Logger logger = LoggerFactory.getLogger(UiMultilanguageTest.class);
    private WebDriver driver;

    @BeforeEach
    void setUp() {
        logger.info("Инициализация UI-теста");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
    }

    @Test
    @DisplayName("Проверка успешного входа")
    @Description("Тест проверяет вход с корректными данными")
    void testSuccessfulLogin() {
        открытьСтраницуВхода();
        ввестиУчетныеДанные("user@example.com", "password");
        нажатьКнопкуВхода();
        проверитьПереходНаДашборд();
    }

    @Test
    @DisplayName("Проверка ошибки при неверном email")
    @Description("Тест проверяет отображение ошибки для некорректного email")
    void testInvalidEmailFormat() {
        открытьСтраницуВхода();
        ввестиНеверныйEmail("invalid-email");
        нажатьКнопкуВхода();
        проверитьСообщениеОбОшибке();
    }

    @Step("Открытие страницы входа")
    private void открытьСтраницуВхода() {
        logger.info("Открытие страницы: https://example.com/login");
        Allure.step("Открытие страницы входа");
        driver.get("https://example.com/login");
    }

    @Step("Ввод учетных данных: email={email}, пароль=****")
    private void ввестиУчетныеДанные(String email, String password) {
        logger.info("Ввод email: {}, пароль: ****", email);
        Allure.step("Ввод email: " + email);
        // driver.findElement(By.id("email")).sendKeys(email);
        // driver.findElement(By.id("password")).sendKeys(password);
    }

    @Step("Ввод email: {email}")
    private void ввестиНеверныйEmail(String email) {
        logger.info("Ввод email: {}", email);
        Allure.step("Ввод некорректного email: " + email);
        try {
            driver.findElement(By.id("email")).sendKeys(email);
        } catch (NoSuchElementException e) {
            logger.error("Поле email не найдено: {}", e.getMessage(), e);
            прикрепитьСкриншот();
            прикрепитьКодСтраницы();
            прикрепитьСтекОшибки(e);
            throw e;
        }
    }

    @Step("Нажатие кнопки входа")
    private void нажатьКнопкуВхода() {
        logger.info("Нажатие кнопки входа");
        Allure.step("Нажатие кнопки входа");
        try {
            driver.findElement(By.id("loginButton")).click();
        } catch (NoSuchElementException e) {
            logger.error("Кнопка входа не найдена: {}", e.getMessage(), e);
            прикрепитьСкриншот();
            прикрепитьКодСтраницы();
            прикрепитьСтекОшибки(e);
            throw e;
        }
    }

    @Step("Проверка перехода на дашборд")
    private void проверитьПереходНаДашборд() {
        logger.info("Проверка перехода на дашборд");
        Allure.step("Ожидается переход на страницу дашборда");
        try {
            assertThat(driver.getCurrentUrl()).contains("dashboard");
        } catch (AssertionError e) {
            logger.error("Ошибка проверки перехода: {}", e.getMessage(), e);
            прикрепитьСкриншот();
            прикрепитьКодСтраницы();
            прикрепитьСтекОшибки(e);
            throw e;
        }
    }

    @Step("Проверка сообщения об ошибке")
    private void проверитьСообщениеОбОшибке() {
        logger.info("Проверка сообщения об ошибке");
        Allure.step("Ожидается сообщение: Неверный формат email");
        try {
            assertThat(driver.getPageSource()).contains("Неверный формат email");
        } catch (AssertionError e) {
            logger.error("Ошибка проверки сообщения: {}", e.getMessage(), e);
            прикрепитьСкриншот();
            прикрепитьКодСтраницы();
            прикрепитьСтекОшибки(e);
            throw e;
        }
    }

    @Attachment(value = "Скриншот ошибки", type = "image/png")
    private byte[] прикрепитьСкриншот() {
        logger.info("Создание скриншота");
        return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
    }

    @Attachment(value = "Код страницы", type = "text/html")
    private String прикрепитьКодСтраницы() {
        logger.info("Прикрепление кода страницы");
        return driver.getPageSource();
    }

    @Attachment(value = "Стек ошибки", type = "text/plain")
    private String прикрепитьСтекОшибки(Throwable throwable) {
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
- **Единый язык**: Все аннотации, шаги и сообщения на русском (`@Epic("Авторизация")`, `@Step("Ввод email: {email}")`).
- **Читаемость**: Названия шагов короткие и понятные, без жаргона (например, «Проверка перехода на дашборд» вместо «Валидация URL»).
- **Ошибки**: При сбое прикрепляются скриншот, код страницы и стек ошибки с читаемыми описаниями.

### API-тест с единым языком и читаемыми аннотациями (английский, TestNG)

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

@Epic("User Management")
@Feature("User API")
@Story("Retrieve User Data")
@Owner("api_team")
@Tag("api")
@Tag("regression")
@Severity(SeverityLevel.CRITICAL)
public class ApiMultilanguageTest {
    private static final Logger logger = LoggerFactory.getLogger(ApiMultilanguageTest.class);

    @Test
    @Description("Test verifies retrieval of user data by ID")
    @DisplayName("Retrieve user data by ID")
    void testGetUserData() {
        Response response = sendGetRequest("1");
        verifyResponseStatus(response);
        verifyUserId(response);
    }

    @Step("Send GET request for user with ID: {userId}")
    private Response sendGetRequest(String userId) {
        logger.info("Sending GET request: /users/{}", userId);
        Allure.step("Send GET request for user ID: " + userId);
        RestAssured.baseURI = "https://api.example.com";
        Response response;
        try {
            response = given()
                    .when()
                    .get("/users/" + userId)
                    .then()
                    .extract().response();
        } catch (Exception e) {
            logger.error("Request failed: {}", e.getMessage(), e);
            attachStackTrace(e);
            throw e;
        }
        attachResponseJson(response);
        attachCurlRequest(userId);
        return response;
    }

    @Step("Verify response status")
    private void verifyResponseStatus(Response response) {
        logger.info("Verifying response status: {}", response.getStatusCode());
        Allure.step("Expect response status: 200");
        try {
            assertEquals(response.getStatusCode(), 200, "Expected status 200");
        } catch (AssertionError e) {
            logger.error("Status verification failed: {}", e.getMessage(), e);
            attachResponseJson(response);
            attachStackTrace(e);
            throw e;
        }
    }

    @Step("Verify user ID in response")
    private void verifyUserId(Response response) {
        logger.info("Verifying user ID");
        Allure.step("Expect user ID: 1");
        try {
            assertEquals(response.jsonPath().getString("id"), "1");
        } catch (AssertionError e) {
            logger.error("User ID verification failed: {}", e.getMessage(), e);
            attachResponseJson(response);
            attachStackTrace(e);
            throw e;
        }
    }

    @Attachment(value = "Response JSON", type = "application/json")
    private String attachResponseJson(Response response) {
        logger.info("Attaching response JSON");
        return response.asPrettyString();
    }

    @Attachment(value = "curl request", type = "text/plain")
    private String attachCurlRequest(String userId) {
        logger.info("Attaching curl request");
        return "curl -X GET https://api.example.com/users/" + userId;
    }

    @Attachment(value = "Stack trace", type = "text/plain")
    private String attachStackTrace(Throwable throwable) {
        logger.info("Attaching stack trace");
        StringBuilder stackTrace = new StringBuilder(throwable.toString()).append("\n");
        for (StackTraceElement element : throwable.getStackTrace()) {
            stackTrace.append(element.toString()).append("\n");
        }
        return stackTrace.toString();
    }
}
```

**Что происходит**:
- **Единый язык**: Все аннотации и сообщения на английском (`@Epic("User Management")`, `@Step("Send GET request for user with ID: {userId}")`).
- **Читаемость**: Названия шагов короткие и понятные, без технических терминов (например, «Verify response status» вместо «Validate HTTP response code»).
- **Ошибки**: При сбое прикрепляются JSON ответа, curl-запрос и стек ошибки.

## Конфигурация Logback

Логи фиксируют шаги и ошибки в едином языке, дополняя Allure.

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

Allure интегрируется с Jenkins для отображения читаемых отчётов в едином языке.

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

- **Читаемость в Allure**: Аннотации и шаги отображаются в едином языке, упрощая анализ.
- **Логи**: Архивируются в `logs/*.log` для корреляции с отчётами.

## Лучшие практики и отладка

### Единый язык

- **Выбор языка**: Договоритесь о языке на старте проекта и зафиксируйте в документации.
- **Проверка**: На этапе ревью кода проверяйте единообразие языка в аннотациях и логах.
- **Инструменты**: Используйте линтеры (например, SonarQube) для выявления смешанных языков.

### Избегание смешивания языков

- **Стандарты**: Установите правило — один язык для всех тестов и отчётов.
- **Автоматизация**: Настройте CI/CD для проверки аннотаций на консистентность.
- **Пример ошибки**: `@Step("Enter email")` в тесте с `@Feature("Модальное окно входа")`.
- **Исправление**: `@Step("Ввод email")`.

### Читаемость

- **Короткие названия**: Максимум 5–7 слов для `@Step`, `@Feature`, `@Story`.
- **Понятность**: Пишите для бизнес-пользователей, избегая терминов вроде «ассерты» или «бэкенд».
- **Пример**: Вместо `@Step("Проверка HTTP-кода ответа сервера")` — `@Step("Проверка ответа сервера")`.

### Отладка

- **Локальная проверка**: Используйте `mvn allure:serve` для просмотра отчётов и проверки языка/читаемости.
- **Логи**: Анализируйте `logs/tests.log` в IntelliJ IDEA для корреляции с Allure.
- **CI/CD**: Убедитесь, что Jenkins отображает шаги и аннотации в едином языке.

### Распространённые ошибки

- **Смешивание языков**: Например, `@Feature("Login")` и `@Step("Проверка пароля")`.
- **Длинные названия**: Например, `@Step("Проверка корректности отображения сообщения об ошибке валидации")`.
- **Технический жаргон**: Например, `@Step("Ассерт на DOM-элемент")`.

## Дополнительно

### Пример с обработкой ошибок (русский)

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
public class UiErrorMultilanguageTest {
    private static final Logger logger = LoggerFactory.getLogger(UiErrorMultilanguageTest.class);
    private WebDriver driver;

    @BeforeEach
    void setUp() {
        logger.info("Инициализация UI-теста");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
    }

    @Test
    @DisplayName("Проверка ошибки при неверном email")
    void testInvalidEmailFormat() {
        открытьСтраницуВхода();
        ввестиНеверныйEmail("invalid-email");
        нажатьКнопкуВхода();
        проверитьСообщениеОбОшибке();
    }

    @Step("Открытие страницы входа")
    private void открытьСтраницуВхода() {
        logger.info("Открытие страницы: https://example.com/login");
        Allure.step("Открытие страницы входа");
        driver.get("https://example.com/login");
    }

    @Step("Ввод email: {email}")
    private void ввестиНеверныйEmail(String email) {
        logger.info("Ввод email: {}", email);
        Allure.step("Ввод некорректного email: " + email);
        try {
            driver.findElement(By.id("email")).sendKeys(email);
        } catch (NoSuchElementException e) {
            logger.error("Поле email не найдено: {}", e.getMessage(), e);
            прикрепитьСкриншот();
            прикрепитьКодСтраницы();
            прикрепитьСтекОшибки(e);
            throw e;
        }
    }

    @Step("Нажатие кнопки входа")
    private void нажатьКнопкуВхода() {
        logger.info("Нажатие кнопки входа");
        Allure.step("Нажатие кнопки входа");
        try {
            driver.findElement(By.id("loginButton")).click();
        } catch (NoSuchElementException e) {
            logger.error("Кнопка входа не найдена: {}", e.getMessage(), e);
            прикрепитьСкриншот();
            прикрепитьКодСтраницы();
            прикрепитьСтекОшибки(e);
            throw e;
        }
    }

    @Step("Проверка сообщения об ошибке")
    private void проверитьСообщениеОбОшибке() {
        logger.info("Проверка сообщения об ошибке");
        Allure.step("Ожидается сообщение: Неверный формат email");
        try {
            assertThat(driver.getPageSource()).contains("Неверный формат email");
        } catch (AssertionError e) {
            logger.error("Ошибка проверки сообщения: {}", e.getMessage(), e);
            прикрепитьСкриншот();
            прикрепитьКодСтраницы();
            прикрепитьСтекОшибки(e);
            throw e;
        }
    }

    @Attachment(value = "Скриншот ошибки", type = "image/png")
    private byte[] прикрепитьСкриншот() {
        logger.info("Создание скриншота");
        return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
    }

    @Attachment(value = "Код страницы", type = "text/html")
    private String прикрепитьКодСтраницы() {
        logger.info("Прикрепление кода страницы");
        return driver.getPageSource();
    }

    @Attachment(value = "Стек ошибки", type = "text/plain")
    private String прикрепитьСтекОшибки(Throwable throwable) {
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

## Источники

- [Allure Framework](https://allurereport.org/docs/)
- [SLF4J User Manual](http://www.slf4j.org/manual.html)
- [Logback Documentation](https://logback.qos.ch/documentation.html)
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/webdriver/)
- [RestAssured Documentation](https://rest-assured.io/)
- [Jenkins Pipeline Documentation](https://www.jenkins.io/doc/book/pipeline/)

---
[**← Назад к оглавлению**](../README.md)