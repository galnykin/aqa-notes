# Логирование ошибок

Логирование ошибок в автоматизированных тестах помогает выявлять и устранять проблемы, обеспечивая прозрачность выполнения тестов. В статье рассматривается, что именно нужно логировать при исключениях (тип, сообщение, стек), когда использовать уровни логов `WARN` и `ERROR`, а также как обрабатывать повторные ошибки. Используется технологический стек с Java 17, SLF4J с Logback, JUnit 5, TestNG, Selenium WebDriver, RestAssured и Allure для отчётности.

## Настройка логирования с SLF4J и Logback

Для логирования ошибок используется SLF4J как фасад и Logback как реализация. Настройка аналогична описанной в предыдущих статьях, но здесь акцент на обработку исключений.

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
</dependencies>
```

### Конфигурация Logback

Файл `logback.xml` в `src/main/resources` для вывода логов в консоль и файл:

```xml
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
        <file>logs/test_errors.log</file>
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

## Что логировать при исключениях

При возникновении исключений необходимо логировать:

- **Тип исключения** (`NullPointerException`, `NoSuchElementException` и т.д.).
- **Сообщение исключения** (описание ошибки, предоставленное JVM или библиотекой).
- **Стек вызовов** (stack trace), чтобы понять, где именно произошла ошибка.

### Пример логирования исключений

Создайте утилитный класс для логирования:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class ErrorLogger {
    private static final Logger logger = LoggerFactory.getLogger(ErrorLogger.class);

    public static void logException(String operation, Exception e) {
        logger.error("Ошибка в операции '{}': Тип: {}, Сообщение: {}", 
            operation, e.getClass().getSimpleName(), e.getMessage(), e);
    }

    public static void logWarning(String operation, String message) {
        logger.warn("Предупреждение в операции '{}': {}", operation, message);
    }
}
```

Пример использования в UI-тесте с Selenium WebDriver:

```java
import org.openqa.selenium.NoSuchElementException;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.openqa.selenium.support.ui.ExpectedConditions;

public class LoginPage {
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
        String operation = "Вход в систему";
        try {
            ErrorLogger.logWarning(operation, "Начало ввода данных");
            wait.until(ExpectedConditions.visibilityOf(usernameField)).sendKeys(username);
            wait.until(ExpectedConditions.elementToBeClickable(loginButton)).click();
        } catch (NoSuchElementException e) {
            ErrorLogger.logException(operation, e);
            throw e;
        }
    }
}
```

Лог будет выглядеть так:

```
2025-10-03 20:21:00 WARN  LoginPage - Предупреждение в операции 'Вход в систему': Начало ввода данных
2025-10-03 20:21:01 ERROR LoginPage - Ошибка в операции 'Вход в систему': Тип: NoSuchElementException, Сообщение: no such element: Unable to locate element: {"method":"css selector","selector":"#username"}
  (полный stack trace следует далее)
```

## Когда использовать `WARN` vs `ERROR`

- **WARN**: Используйте для ситуаций, которые не приводят к сбою теста, но требуют внимания. Примеры:
  - Элемент временно недоступен, но тест продолжает выполнение с повторной попыткой.
  - Предупреждение о некритичной проблеме, например, медленная загрузка страницы.
  - Пропуск необязательной проверки (например, баннер не отобразился, но тест продолжается).

- **ERROR**: Используйте для критических ошибок, которые приводят к сбою теста. Примеры:
  - Исключения, такие как `NoSuchElementException` или `TimeoutException`.
  - Ошибки API-запросов (например, статус 500).
  - Провал утверждений (`assertThat` в JUnit 5 или `assertTrue` в TestNG).

Пример теста с JUnit 5:

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.SoftAssertions.assertSoftly;

public class LoginTest {
    private WebDriver driver;
    private LoginPage loginPage;

    @BeforeEach
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        loginPage = new LoginPage(driver);
    }

    @Test
    void testLoginWithError() {
        try {
            ErrorLogger.logWarning("Тест логина", "Попытка входа с неверными данными");
            loginPage.login("invalid_user");
            assertThat(driver.getCurrentUrl()).contains("dashboard");
        } catch (Exception e) {
            ErrorLogger.logException("Тест логина", e);
            assertSoftly(softly -> softly.assertThat(false).isTrue());
        }
    }

    @AfterEach
    void tearDown() {
        if (driver != null) {
            driver.quit();
        }
    }
}
```

Пример теста с TestNG:

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;

public class LoginTestNG {
    private WebDriver driver;
    private LoginPage loginPage;

    @BeforeMethod
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        loginPage = new LoginPage(driver);
    }

    @Test
    void testLoginWithError() {
        try {
            ErrorLogger.logWarning("Тест логина", "Попытка входа с неверными данными");
            loginPage.login("invalid_user");
            assertTrue(driver.getCurrentUrl().contains("dashboard"));
        } catch (Exception e) {
            ErrorLogger.logException("Тест логина", e);
            throw e;
        }
    }

    @AfterMethod
    void tearDown() {
        if (driver != null) {
            driver.quit();
        }
    }
}
```

## Поведение при повторных ошибках

Повторные ошибки (flaky tests) часто возникают из-за нестабильных условий, таких как медленная загрузка страницы или временная недоступность API. Для их обработки:

- **Повторные попытки (Retry)**: Используйте механизмы повторных попыток для UI- и API-тестов.
- **Ограничение логов**: Избегайте дублирования логов для повторяющихся ошибок, чтобы не перегружать отчёты.
- **Агрегация ошибок**: Логируйте только первую ошибку или агрегируйте повторные ошибки в одном сообщении.

### Реализация механизма повторных попыток

Для JUnit 5 можно использовать библиотеку `junit5-retry`:

```xml
<dependency>
    <groupId>com.github.junit5-retry</groupId>
    <artifactId>junit5-retry</artifactId>
    <version>1.0.0</version>
