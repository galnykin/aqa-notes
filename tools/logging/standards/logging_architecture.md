# Логирование в архитектуре тестирования

Логирование в автоматизации тестирования должно быть организовано по слоям архитектуры (UI, API, сервисы, хранилище) для упрощения отладки и анализа. В статье рассматривается организация логов по слоям, разделение логов по модулям и файлам, а также использование MDC (Mapped Diagnostic Context) для добавления контекста, такого как `requestId` или `userId`. Используется технологический стек: Java 17, Maven, SLF4J с Logback, JUnit 5, TestNG, Selenium WebDriver, RestAssured и Allure.

## Логирование по слоям архитектуры

Архитектура тестов обычно включает несколько слоёв: UI (Selenium WebDriver), API (RestAssured), сервисы (бизнес-логика) и хранилище (база данных, файлы). Каждый слой требует специфического подхода к логированию.

### UI-слой (Selenium WebDriver)

На UI-слое логируются действия пользователя (клики, ввод данных, переходы) и состояние элементов. Используйте Page Object Model для структурирования кода.

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
        logger.info("UI: Открытие страницы логина");
        driver.get("https://example.com/login");

        logger.info("UI: Ввод имени пользователя: {}", username);
        wait.until(ExpectedConditions.visibilityOf(usernameField)).sendKeys(username);

        logger.info("UI: Клик по кнопке входа");
        wait.until(ExpectedConditions.elementToBeClickable(loginButton)).click();
    }
}
```

### API-слой (RestAssured)

На API-слое логируются запросы, ответы и их параметры.

```java
import io.restassured.RestAssured;
import io.restassured.response.Response;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class ApiClient {
    private static final Logger logger = LoggerFactory.getLogger(ApiClient.class);

    public Response getUser(int userId) {
        logger.info("API: Отправка GET-запроса для пользователя: {}", userId);
        Response response = RestAssured.given()
                .baseUri("https://api.example.com")
                .when()
                .get("/users/" + userId)
                .then()
                .extract().response();

        logger.info("API: Получен ответ, статус: {}", response.getStatusCode());
        return response;
    }
}
```

### Сервисный слой

Сервисный слой обрабатывает бизнес-логику, например, валидацию данных. Логируйте входные параметры и результаты.

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class UserService {
    private static final Logger logger = LoggerFactory.getLogger(UserService.class);

    public boolean validateUser(String username) {
        logger.info("Service: Валидация пользователя: {}", username);
        boolean isValid = username != null && !username.isEmpty();
        logger.info("Service: Результат валидации: {}", isValid);
        return isValid;
    }
}
```

### Слой хранилища

На уровне хранилища (например, база данных) логируйте запросы и результаты. Используйте Testcontainers для эмуляции базы данных.

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

public class DatabaseClient {
    private static final Logger logger = LoggerFactory.getLogger(DatabaseClient.class);

    public String getUserFromDb(Connection connection, int userId) {
        logger.info("DB: Выполнение запроса для пользователя: {}", userId);
        try (PreparedStatement stmt = connection.prepareStatement("SELECT username FROM users WHERE id = ?")) {
            stmt.setInt(1, userId);
            ResultSet rs = stmt.executeQuery();
            if (rs.next()) {
                String username = rs.getString("username");
                logger.info("DB: Пользователь найден: {}", username);
                return username;
            }
            logger.warn("DB: Пользователь с id {} не найден", userId);
            return null;
        } catch (Exception e) {
            logger.error("DB: Ошибка запроса для пользователя {}: {}", userId, e.getMessage(), e);
            throw new RuntimeException(e);
        }
    }
}
```

## Разделение логов по модулям и файлам

Для разделения логов по модулям создайте отдельные логгеры и аппендеры для каждого слоя.

### Конфигурация Logback для модулей

Обновите `logback.xml` для разделения логов по файлам:

```xml
<configuration>
    <appender name="UI_FILE" class="ch.qos.logback.core.FileAppender">
        <file>logs/ui.log</file>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <appender name="API_FILE" class="ch.qos.logback.core.FileAppender">
        <file>logs/api.log</file>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <appender name="SERVICE_FILE" class="ch.qos.logback.core.FileAppender">
        <file>logs/service.log</file>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <appender name="DB_FILE" class="ch.qos.logback.core.FileAppender">
        <file>logs/db.log</file>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <logger name="LoginPage" level="info" additivity="false">
        <appender-ref ref="UI_FILE"/>
    </logger>
    <logger name="ApiClient" level="info" additivity="false">
        <appender-ref ref="API_FILE"/>
    </logger>
    <logger name="UserService" level="info" additivity="false">
        <appender-ref ref="SERVICE_FILE"/>
    </logger>
    <logger name="DatabaseClient" level="info" additivity="false">
        <appender-ref ref="DB_FILE"/>
    </logger>

    <root level="info">
        <appender-ref ref="CONSOLE"/>
    </root>
</configuration>
```

Каждый слой пишет логи в свой файл (`ui.log`, `api.log`, `service.log`, `db.log`), что упрощает анализ.

## Поддержка MDC (Mapped Diagnostic Context)

MDC позволяет добавлять контекстные данные, такие как `requestId` или `userId`, к логам для отслеживания операций.

### Настройка MDC

Пример использования MDC в тесте:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.slf4j.MDC;
import java.util.UUID;

public class MdcLogger {
    private static final Logger logger = LoggerFactory.getLogger(MdcLogger.class);

    public static void logWithContext(String operation, String userId) {
        MDC.put("requestId", UUID.randomUUID().toString());
        MDC.put("userId", userId);
        logger.info("Операция: {}", operation);
        MDC.clear();
    }
}
```

Обновите `logback.xml` для включения MDC в шаблон логов:

