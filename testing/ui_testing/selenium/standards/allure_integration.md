# Интеграция с Allure

Allure — мощный инструмент для создания информативных отчётов в автоматизации тестирования. Он позволяет добавлять шаги, скриншоты и вложения (HTML, JSON, логи, curl-запросы) для упрощения анализа результатов тестов. В статье описаны подходы к интеграции Allure с Java, JUnit 5, TestNG, Selenium WebDriver и RestAssured, включая добавление шагов, автоматическое создание скриншотов при падении тестов и прикрепление различных типов вложений.

## Настройка Allure

Для использования Allure необходимо добавить зависимости в `pom.xml` и настроить плагин для генерации отчётов.

### Зависимости в Maven

```xml
<dependencies>
    <dependency>
        <groupId>io.qameta.allure</groupId>
        <artifactId>allure-junit5</artifactId>
        <version>2.30.0</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>io.qameta.allure</groupId>
        <artifactId>allure-testng</artifactId>
        <version>2.30.0</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>io.qameta.allure</groupId>
        <artifactId>allure-rest-assured</artifactId>
        <version>2.30.0</version>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.3.1</version>
            <configuration>
                <systemPropertyVariables>
                    <allure.results.directory>target/allure-results</allure.results.directory>
                </systemPropertyVariables>
            </configuration>
        </plugin>
        <plugin>
            <groupId>io.qameta.allure</groupId>
            <artifactId>allure-maven</artifactId>
            <version>2.12.0</version>
            <executions>
                <execution>
                    <goals>
                        <goal>report</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

### Конфигурация Logback для логов

Добавьте файл `logback.xml` в `src/main/resources` для логирования:

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

## Добавление шагов в Allure

Шаги в Allure добавляются с помощью аннотации `@Step` или метода `Allure.step()`. Это улучшает читаемость отчётов, разбивая тесты на логические блоки.

### Использование аннотации `@Step`

```java
import io.qameta.allure.Step;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LoginPage {
    private static final Logger logger = LoggerFactory.getLogger(LoginPage.class);
    private final WebDriver driver;

    @FindBy(id = "username")
    private WebElement usernameField;

    @FindBy(id = "password")
    private WebElement passwordField;

    @FindBy(id = "login-button")
    private WebElement loginButton;

    public LoginPage(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this);
    }

    @Step("Opening login page: {url}")
    public void openPage(String url) {
        logger.info("Navigating to login page: {}", url);
        driver.get(url);
    }

    @Step("Logging in as user: {username}")
    public void loginAs(String username, String password) {
        logger.info("Entering username: {}", username);
        usernameField.sendKeys(username);
        logger.info("Entering password: [PROTECTED]");
        passwordField.sendKeys(password);
        logger.info("Clicking login button");
        loginButton.click();
    }
}
```

### Использование `Allure.step()`

Для более гибкого подхода можно использовать `Allure.step()` внутри методов.

```java
import io.qameta.allure.Allure;
import org.openqa.selenium.WebDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class NavigationUtil {
    private static final Logger logger = LoggerFactory.getLogger(NavigationUtil.class);
    private final WebDriver driver;

    public NavigationUtil(WebDriver driver) {
        this.driver = driver;
    }

    public void navigateTo(String url) {
        Allure.step("Navigating to URL: " + url, () -> {
            logger.info("Navigating to URL: {}", url);
            driver.get(url);
        });
    }
}
```

### Особенности
- **Аннотация `@Step`**: Подходит для методов в Page Object или утилитах, автоматически добавляет шаги в отчёт.
- **Метод `Allure.step()`**: Полезен для добавления шагов в произвольных местах кода.
- **Логирование**: Все шаги сопровождаются логами через SLF4J.

## Скриншоты при падении теста

Для автоматического создания скриншотов при падении тестов используется интерфейс `TakesScreenshot` и `Allure.addAttachment()`.

### Реализация слушателя для JUnit 5

```java
import io.qameta.allure.Allure;
import org.junit.jupiter.api.extension.AfterTestExecutionCallback;
import org.junit.jupiter.api.extension.ExtensionContext;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class ScreenshotListener implements AfterTestExecutionCallback {
    private static final Logger logger = LoggerFactory.getLogger(ScreenshotListener.class);
    private final WebDriver driver;

    public ScreenshotListener(WebDriver driver) {
        this.driver = driver;
    }

    @Override
    public void afterTestExecution(ExtensionContext context) {
        boolean testFailed = context.getExecutionException().isPresent();
        if (testFailed && driver instanceof TakesScreenshot) {
            byte[] screenshot = ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
            Allure.addAttachment("Screenshot on failure", "image/png", screenshot, ".png");
            logger.info("Screenshot captured and attached to Allure report");
        }
    }
}
```

### Регистрация слушателя в JUnit 5

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.RegisterExtension;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import static org.assertj.core.api.Assertions.assertThat;

public class LoginTest {
    private static final Logger logger = LoggerFactory.getLogger(LoginTest.class);
    private WebDriver driver;
    private LoginPage loginPage;

    @RegisterExtension
    ScreenshotListener screenshotListener;

    @BeforeEach
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        screenshotListener = new ScreenshotListener(driver);
        loginPage = new LoginPage(driver);
        logger.info("WebDriver initialized");
    }

    @Test
    void testFailedLogin() {
        loginPage.openPage("https://example.com/login");
        loginPage.loginAs("wronguser", "wrongpass");
        assertThat(driver.getCurrentUrl()).contains("dashboard"); // Намеренно проваливаем тест
    }

    @AfterEach
    void tearDown() {
        logger.info("Closing WebDriver");
        if (driver != null) {
            driver.quit();
        }
    }
}
```

