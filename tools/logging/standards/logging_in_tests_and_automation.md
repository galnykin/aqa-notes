# Логирование шагов в автоматизированных тестах

Логирование шагов теста, входных данных и ожидаемых результатов упрощает отладку, анализ результатов и создание отчётов. В статье рассматривается настройка логирования с использованием SLF4J и Logback, интеграция с Allure для отчётности и настройка CI/CD для генерации отчётов. Используется технологический стек: Java 17, Maven, JUnit 5, TestNG, Selenium WebDriver, RestAssured и Allure.

## Настройка логирования с SLF4J и Logback

SLF4J используется как фасад для логирования, а Logback — как реализация. Логирование шагов теста включает фиксацию действий, входных данных и ожидаемых результатов.

### Зависимости в Maven

Добавьте зависимости в `pom.xml`:

```xml
<dependencies>
    <!-- SLF4J API -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>2.0.16</version>
    </dependency>
    <!-- Logback Classic -->
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.5.8</version>
    </dependency>
    <!-- Allure JUnit 5 -->
    <dependency>
        <groupId>io.qameta.allure</groupId>
        <artifactId>allure-junit5</artifactId>
        <version>2.29.0</version>
    </dependency>
    <!-- Allure TestNG -->
    <dependency>
        <groupId>io.qameta.allure</groupId>
        <artifactId>allure-testng</artifactId>
        <version>2.29.0</version>
    </dependency>
</dependencies>
```

### Конфигурация Logback

Создайте файл `logback.xml` в `src/main/resources`:

```xml
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
        <file>logs/test_steps.log</file>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="info">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

## Логирование шагов теста

Логирование шагов теста включает фиксацию каждого действия, входных данных и ожидаемых результатов. Создайте утилитный класс для структурированного логирования:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class TestStepLogger {
    private static final Logger logger = LoggerFactory.getLogger(TestStepLogger.class);

    public static void logStep(String stepDescription) {
        logger.info("Шаг теста: {}", stepDescription);
    }

    public static void logInputData(String inputDescription, Object data) {
        logger.info("Входные данные: {} = {}", inputDescription, data);
    }

    public static void logExpectedResult(String expectedDescription, Object expected) {
        logger.info("Ожидаемый результат: {} = {}", expectedDescription, expected);
    }

    public static void logActualResult(String actualDescription, Object actual) {
        logger.info("Фактический результат: {} = {}", actualDescription, actual);
    }
}
```

### Пример логирования в UI-тестах с Selenium WebDriver

Создайте Page Object для страницы логина с логированием шагов:

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;

public class LoginPage {
    private final WebDriver driver;
    private final WebDriverWait wait;

    @FindBy(id = "username")
    private WebElement usernameField;

    @FindBy(id = "password")
    private WebElement passwordField;

    @FindBy(id = "loginButton")
    private WebElement loginButton;

    public LoginPage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        PageFactory.initElements(driver, this);
    }

    public void login(String username, String password) {
        TestStepLogger.logStep("Открытие страницы логина");
        driver.get("https://example.com/login");

        TestStepLogger.logInputData("Имя пользователя", username);
        wait.until(ExpectedConditions.visibilityOf(usernameField)).sendKeys(username);

        TestStepLogger.logInputData("Пароль", password);
        wait.until(ExpectedConditions.visibilityOf(passwordField)).sendKeys(password);

        TestStepLogger.logStep("Клик по кнопке входа");
        wait.until(ExpectedConditions.elementToBeClickable(loginButton)).click();

        TestStepLogger.logExpectedResult("URL после входа", "https://example.com/dashboard");
    }
}
```

Пример теста на JUnit 5:

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

import static org.assertj.core.api.Assertions.assertThat;

public class LoginTest {
    private WebDriver driver;
    private LoginPage loginPage;

    @BeforeEach
    void setUp() {
        TestStepLogger.logStep("Инициализация теста логина");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        loginPage = new LoginPage(driver);
    }

    @Test
    void testSuccessfulLogin() {
        TestStepLogger.logStep("Запуск теста успешного логина");
        loginPage.login("user", "password");
        String actualUrl = driver.getCurrentUrl();
        TestStepLogger.logActualResult("URL после входа", actualUrl);
        assertThat(actualUrl).contains("dashboard");
    }

    @AfterEach
    void tearDown() {
        TestStepLogger.logStep("Завершение теста");
        if (driver != null) {
            driver.quit();
        }
    }
}
```

Пример теста на TestNG:

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;

import static org.testng.Assert.assertTrue;

public class LoginTestNG {
    private WebDriver driver;
    private LoginPage loginPage;

    @BeforeMethod
    void setUp() {
        TestStepLogger.logStep("Инициализация теста логина");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        loginPage = new LoginPage(driver);
    }

    @Test
    void testSuccessfulLogin() {
        TestStepLogger.logStep("Запуск теста успешного логина");
        loginPage.login("user", "password");
        String actualUrl = driver.getCurrentUrl();
        TestStepLogger.logActualResult("URL после входа", actualUrl);
        assertTrue(actualUrl.contains("dashboard"));
    }

    @AfterMethod
    void tearDown() {
        TestStepLogger.logStep("Завершение теста");
        if (driver != null) {
            driver.quit();
        }
    }
}
```

### Пример логирования в API-тестах с RestAssured

Логирование шагов теста для API включает запросы, входные данные и ожидаемые результаты.

```java
import io.restassured.RestAssured;
import io.restassured.response.Response;
import org.junit.jupiter.api.Test;

