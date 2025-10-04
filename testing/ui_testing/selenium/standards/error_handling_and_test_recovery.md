# Обработка ошибок и падений тестов

Обработка ошибок и падений тестов в автоматизации тестирования критически важна для обеспечения стабильности, воспроизводимости и удобства отладки. В этой статье описаны подходы к созданию скриншотов и логов при исключениях, использованию блоков `try-catch` для управления нестабильными действиями, а также автоматическому удалению данных или восстановлению состояния системы после теста. Все примеры реализованы на Java с использованием Selenium WebDriver, Page Object Model, JUnit 5, TestNG, Allure и других инструментов из заданного стека.

## Скриншоты и логи при исключениях

Автоматическое создание скриншотов и сохранение логов при падении тестов упрощает диагностику ошибок. Allure используется для прикрепления этих данных к отчётам.

### Настройка зависимостей

Добавьте зависимости в `pom.xml` для Allure, SLF4J и Logback:

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
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>2.0.16</version>
    </dependency>
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.5.8</version>
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
    </plugins>
</build>
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

### Слушатель для скриншотов и логов (JUnit 5)

Создайте слушатель для автоматического создания скриншотов и прикрепления логов при падении теста.

```java
import io.qameta.allure.Allure;
import org.junit.jupiter.api.extension.AfterTestExecutionCallback;
import org.junit.jupiter.api.extension.ExtensionContext;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.nio.file.Files;
import java.nio.file.Paths;

public class FailureListener implements AfterTestExecutionCallback {
    private static final Logger logger = LoggerFactory.getLogger(FailureListener.class);
    private final WebDriver driver;
    private final String logFilePath = "logs/test.log";

    public FailureListener(WebDriver driver) {
        this.driver = driver;
    }

    @Override
    public void afterTestExecution(ExtensionContext context) {
        if (context.getExecutionException().isPresent()) {
            // Attach screenshot
            if (driver instanceof TakesScreenshot) {
                byte[] screenshot = ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
                Allure.addAttachment("Screenshot on failure", "image/png", screenshot, ".png");
                logger.info("Screenshot captured and attached to Allure");
            }

            // Attach log file
            try {
                String logContent = new String(Files.readAllBytes(Paths.get(logFilePath)));
                Allure.addAttachment("Test Log", "text/plain", logContent, ".log");
                logger.info("Log file attached to Allure");
            } catch (Exception e) {
                logger.error("Failed to attach log file: {}", e.getMessage());
            }
        }
    }
}
```

### Слушатель для TestNG

```java
import io.qameta.allure.Allure;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testng.ITestListener;
import org.testng.ITestResult;

import java.nio.file.Files;
import java.nio.file.Paths;

public class TestNGFailureListener implements ITestListener {
    private static final Logger logger = LoggerFactory.getLogger(TestNGFailureListener.class);
    private final WebDriver driver;
    private final String logFilePath = "logs/test.log";

    public TestNGFailureListener(WebDriver driver) {
        this.driver = driver;
    }

    @Override
    public void onTestFailure(ITestResult result) {
        // Attach screenshot
        if (driver instanceof TakesScreenshot) {
            byte[] screenshot = ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
            Allure.addAttachment("Screenshot on failure", "image/png", screenshot, ".png");
            logger.info("Screenshot captured and attached to Allure");
        }

        // Attach log file
        try {
            String logContent = new String(Files.readAllBytes(Paths.get(logFilePath)));
            Allure.addAttachment("Test Log", "text/plain", logContent, ".log");
            logger.info("Log file attached to Allure");
        } catch (Exception e) {
            logger.error("Failed to attach log file: {}", e.getMessage());
        }
    }
}
```

## Использование try-catch для нестабильных действий

Нестабильные действия, такие как взаимодействие с элементами UI или API-запросы, должны быть обернуты в блоки `try-catch` для обработки исключений и предотвращения падения тестов.

### Пример Page Object с обработкой ошибок

```java
import io.qameta.allure.Step;
import org.openqa.selenium.NoSuchElementException;
import org.openqa.selenium.TimeoutException;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.openqa.selenium.support.ui.ExpectedConditions;
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

    @FindBy(id = "login-button")
    private WebElement loginButton;

    public LoginPage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        PageFactory.initElements(driver, this);
    }

    @Step("Logging in as user: {username}")
    public DashboardPage loginAs(String username, String password) {
        try {
            logger.info("Attempting login for user: {}", username);
            wait.until(ExpectedConditions.visibilityOf(usernameField));
            usernameField.sendKeys(username);
            passwordField.sendKeys(password);
            wait.until(ExpectedConditions.elementToBeClickable(loginButton));
            loginButton.click();
            logger.info("Login attempt completed");
            return new DashboardPage(driver);
        } catch (TimeoutException | NoSuchElementException e) {
            logger.error("Login failed: {}", e.getMessage());
            throw new IllegalStateException("Failed to login", e);
        }
    }
}
```