### Реализация слушателя для TestNG

```java
import io.qameta.allure.Allure;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testng.ITestListener;
import org.testng.ITestResult;

public class TestNGAllureListener implements ITestListener {
    private static final Logger logger = LoggerFactory.getLogger(TestNGAllureListener.class);
    private final WebDriver driver;

    public TestNGAllureListener(WebDriver driver) {
        this.driver = driver;
    }

    @Override
    public void onTestFailure(ITestResult result) {
        if (driver instanceof TakesScreenshot) {
            byte[] screenshot = ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
            Allure.addAttachment("Screenshot on failure", "image/png", screenshot, ".png");
            logger.info("Screenshot captured and attached to Allure report");
        }
    }
}
```

### Регистрация слушателя в TestNG

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Listeners;
import org.testng.annotations.Test;

import static org.assertj.core.api.Assertions.assertThat;

@Listeners(TestNGAllureListener.class)
public class LoginTestNG {
    private static final Logger logger = LoggerFactory.getLogger(LoginTestNG.class);
    private WebDriver driver;
    private LoginPage loginPage;

    @BeforeMethod
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        loginPage = new LoginPage(driver);
        logger.info("WebDriver initialized");
    }

    @Test
    void testFailedLogin() {
        loginPage.openPage("https://example.com/login");
        loginPage.loginAs("wronguser", "wrongpass");
        assertThat(driver.getCurrentUrl()).contains("dashboard"); // Намеренно проваливаем тест
    }

    @AfterMethod
    void tearDown() {
        logger.info("Closing WebDriver");
        if (driver != null) {
            driver.quit();
        }
    }
}
```

## Вложение HTML, JSON, логов и curl-запросов

Allure позволяет прикреплять различные типы данных, такие как HTML, JSON, логи и curl-запросы, для дополнительной информации в отчётах.

### Вложение HTML

```java
import io.qameta.allure.Allure;
import org.openqa.selenium.WebDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HtmlAttachmentUtil {
    private static final Logger logger = LoggerFactory.getLogger(HtmlAttachmentUtil.class);

    public static void attachPageSource(WebDriver driver) {
        String pageSource = driver.getPageSource();
        Allure.addAttachment("Page Source", "text/html", pageSource, ".html");
        logger.info("Page source attached to Allure report");
    }
}
```

### Вложение JSON

```java
import io.qameta.allure.Allure;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import com.fasterxml.jackson.databind.ObjectMapper;

public class JsonAttachmentUtil {
    private static final Logger logger = LoggerFactory.getLogger(JsonAttachmentUtil.class);

    public static void attachJson(Object data) {
        try {
            ObjectMapper mapper = new ObjectMapper();
            String json = mapper.writeValueAsString(data);
            Allure.addAttachment("JSON Data", "application/json", json, ".json");
            logger.info("JSON data attached to Allure report");
        } catch (Exception e) {
            logger.error("Failed to attach JSON: {}", e.getMessage());
        }
    }
}
```

### Вложение логов

Логи можно прикрепить, считав их из файла (например, `logs/test.log`).

```java
import io.qameta.allure.Allure;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.nio.file.Files;
import java.nio.file.Paths;

public class LogAttachmentUtil {
    private static final Logger logger = LoggerFactory.getLogger(LogAttachmentUtil.class);

    public static void attachLogFile(String logFilePath) {
        try {
            String logContent = new String(Files.readAllBytes(Paths.get(logFilePath)));
            Allure.addAttachment("Test Log", "text/plain", logContent, ".log");
            logger.info("Log file attached to Allure report");
        } catch (Exception e) {
            logger.error("Failed to attach log file: {}", e.getMessage());
        }
    }
}
```

### Вложение curl-запросов (RestAssured)

Для API-тестирования с RestAssured можно автоматически прикреплять запросы и ответы.

#### Зависимость для RestAssured

```xml
<dependencies>
    <dependency>
        <groupId>io.rest-assured</groupId>
        <artifactId>rest-assured</artifactId>
        <version>5.5.0</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

#### Пример прикрепления curl-запроса