</dependency>
```

Пример теста с повторными попытками:

```java
import com.github.junit5.retry.Retry;
import org.junit.jupiter.api.Test;

public class RetryTest {
    @Test
    @Retry(maxRetries = 3, delay = 1000)
    void testWithRetry() {
        try {
            ErrorLogger.logWarning("Тест с повторными попытками", "Попытка выполнения");
            // Логика теста
            throw new RuntimeException("Сбой теста");
        } catch (Exception e) {
            ErrorLogger.logException("Тест с повторными попытками", e);
            throw e;
        }
    }
}
```

Для TestNG используйте встроенный механизм `IRetryAnalyzer`:

```java
import org.testng.IRetryAnalyzer;
import org.testng.ITestResult;

public class RetryAnalyzer implements IRetryAnalyzer {
    private int retryCount = 0;
    private static final int MAX_RETRY_COUNT = 3;

    @Override
    public boolean retry(ITestResult result) {
        if (retryCount < MAX_RETRY_COUNT) {
            retryCount++;
            ErrorLogger.logWarning("Повторная попытка", "Попытка #" + retryCount + " для теста " + result.getName());
            return true;
        }
        return false;
    }
}
```

Применение в тесте:

```java
import org.testng.annotations.Test;

public class RetryTestNG {
    @Test(retryAnalyzer = RetryAnalyzer.class)
    void testWithRetry() {
        try {
            ErrorLogger.logWarning("Тест с повторными попытками", "Попытка выполнения");
            // Логика теста
            throw new RuntimeException("Сбой теста");
        } catch (Exception e) {
            ErrorLogger.logException("Тест с повторными попытками", e);
            throw e;
        }
    }
}
```

### Ограничение дублирования логов

Для предотвращения дублирования логов при повторных ошибках используйте условную логику:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class ErrorLogger {
    private static final Logger logger = LoggerFactory.getLogger(ErrorLogger.class);
    private static final Set<String> loggedErrors = new HashSet<>();

    public static void logUniqueException(String operation, Exception e) {
        String errorKey = operation + e.getClass().getSimpleName() + e.getMessage();
        if (loggedErrors.add(errorKey)) {
            logger.error("Ошибка в операции '{}': Тип: {}, Сообщение: {}", 
                operation, e.getClass().getSimpleName(), e.getMessage(), e);
        } else {
            logger.warn("Повторная ошибка в операции '{}': {}", operation, e.getMessage());
        }
    }
}
```

## Интеграция с Allure для отчётности

Allure позволяет добавлять информацию об ошибках в отчёты. Пример:

```java
import io.qameta.allure.Allure;
import org.junit.jupiter.api.Test;

public class AllureErrorTest {
    @Test
    void testWithError() {
        try {
            Allure.step("Шаг 1: Выполнение действия", () -> {
                throw new RuntimeException("Тестовая ошибка");
            });
        } catch (Exception e) {
            ErrorLogger.logException("Тест с Allure", e);
            Allure.step("Шаг 2: Обработка ошибки", () -> {
                Allure.addAttachment("Stack Trace", e.getMessage());
            });
            throw e;
        }
    }
}
```

Добавьте зависимость Allure в `pom.xml`:

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

### Стабильность тестов

- **Явные ожидания**: Используйте `WebDriverWait` для UI-тестов и таймауты в RestAssured для API-тестов, чтобы избежать преждевременных ошибок.
- **Стабильные локаторы**: Используйте `id` или `data-*` атрибуты, чтобы минимизировать `NoSuchElementException`.
- **Проверки перед действиями**: Проверяйте `isDisplayed()` и `isEnabled()` перед взаимодействием с элементами.

### Отладка

- В IntelliJ IDEA добавьте точки останова в блоках `catch` для анализа исключений.
- Используйте логи для проверки значений переменных и локаторов перед возникновением ошибки.

### Распространённые ошибки

- **Неполные логи**: Не забывайте логировать полный стек вызовов для точной диагностики.
- **Перегрузка логов**: Избегайте избыточного логирования `WARN` для некритичных событий.
- **Игнорирование повторных ошибок**: Без механизма `Retry` тесты могут быть нестабильными.

## Дополнительно

### Интеграция с Jenkins

Настройте Jenkins для запуска тестов и анализа логов ошибок:

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

### Использование Testcontainers

Testcontainers полезен для тестирования API с поднятием сервисов в контейнерах. Пример логирования ошибок:

```java
import org.junit.jupiter.api.Test;
import org.testcontainers.containers.GenericContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

@Testcontainers
public class ContainerErrorTest {
    @Container
    private static final GenericContainer<?> container = new GenericContainer<>("httpd:2.4")
            .withExposedPorts(80);

    @Test
    void testContainerWithError() {
        try {
            String address = "http://" + container.getHost() + ":" + container.getFirstMappedPort();
            // Выполнить запрос
            throw new RuntimeException("Ошибка в контейнере");
        } catch (Exception e) {
            ErrorLogger.logException("Тест контейнера", e);
            throw e;
        }
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