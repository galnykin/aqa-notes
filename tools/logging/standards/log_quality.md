# Качество логов в тестировании

Качественное логирование повышает читаемость и полезность логов, упрощая анализ и отладку тестов. В статье рассматриваются подходы к избеганию дублирования, исключению ненужных записей вроде "успешно" без проверок или "ничего не произошло", а также логирование только значимой информации. Используется технологический стек: Java 17, Maven, SLF4J с Logback, JUnit 5, TestNG, Selenium WebDriver, RestAssured и Allure.

## Избегание дублирования логов

Дублирование логов усложняет анализ и увеличивает объём данных. Для предотвращения дублирования используйте уникальные идентификаторы операций и фильтры.

### Пример утилитного класса для логирования

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.HashSet;
import java.util.Set;

public class TestLogger {
    private static final Logger logger = LoggerFactory.getLogger(TestLogger.class);
    private static final Set<String> loggedEvents = new HashSet<>();

    public static void logUniqueAction(String action, String details) {
        String logKey = action + ":" + details;
        if (loggedEvents.add(logKey)) {
            logger.info("Действие: {} - {}", action, details);
        } else {
            logger.debug("Пропущено дублирование: {} - {}", action, details);
        }
    }

    public static void clearLoggedEvents() {
        loggedEvents.clear();
    }
}
```

- `loggedEvents`: Хранит уникальные ключи для предотвращения дублирования.
- Логи уровня `DEBUG` для дублирующихся событий минимизируют "шум" в логах.

## Не логировать "успешно" без проверки

Логирование "успешно" без проверки результата создаёт ложное впечатление надёжности. Логируйте только после выполнения проверки (например, с использованием AssertJ или TestNG).

### Пример теста с JUnit 5

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import static org.assertj.core.api.Assertions.assertThat;

public class LoginTest {
    private static final Logger logger = LoggerFactory.getLogger(LoginTest.class);
    private WebDriver driver;

    @BeforeEach
    void setUp() {
        logger.info("Инициализация теста");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        TestLogger.clearLoggedEvents();
    }

    @Test
    void testLogin() {
        logger.info("Запуск теста логина");
        driver.get("https://example.com/login");
        TestLogger.logUniqueAction("Открытие страницы", "https://example.com/login");

        // Ввод данных
        TestLogger.logUniqueAction("Ввод имени пользователя", "user");
        // Пропущено: logger.info("Ввод успешен") — нет проверки

        // Проверка
        String currentUrl = driver.getCurrentUrl();
        assertThat(currentUrl).contains("login");
        TestLogger.logUniqueAction("Проверка URL", "Содержит 'login'");
    }

    @AfterEach
    void tearDown() {
        logger.info("Завершение теста");
        if (driver != null) {
            driver.quit();
        }
    }
}
```

Логирование "успешно" происходит только после проверки `assertThat`.

### Пример теста с TestNG

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;

import static org.testng.Assert.assertTrue;

public class LoginTestNG {
    private static final Logger logger = LoggerFactory.getLogger(LoginTestNG.class);
    private WebDriver driver;

    @BeforeMethod
    void setUp() {
        logger.info("Инициализация теста");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        TestLogger.clearLoggedEvents();
    }

    @Test
    void testLogin() {
        logger.info("Запуск теста логина");
        driver.get("https://example.com/login");
        TestLogger.logUniqueAction("Открытие страницы", "https://example.com/login");

        // Ввод данных
        TestLogger.logUniqueAction("Ввод имени пользователя", "user");
        // Пропущено: logger.info("Ввод успешен") — нет проверки

        // Проверка
        String currentUrl = driver.getCurrentUrl();
        assertTrue(currentUrl.contains("login"));
        TestLogger.logUniqueAction("Проверка URL", "Содержит 'login'");
    }

    @AfterMethod
    void tearDown() {
        logger.info("Завершение теста");
        if (driver != null) {
            driver.quit();
        }
    }
}
```

## Не логировать "ничего не произошло"

Логи типа "ничего не произошло" или "действие не выполнено" не несут ценности и загромождают вывод. Логируйте только действия, которые влияют на тест.

### Пример Page Object с выборочным логированием

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.time.Duration;

public class LoginPage {
    private static final Logger logger = LoggerFactory.getLogger(LoginPage.class);
    private final WebDriver driver;
    private final WebDriverWait wait;

    @FindBy(id = "username")
    private WebElement usernameField;

    @FindBy(id = "loginButton")
    private WebElement loginButton;

    public LoginPage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        PageFactory.initElements(driver, this);
    }

    public void login(String username) {
        TestLogger.logUniqueAction("Открытие страницы", "https://example.com/login");
        driver.get("https://example.com/login");

        if (wait.until(ExpectedConditions.visibilityOf(usernameField)).isDisplayed()) {
            TestLogger.logUniqueAction("Ввод имени пользователя", username);
            usernameField.sendKeys(username);
        } else {
            // Не логируем "ничего не произошло"
            logger.warn("Элемент usernameField не отображается");
        }

        if (wait.until(ExpectedConditions.elementToBeClickable(loginButton)).isEnabled()) {
            TestLogger.logUniqueAction("Клик по кнопке входа", "loginButton");
            loginButton.click();
        } else {
            // Не логируем "ничего не произошло"
            logger.warn("Кнопка loginButton неактивна");
        }
    }
}
```

