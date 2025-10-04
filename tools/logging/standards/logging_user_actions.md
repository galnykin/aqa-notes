# Логирование действий

Логирование в автоматизированных тестах необходимо для отслеживания действий, таких как ввод данных, клики, переходы, загрузки, а также фиксации начала и завершения операций и их результатов. Это помогает в отладке, анализе ошибок и создании отчётов. В статье рассматривается настройка логирования с использованием SLF4J и Logback, интеграция с тестами на JUnit 5 и TestNG, а также лучшие практики для UI- и API-тестирования с использованием Selenium WebDriver и RestAssured.

## Настройка логирования с SLF4J и Logback

SLF4J (Simple Logging Facade for Java) используется как фасад для различных систем логирования, а Logback — как реализация. Для начала необходимо настроить зависимости в Maven.

### Зависимости в Maven

Добавьте зависимости в `pom.xml` для SLF4J и Logback:

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
</dependencies>
```

### Конфигурация Logback

Создайте файл `logback.xml` в папке `src/main/resources`:

```xml
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
        <file>logs/test.log</file>
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

Этот файл настраивает вывод логов в консоль и файл `logs/test.log` с уровнем логирования `INFO`.

### Логирование в тестах

Создайте класс для логирования действий. Например, используйте SLF4J `Logger`:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class TestLogger {
    private static final Logger logger = LoggerFactory.getLogger(TestLogger.class);

    public static void logStart(String operation) {
        logger.info("Начало операции: {}", operation);
    }

    public static void logAction(String action, String details) {
        logger.info("Действие: {} - {}", action, details);
    }

    public static void logResult(String operation, boolean success) {
        if (success) {
            logger.info("Операция '{}' завершена успешно", operation);
        } else {
            logger.error("Операция '{}' завершилась с ошибкой", operation);
        }
    }
}
```

## Логирование действий в UI-тестах с Selenium WebDriver

Для UI-тестирования используем Selenium WebDriver с Page Object Model. Логируем действия, такие как ввод данных, клики, переходы и загрузки страниц, а также фиксируем начало и завершение операций.

### Пример теста на JUnit 5

Создайте Page Object для страницы логина:

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LoginPage {
    private static final Logger logger = LoggerFactory.getLogger(LoginPage.class);
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
        TestLogger.logStart("Вход в систему");
        logger.info("Открытие страницы логина");
        driver.get("https://example.com/login");

        TestLogger.logAction("Ввод имени пользователя", username);
        wait.until(ExpectedConditions.visibilityOf(usernameField)).sendKeys(username);

        TestLogger.logAction("Ввод пароля", password);
        wait.until(ExpectedConditions.visibilityOf(passwordField)).sendKeys(password);

        TestLogger.logAction("Клик по кнопке входа", "loginButton");
        wait.until(ExpectedConditions.elementToBeClickable(loginButton)).click();

        TestLogger.logResult("Вход в систему", true);
    }
}
```

Тест с использованием JUnit 5 и WebDriverManager:

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
    private LoginPage loginPage;

    @BeforeEach
    void setUp() {
        TestLogger.logStart("Инициализация теста логина");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        loginPage = new LoginPage(driver);
        TestLogger.logResult("Инициализация теста логина", true);
    }

    @Test
    void testSuccessfulLogin() {
        TestLogger.logStart("Тест успешного логина");
        loginPage.login("user", "password");
        assertThat(driver.getCurrentUrl()).contains("dashboard");
        TestLogger.logResult("Тест успешного логина", true);
    }

    @AfterEach
    void tearDown() {
        TestLogger.logStart("Завершение теста");
        if (driver != null) {
            driver.quit();
        }
        TestLogger.logResult("Завершение теста", true);
    }
}
```

### Пример теста на TestNG

Тот же функционал, но с использованием TestNG:

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
    private LoginPage loginPage;

    @BeforeMethod
    void setUp() {
        TestLogger.logStart("Инициализация теста логина");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        loginPage = new LoginPage(driver);
        TestLogger.logResult("Инициализация теста логина", true);
    }

    @Test
    void testSuccessfulLogin() {
        TestLogger.logStart("Тест успешного логина");
        loginPage.login("user", "password");
        assertTrue(driver.getCurrentUrl().contains("dashboard"));
        TestLogger.logResult("Тест успешного логина", true);
    }

    @AfterMethod
    void tearDown() {
        TestLogger.logStart("Завершение теста");
        if (driver != null) {
            driver.quit();
        }
        TestLogger.logResult("Завершение теста", true);
    }
}
```