```java
import io.qameta.allure.rest.assured.AllureRestAssured;
import io.restassured.RestAssured;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class ApiTestUtil {
    private static final Logger logger = LoggerFactory.getLogger(ApiTestUtil.class);

    public static void testApiRequest() {
        RestAssured.given()
                .filter(new AllureRestAssured())
                .baseUri("https://api.example.com")
                .when()
                .get("/endpoint")
                .then()
                .statusCode(200);
        logger.info("API request completed and logged to Allure");
    }
}
```

### Особенности
- **HTML**: Прикрепление исходного кода страницы помогает анализировать DOM при ошибках.
- **JSON**: Полезно для вложения данных запросов/ответов API или конфигураций.
- **Логи**: Прикрепление логов из `logback.xml` упрощает отладку.
- **Curl-запросы**: `AllureRestAssured` автоматически добавляет запросы и ответы в отчёт.

## Интеграция с Jenkins

Для генерации Allure-отчётов в Jenkins настройте задачу Maven и плагин Allure.

### Пример Jenkins Pipeline

```groovy
pipeline {
    agent {
        docker {
            image 'maven:3.9.9-eclipse-temurin-17'
            args '-v $HOME/.m2:/root/.m2'
        }
    }
    stages {
        stage('Test') {
            steps {
                sh 'mvn clean test'
            }
        }
    }
    post {
        always {
            allure results: [[path: 'target/allure-results']]
        }
    }
}
```

## Отладка и устойчивость

1. **Отладка в IntelliJ IDEA**:
   - Установите точки останова в методах, где добавляются шаги или вложения.
   - Проверяйте содержимое вложений (например, JSON или HTML) в дебаггере.
   - Используйте логи SLF4J для отслеживания добавления шагов и вложений.

2. **Устойчивость тестов**:
   - Используйте `WebDriverWait` для ожидания элементов перед скриншотами.
   - Проверяйте доступность файлов логов перед их прикреплением.
   - Пример:

```java
public static void safeAttachLogFile(String logFilePath) {
    try {
        if (Files.exists(Paths.get(logFilePath))) {
            attachLogFile(logFilePath);
        } else {
            logger.warn("Log file not found: {}", logFilePath);
        }
    } catch (Exception e) {
        logger.error("Failed to attach log file: {}", e.getMessage());
    }
}
```

## Распространённые ошибки

1. **Отсутствие шагов**: Неправильное использование `@Step` или `Allure.step()` приводит к пустым отчётам.
2. **Скриншоты не прикрепляются**: Проблемы с `TakesScreenshot` из-за некорректной инициализации WebDriver.
3. **Большие вложения**: Прикрепление больших файлов (логи, HTML) может замедлить генерацию отчёта.
4. **Чувствительные данные**: Случайное включение паролей или токенов в вложения.

## Дополнительно

- **Selenide для упрощения**:
  - Selenide автоматически интегрируется с Allure:

```java
import com.codeborne.selenide.Configuration;
import com.codeborne.selenide.Selenide;
import com.codeborne.selenide.logevents.SelenideLogger;
import io.qameta.allure.selenide.AllureSelenide;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import static com.codeborne.selenide.Selenide.*;

public class SelenideAllureExample {
    private static final Logger logger = LoggerFactory.getLogger(SelenideAllureExample.class);

    public static void setupAllure() {
        SelenideLogger.addListener("AllureSelenide", new AllureSelenide()
                .screenshots(true)
                .savePageSource(true));
        Configuration.browser = "chrome";
        Configuration.timeout = 10000;
        logger.info("Selenide configured with Allure listener");
    }

    public static void testLogin() {
        setupAllure();
        open("https://example.com/login");
        $("#username").setValue("testuser");
        $("#password").setValue("testpass");
        $("#login-button").click();
        logger.info("Login test completed");
    }
}
```

- **Testcontainers для изолированного окружения**:

```xml
<dependencies>
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>selenium</artifactId>
        <version>1.20.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

```java
import io.qameta.allure.Allure;
import org.openqa.selenium.WebDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testcontainers.containers.BrowserWebDriverContainer;

public class TestcontainersAllureExample {
    private static final Logger logger = LoggerFactory.getLogger(TestcontainersAllureExample.class);

    public void testWithAllure() {
        try (BrowserWebDriverContainer<?> container = new BrowserWebDriverContainer<>()
                .withCapabilities(new ChromeOptions())) {
            container.start();
            WebDriver driver = container.getWebDriver();
            Allure.step("Navigating to login page", () -> {
                driver.get("https://example.com/login");
                logger.info("Navigated to login page in Testcontainers");
            });
            byte[] screenshot = ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
            Allure.addAttachment("Testcontainers Screenshot", "image/png", screenshot, ".png");
            driver.quit();
        }
    }
}
```

## Источники
- [Allure Framework](https://allurereport.org/)
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/)
- [Selenide Documentation](https://selenide.org/)
- [Testcontainers Documentation](https://www.testcontainers.org/)

---
[**← Назад к оглавлению**](../README.md)