import static io.restassured.RestAssured.given;
import static org.assertj.core.api.Assertions.assertThat;

public class ApiTest {
    @Test
    void testApiGet() {
        TestStepLogger.logStep("Запуск теста API GET-запроса");
        RestAssured.baseURI = "https://api.example.com";

        TestStepLogger.logInputData("Эндпоинт", "/users/1");
        Response response = given()
                .when()
                .get("/users/1")
                .then()
                .extract().response();

        TestStepLogger.logExpectedResult("Статус код", 200);
        TestStepLogger.logActualResult("Статус код", response.getStatusCode());
        assertThat(response.getStatusCode()).isEqualTo(200);
    }
}
```

## Интеграция с Allure

Allure позволяет создавать структурированные отчёты с шагами теста, входными данными и результатами.

### Настройка Allure

Добавьте зависимости в `pom.xml` (уже указаны выше). Пример теста с Allure:

```java
import io.qameta.allure.Allure;
import io.qameta.allure.Description;
import io.qameta.allure.Step;
import org.junit.jupiter.api.Test;

public class AllureTest {
    @Test
    @Description("Тест с логированием шагов в Allure")
    void testWithAllure() {
        loginStep("user", "password");
        verifyResultStep("https://example.com/dashboard");
    }

    @Step("Шаг: Вход с данными username={username}, password={password}")
    private void loginStep(String username, String password) {
        TestStepLogger.logStep("Вход в систему");
        TestStepLogger.logInputData("Имя пользователя", username);
        TestStepLogger.logInputData("Пароль", password);
        // Логика входа
        Allure.addAttachment("Входные данные", "username: " + username + ", password: " + password);
    }

    @Step("Шаг: Проверка результата, ожидаемый URL={expectedUrl}")
    private void verifyResultStep(String expectedUrl) {
        TestStepLogger.logExpectedResult("URL после входа", expectedUrl);
        TestStepLogger.logActualResult("URL после входа", "https://example.com/dashboard");
        // Проверка
    }
}
```

Allure отображает шаги, входные данные и результаты в отчёте, включая прикреплённые данные.

## Интеграция с CI/CD и отчётами

Для автоматизации запуска тестов и генерации отчётов настройте Jenkins.

### Настройка Jenkins

Создайте `Jenkinsfile` для запуска тестов и генерации отчётов Allure:

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
}
```

Установите плагин Allure в Jenkins для просмотра отчётов. Логи из `logback.xml` и результаты Allure сохраняются в `target/allure-results` и доступны в CI.

## Лучшие практики и отладка

### Логирование шагов

- Логируйте каждый шаг теста с чётким описанием (например, "Открытие страницы", "Ввод данных").
- Фиксируйте входные данные и ожидаемые результаты перед выполнением действий.
- Используйте шаблон AAA (Arrange-Act-Assert) для структурирования тестов и логов.

### Стабильность тестов

- Используйте `WebDriverWait` для UI-тестов, чтобы дождаться готовности элементов.
- Применяйте стабильные локаторы (`id`, `data-*`) для избежания `NoSuchElementException`.
- Проверяйте состояние элементов (`isDisplayed`, `isEnabled`) перед действиями.

### Отладка

- В IntelliJ IDEA настройте точки останова в методах Page Object или тестов для проверки входных данных и локаторов.
- Используйте логи для анализа фактических и ожидаемых результатов.

### Распространённые ошибки

- **Недостаточное логирование**: Пропуск шагов или входных данных затрудняет анализ ошибок.
- **Перегрузка логов**: Избегайте избыточного логирования некритичных деталей.
- **Игнорирование Allure**: Неправильная настройка аннотаций `@Step` может привести к неполным отчётам.

## Дополнительно

### Использование Lombok

Lombok упрощает код Page Object. Добавьте зависимость:

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.34</version>
    <scope>provided</scope>
</dependency>
```

Пример Page Object с Lombok:

```java
import lombok.Getter;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;

@Getter
public class LoginPageLombok {
    @FindBy(id = "username")
    private WebElement usernameField;

    @FindBy(id = "password")
    private WebElement passwordField;

    public LoginPageLombok(WebDriver driver) {
        PageFactory.initElements(driver, this);
    }
}
```

### Использование Testcontainers

Testcontainers полезен для API-тестов. Пример:

```java
import org.junit.jupiter.api.Test;
import org.testcontainers.containers.GenericContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

@Testcontainers
public class ContainerTest {
    @Container
    private static final GenericContainer<?> container = new GenericContainer<>("httpd:2.4")
            .withExposedPorts(80);

    @Test
    void testContainer() {
        TestStepLogger.logStep("Запуск теста контейнера");
        String address = "http://" + container.getHost() + ":" + container.getFirstMappedPort();
        TestStepLogger.logInputData("Адрес контейнера", address);
        TestStepLogger.logExpectedResult("Статус код", 200);
        // Выполнить запрос
        TestStepLogger.logActualResult("Статус код", 200);
    }
}
```

## Источники

- [SLF4J User Manual](http://www.slf4j.org/manual.html)
- [Logback Documentation](https://logback.qos.ch/documentation.html)
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/webdriver/)
- [RestAssured Documentation](https://rest-assured.io/)
- [Allure Framework](https://allurereport.org/docs/)
- [Testcontainers Documentation](https://testcontainers.org/)

---
[**← Назад к оглавлению**](../README.md)