### Логирование в API-тестах с RestAssured

Для API-тестирования логируем запросы, ответы и результаты операций.

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
        TestLogger.logStart("Тест API GET-запроса");
        RestAssured.baseURI = "https://api.example.com";

        TestLogger.logAction("Отправка GET-запроса", "/users/1");
        Response response = given()
                .when()
                .get("/users/1")
                .then()
                .extract().response();

        TestLogger.logAction("Получен ответ", response.asString());
        assertThat(response.getStatusCode()).isEqualTo(200);
        TestLogger.logResult("Тест API GET-запроса", true);
    }
}
```

## Лучшие практики и отладка

### Стабильность тестов

- **Ожидания**: Используйте `WebDriverWait` для явных ожиданий, чтобы избежать `flaky tests`. Например, ждите видимость элемента перед взаимодействием.
- **Локаторы**: Предпочитайте стабильные локаторы, такие как `id` или `data-*` атрибуты, чтобы минимизировать влияние изменений в DOM.
- **Проверки**: Используйте методы `isDisplayed()` и `isEnabled()` перед действиями с элементами.

### Отладка

- В IntelliJ IDEA настройте точки останова в тестах или Page Object классах. Проверяйте значения локаторов и состояние элементов в дебаггере.
- Логируйте полные сообщения об ошибках (например, `logger.error("Ошибка: ", e)`), чтобы упростить анализ.

### Распространённые ошибки

- **Отсутствие ожиданий**: Взаимодействие с элементами до их готовности вызывает `NoSuchElementException`. Всегда используйте `WebDriverWait`.
- **Неправильные локаторы**: Изменения в интерфейсе ломают тесты. Проверяйте локаторы в дебаггере или через DevTools.
- **Перегрузка логов**: Логируйте только ключевые действия, чтобы избежать "шума" в логах.

## Интеграция с Allure для отчётности

Allure позволяет создавать отчёты с логами действий. Добавьте зависимость в `pom.xml`:

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

Добавьте аннотации в тесты:

```java
import io.qameta.allure.Allure;
import io.qameta.allure.Description;
import org.junit.jupiter.api.Test;

public class AllureTest {
    @Test
    @Description("Тест с Allure")
    void testWithAllure() {
        Allure.step("Шаг 1: Ввод данных", () -> {
            TestLogger.logAction("Ввод данных", "user");
            // Действие
        });
        Allure.step("Шаг 2: Проверка результата", () -> {
            TestLogger.logResult("Проверка результата", true);
            // Проверка
        });
    }
}
```

## Дополнительно

### Интеграция с Jenkins

Для запуска тестов в Jenkins настройте Maven job:

1. Создайте файл `Jenkinsfile`:
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

2. Установите плагин Allure в Jenkins для отображения отчётов.

### Использование Testcontainers

Testcontainers полезен для API-тестирования, если требуется запуск сервиса в контейнере. Пример:

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
        TestLogger.logStart("Тест контейнера");
        String address = "http://" + container.getHost() + ":" + container.getFirstMappedPort();
        TestLogger.logAction("Запрос к контейнеру", address);
        // Выполнить запрос
        TestLogger.logResult("Тест контейнера", true);
    }
}
```

### Использование Lombok для упрощения кода

Lombok сокращает boilerplate-код. Добавьте зависимость:

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

## Источники

- [SLF4J User Manual](http://www.slf4j.org/manual.html)
- [Logback Documentation](https://logback.qos.ch/documentation.html)
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/webdriver/)
- [RestAssured Documentation](https://rest-assured.io/)
- [Allure Framework](https://allurereport.org/docs/)
- [Testcontainers Documentation](https://testcontainers.org/)

---
[**← Назад к оглавлению**](../README.md)