```xml
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} [requestId=%X{requestId}, userId=%X{userId}] - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="info">
        <appender-ref ref="CONSOLE"/>
    </root>
</configuration>
```

### Пример теста с MDC (JUnit 5)

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.MDC;
import java.util.UUID;

import static org.assertj.core.api.Assertions.assertThat;

public class LoginTest {
    private WebDriver driver;
    private LoginPage loginPage;

    @BeforeEach
    void setUp() {
        MDC.put("requestId", UUID.randomUUID().toString());
        MDC.put("userId", "testUser");
        MdcLogger.logWithContext("Инициализация теста логина", "testUser");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        loginPage = new LoginPage(driver);
        MDC.clear();
    }

    @Test
    void testLogin() {
        MDC.put("requestId", UUID.randomUUID().toString());
        MDC.put("userId", "testUser");
        MdcLogger.logWithContext("Запуск теста логина", "testUser");
        loginPage.login("user");
        assertThat(driver.getCurrentUrl()).contains("dashboard");
        MDC.clear();
    }

    @AfterEach
    void tearDown() {
        MDC.put("requestId", UUID.randomUUID().toString());
        MDC.put("userId", "testUser");
        MdcLogger.logWithContext("Завершение теста", "testUser");
        if (driver != null) {
            driver.quit();
        }
        MDC.clear();
    }
}
```

Лог будет выглядеть так:

```
2025-10-03 20:28:00 INFO  MdcLogger [requestId=123e4567-e89b-12d3-a456-426614174000, userId=testUser] - Операция: Запуск теста логина
```

### Пример теста с MDC (TestNG)

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.MDC;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import java.util.UUID;

import static org.testng.Assert.assertTrue;

public class LoginTestNG {
    private WebDriver driver;
    private LoginPage loginPage;

    @BeforeMethod
    void setUp() {
        MDC.put("requestId", UUID.randomUUID().toString());
        MDC.put("userId", "testUser");
        MdcLogger.logWithContext("Инициализация теста логина", "testUser");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        loginPage = new LoginPage(driver);
        MDC.clear();
    }

    @Test
    void testLogin() {
        MDC.put("requestId", UUID.randomUUID().toString());
        MDC.put("userId", "testUser");
        MdcLogger.logWithContext("Запуск теста логина", "testUser");
        loginPage.login("user");
        assertTrue(driver.getCurrentUrl().contains("dashboard"));
        MDC.clear();
    }

    @AfterMethod
    void tearDown() {
        MDC.put("requestId", UUID.randomUUID().toString());
        MDC.put("userId", "testUser");
        MdcLogger.logWithContext("Завершение теста", "testUser");
        if (driver != null) {
            driver.quit();
        }
        MDC.clear();
    }
}
```

## Интеграция с Allure

Allure поддерживает отображение контекста MDC в отчётах. Пример:

```java
import io.qameta.allure.Allure;
import io.qameta.allure.Step;
import org.junit.jupiter.api.Test;
import org.slf4j.MDC;
import java.util.UUID;

public class AllureMdcTest {
    @Test
    void testWithMdc() {
        MDC.put("requestId", UUID.randomUUID().toString());
        MDC.put("userId", "testUser");
        loginStep();
        MDC.clear();
    }

    @Step("Шаг: Вход в систему, requestId={0}, userId={1}")
    private void loginStep() {
        MdcLogger.logWithContext("Вход в систему", MDC.get("userId"));
        Allure.addAttachment("Контекст", "requestId: " + MDC.get("requestId") + ", userId: " + MDC.get("userId"));
    }
}
```

## Лучшие практики и отладка

### Логирование по слоям

- Используйте префиксы в сообщениях логов (например, "UI:", "API:") для идентификации слоя.
- Логируйте только ключевые действия и результаты, чтобы избежать перегрузки логов.

### Разделение логов

- Разделяйте логи по модулям, используя разные файлы или логгеры.
- Настройте уровень логирования (`INFO`, `DEBUG`) для каждого модуля в `logback.xml`.

### MDC

- Всегда очищайте MDC (`MDC.clear()`) после завершения операции, чтобы избежать утечек контекста.
- Используйте уникальные идентификаторы (`UUID`) для `requestId`.

### Отладка

- В IntelliJ IDEA используйте точки останова в классах слоёв для проверки логов.
- Анализируйте файлы логов (`ui.log`, `api.log`) для диагностики ошибок.

### Распространённые ошибки

- **Смешивание логов**: Без разделения по модулям трудно найти нужную информацию.
- **Неправильное использование MDC**: Забытый `MDC.clear()` может привести к некорректным данным в логах.
- **Избыточное логирование**: Логирование всех деталей снижает читаемость.

## Дополнительно

### Интеграция с Jenkins

Настройте Jenkins для запуска тестов и анализа логов:

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

Логи из файлов `ui.log`, `api.log` и т.д. можно архивировать в Jenkins:

```groovy
post {
    always {
        archiveArtifacts artifacts: 'logs/*.log', allowEmptyArchive: true
    }
}
```

### Использование Testcontainers

Пример логирования на уровне хранилища с Testcontainers:

```java
import org.junit.jupiter.api.Test;
import org.slf4j.MDC;
import org.testcontainers.containers.PostgreSQLContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;
import java.util.UUID;

@Testcontainers
public class DatabaseTest {
    @Container
    private static final PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:13")
            .withDatabaseName("testdb")
            .withUsername("user")
            .withPassword("pass");

    @Test
    void testDatabaseQuery() {
        MDC.put("requestId", UUID.randomUUID().toString());
        MDC.put("userId", "testUser");
        DatabaseClient client = new DatabaseClient();
        String username = client.getUserFromDb(postgres.createConnection(""), 1);
        MdcLogger.logWithContext("Запрос к БД", "testUser");
        MDC.clear();
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