### Пример API-теста с обработкой ошибок

```java
import io.qameta.allure.Step;
import io.restassured.RestAssured;
import io.restassured.response.Response;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class ApiUtil {
    private static final Logger logger = LoggerFactory.getLogger(ApiUtil.class);

    @Step("Sending GET request to {endpoint}")
    public Response sendGetRequest(String endpoint) {
        try {
            logger.info("Sending GET request to: {}", endpoint);
            Response response = RestAssured.get(endpoint);
            logger.info("Received response with status: {}", response.getStatusCode());
            return response;
        } catch (Exception e) {
            logger.error("API request failed: {}", e.getMessage());
            throw new RuntimeException("API request failed", e);
        }
    }
}
```

### Особенности
- **Обработка исключений**: Используйте конкретные исключения (`TimeoutException`, `NoSuchElementException`) для точной диагностики.
- **Логирование**: Логируйте ошибки с помощью SLF4J для отслеживания причин.
- **Allure**: Аннотации `@Step` документируют действия в отчётах.

## Автоматическое удаление данных или восстановление состояния

После выполнения тестов важно очищать созданные данные или восстанавливать исходное состояние системы, чтобы избежать влияния на последующие тесты.

### Очистка данных через API

```java
import io.qameta.allure.Step;
import io.restassured.RestAssured;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class CleanupUtil {
    private static final Logger logger = LoggerFactory.getLogger(CleanupUtil.class);

    @Step("Cleaning up test data for user: {username}")
    public void deleteUser(String username) {
        try {
            logger.info("Deleting user: {}", username);
            RestAssured.given()
                    .baseUri("https://api.example.com")
                    .delete("/users/" + username)
                    .then()
                    .statusCode(204);
            logger.info("User deleted successfully");
        } catch (Exception e) {
            logger.warn("Failed to delete user: {}", e.getMessage());
        }
    }
}
```

### Восстановление состояния UI

```java
import io.qameta.allure.Step;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class SettingsPage {
    private static final Logger logger = LoggerFactory.getLogger(SettingsPage.class);
    private final WebDriver driver;

    @FindBy(id = "reset-settings")
    private WebElement resetButton;

    public SettingsPage(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this);
    }

    @Step("Resetting settings to default")
    public void resetToDefault() {
        try {
            logger.info("Resetting settings to default");
            resetButton.click();
            logger.info("Settings reset successfully");
        } catch (Exception e) {
            logger.warn("Failed to reset settings: {}", e.getMessage());
        }
    }
}
```

### Пример теста с очисткой (JUnit 5)

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import io.qameta.allure.Step;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.RegisterExtension;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import static org.assertj.core.api.Assertions.assertThat;

public class ErrorHandlingTest {
    private static final Logger logger = LoggerFactory.getLogger(ErrorHandlingTest.class);
    private WebDriver driver;
    private LoginPage loginPage;
    private CleanupUtil cleanupUtil;

    @RegisterExtension
    FailureListener failureListener;