Логирование происходит только для успешных действий или значимых ошибок.

## Логировать только полезную информацию

Логи должны содержать информацию, необходимую для анализа: действия, входные данные, результаты проверок и ошибки.

### Пример API-теста с RestAssured

```java
import io.restassured.RestAssured;
import io.restassured.response.Response;
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import static io.restassured.RestAssured.given;
import static org.assertj.core.api.Assertions.assertThat;

public class ApiTest {
    private static final Logger logger = LoggerFactory.getLogger(ApiTest.class);

    @Test
    void testApiGet() {
        logger.info("Запуск теста API");
        RestAssured.baseURI = "https://api.example.com";

        TestLogger.logUniqueAction("Отправка GET-запроса", "/users/1");
        Response response = given()
                .when()
                .get("/users/1")
                .then()
                .extract().response();

        TestLogger.logUniqueAction("Проверка статуса", String.valueOf(response.getStatusCode()));
        assertThat(response.getStatusCode()).isEqualTo(200);
    }
}
```

Логи содержат только ключевые шаги и результаты, избегая лишних деталей.

## Интеграция с Allure

Allure помогает структурировать логи в отчётах, исключая ненужные записи.

**Пример теста с Allure**:

```java
import io.qameta.allure.Allure;
import io.qameta.allure.Step;
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class AllureTest {
    private static final Logger logger = LoggerFactory.getLogger(AllureTest.class);

    @Test
    void testWithQualityLogs() {
        logStep("Запуск теста");
        performAction("Ввод данных", "user");
        verifyResult("URL", "https://example.com/login");
    }

    @Step("Шаг: {step}")
    private void logStep(String step) {
        TestLogger.logUniqueAction("Шаг теста", step);
    }

    @Step("Действие: {action}, данные: {data}")
    private void performAction(String action, String data) {
        TestLogger.logUniqueAction(action, data);
        // Логика действия
    }

    @Step("Проверка: {check}, ожидается: {expected}")
    private void verifyResult(String check, String expected) {
        TestLogger.logUniqueAction("Проверка результата", check + " = " + expected);
        Allure.addAttachment("Результат", expected);
    }
}
```

**Зависимости для Allure**:

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
```

## Лучшие практики и отладка

### Избегание дублирования

- Используйте `Set` или другие структуры данных для отслеживания логов.
- Настройте уровень `DEBUG` для дублирующихся событий, чтобы они не попадали в основные логи.

### Логирование проверок

- Логируйте "успешно" только после выполнения утверждений (`assertThat`, `assertTrue`).
- Избегайте логов для промежуточных действий без результата.

### Полезность логов

- Логируйте действия, входные данные, ожидаемые и фактические результаты.
- Используйте MDC (Mapped Diagnostic Context) для добавления контекста, например, `testId`.

### Отладка

- В IntelliJ IDEA используйте точки останова для проверки логов перед их записью.
- Анализируйте Allure-отчёты для проверки структуры логов.

### Распространённые ошибки

- **Дублирование логов**: Отсутствие фильтрации приводит к "шуму".
- **Логирование без проверки**: Сообщения "успешно" без `assert` вводят в заблуждение.
- **Избыточные логи**: Логирование незначительных событий усложняет анализ.

## Дополнительно

### Интеграция с Jenkins

Настройте Jenkins для архивирования логов и генерации Allure-отчётов:

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
            archiveArtifacts artifacts: 'logs/*.log', allowEmptyArchive: true
        }
    }
}
```

### Использование Testcontainers

Testcontainers полезен для проверки логов в интеграционных тестах.

```java
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testcontainers.containers.GenericContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

@Testcontainers
public class LogTest {
    private static final Logger logger = LoggerFactory.getLogger(LogTest.class);

    @Container
    private static final GenericContainer<?> container = new GenericContainer<>("httpd:2.4")
            .withExposedPorts(80);

    @Test
    void testContainer() {
        TestLogger.logUniqueAction("Запуск контейнера", container.getContainerId());
        logger.info("Контейнер запущен на {}:{}", container.getHost(), container.getFirstMappedPort());
        // Проверка без логирования "успешно"
        String address = "http://" + container.getHost() + ":" + container.getFirstMappedPort();
        TestLogger.logUniqueAction("Проверка адреса", address);
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