    @BeforeEach
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        failureListener = new FailureListener(driver);
        loginPage = new LoginPage(driver);
        cleanupUtil = new CleanupUtil();
        logger.info("WebDriver initialized");
    }

    @Test
    @Step("Test login with invalid credentials")
    void testInvalidLogin() {
        // Arrange
        String username = "invaliduser";
        loginPage.openPage("https://example.com/login");

        // Act
        try {
            loginPage.loginAs(username, "wrongpass");
        } catch (IllegalStateException e) {
            logger.info("Expected login failure: {}", e.getMessage());
        }

        // Assert
        assertThat(driver.getCurrentUrl()).doesNotContain("dashboard");
        logger.info("Login failure verified");
    }

    @AfterEach
    void tearDown() {
        // Cleanup
        cleanupUtil.deleteUser("invaliduser");
        logger.info("Cleanup completed");
        if (driver != null) {
            driver.quit();
            logger.info("WebDriver closed");
        }
    }
}
```

### Пример теста с очисткой (TestNG)

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

@Listeners(TestNGFailureListener.class)
public class ErrorHandlingTestNG {
    private static final Logger logger = LoggerFactory.getLogger(ErrorHandlingTestNG.class);
    private WebDriver driver;
    private LoginPage loginPage;
    private CleanupUtil cleanupUtil;

    @BeforeMethod
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        loginPage = new LoginPage(driver);
        cleanupUtil = new CleanupUtil();
        logger.info("WebDriver initialized");
    }

    @Test
    void testInvalidLogin() {
        // Arrange
        String username = "invaliduser";
        loginPage.openPage("https://example.com/login");

        // Act
        try {
            loginPage.loginAs(username, "wrongpass");
        } catch (IllegalStateException e) {
            logger.info("Expected login failure: {}", e.getMessage());
        }

        // Assert
        assertThat(driver.getCurrentUrl()).doesNotContain("dashboard");
        logger.info("Login failure verified");
    }

    @AfterMethod
    void tearDown() {
        // Cleanup
        cleanupUtil.deleteUser("invaliduser");
        logger.info("Cleanup completed");
        if (driver != null) {
            driver.quit();
            logger.info("WebDriver closed");
        }
    }
}
```

## Интеграция с Jenkins

Настройте Jenkins для генерации Allure-отчётов с прикреплёнными скриншотами и логами.

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
   - Установите точки останова в блоках `try-catch` или методах очистки.
   - Проверяйте содержимое скриншотов и логов в дебаггере.
   - Используйте DevTools браузера для проверки локаторов при падении тестов.

2. **Устойчивость тестов**:
   - Используйте `WebDriverWait` для ожидания элементов перед действиями.
   - Проверяйте наличие данных перед очисткой, чтобы избежать ошибок.
   - Пример:

```java
public void safeDeleteUser(String username) {
    try {
        deleteUser(username);
    } catch (Exception e) {
        logger.warn("No user data to delete for: {}", username);
    }
}
```

## Распространённые ошибки

1. **Неполная очистка**: Пропуск очистки данных приводит к влиянию на последующие тесты.
2. **Игнорирование исключений**: Пустые блоки `catch` скрывают реальные проблемы.
3. **Большие логи**: Чрезмерное логирование замедляет тесты и отчёты.
4. **Чувствительные данные в логах**: Случайное включение паролей или токенов в скриншоты или логи.

## Дополнительно

- **Selenide для упрощения**:
  - Selenide автоматически интегрируется с Allure и поддерживает скриншоты:

```java
import com.codeborne.selenide.Configuration;
import com.codeborne.selenide.Selenide;
import com.codeborne.selenide.logevents.SelenideLogger;
import io.qameta.allure.selenide.AllureSelenide;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import static com.codeborne.selenide.Selenide.*;

public class SelenideErrorHandling {
    private static final Logger logger = LoggerFactory.getLogger(SelenideErrorHandling.class);

    public static void setupSelenide() {
        SelenideLogger.addListener("AllureSelenide", new AllureSelenide()
                .screenshots(true)
                .savePageSource(true));
        Configuration.browser = "chrome";
        Configuration.timeout = 10000;
        logger.info("Selenide configured with Allure listener");
    }

    public static void testLoginWithErrorHandling() {
        setupSelenide();
        try {
            open("https://example.com/login");
            $("#username").setValue("invaliduser");
            $("#password").setValue("wrongpass");
            $("#login-button").click();
        } catch (Exception e) {
            logger.error("Login failed: {}", e.getMessage());
            throw e; // Пробрасываем для Allure
        } finally {
            // Cleanup
            logger.info("Cleaning up test data");
            // Логика очистки
        }
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

public class TestcontainersErrorHandling {
    private static final Logger logger = LoggerFactory.getLogger(TestcontainersErrorHandling.class);

    public void testWithErrorHandling() {
        try (BrowserWebDriverContainer<?> container = new BrowserWebDriverContainer<>()
                .withCapabilities(new ChromeOptions())) {
            container.start();
            WebDriver driver = container.getWebDriver();
            Allure.step("Navigating to login page", () -> {
                try {
                    driver.get("https://example.com/login");
                    logger.info("Navigated to login page");
                } catch (Exception e) {
                    logger.error("Navigation failed: {}", e.getMessage());
                    throw e;
                }
            });
            driver.quit();
        } catch (Exception e) {
            logger.error("Test failed: {}", e.getMessage());
            throw e;
        } finally {
            logger.info("Cleaning up test data");
            // Логика очистки
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