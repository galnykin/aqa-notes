# Стандарты логирования действий и ошибок

Логирование в автоматизированном тестировании фиксирует ключевые действия и ошибки, упрощая отладку и анализ результатов. Эффективное логирование включает запись шагов теста, исключений и интеграцию с отчётными системами.

## Настройка логгеров

Используйте SLF4J с Logback для гибкого и производительного логирования. SLF4J предоставляет единый API, а Logback — реализацию с поддержкой конфигурации через XML.

### Пример настройки Logback

Создайте файл `logback.xml` для конфигурации логгера.

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

### Зависимости в `pom.xml`

Минимальный `pom.xml` для SLF4J и Logback:

```xml
<dependencies>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>2.0.9</version>
    </dependency>
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.4.11</version>
    </dependency>
</dependencies>
```

- `slf4j-api`: API для логирования.
- `logback-classic`: Реализация логирования с поддержкой вывода в консоль и файл.

## Логирование ключевых шагов

Логируйте действия, такие как переходы по страницам, клики, ввод данных. Используйте уровень `INFO` для шагов и `ERROR` для исключений.

### Пример логирования в UI-тесте (JUnit 5)

```java
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LoginTest {
    private static final Logger logger = LoggerFactory.getLogger(LoginTest.class);

    @Test
    void loginTest() {
        // Arrange
        logger.info("Инициализация WebDriver");
        WebDriver driver = new ChromeDriver();
        driver.get("https://example.com/login");

        // Act
        logger.info("Ввод логина");
        driver.findElement(By.id("username")).sendKeys("testuser");
        logger.info("Ввод пароля");
        driver.findElement(By.id("password")).sendKeys("password");
        logger.info("Клик по кнопке входа");
        driver.findElement(By.id("loginButton")).click();

        // Assert
        logger.info("Проверка успешного входа");
        Assertions.assertTrue(driver.findElement(By.id("welcome")).isDisplayed());
        driver.quit();
    }
}
```

### Пример логирования в UI-тесте (TestNG)

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testng.annotations.Test;

public class LoginTest {
    private static final Logger logger = LoggerFactory.getLogger(LoginTest.class);

    @Test
    public void loginTest() {
        // Arrange
        logger.info("Инициализация WebDriver");
        WebDriver driver = new ChromeDriver();
        driver.get("https://example.com/login");

        // Act
        logger.info("Ввод логина");
        driver.findElement(By.id("username")).sendKeys("testuser");
        logger.info("Ввод пароля");
        driver.findElement(By.id("password")).sendKeys("password");
        logger.info("Клик по кнопке входа");
        driver.findElement(By.id("loginButton")).click();

        // Assert
        logger.info("Проверка успешного входа");
        Assert.assertTrue(driver.findElement(By.id("welcome")).isDisplayed());
        driver.quit();
    }
}
```

## Логирование исключений

Обрабатывайте исключения и логируйте их с указанием причины. Это помогает быстрее находить источник проблемы.

### Пример обработки исключений

```java
@Test
void loginTestWithException() {
    logger.info("Инициализация WebDriver");
    WebDriver driver = null;
    try {
        driver = new ChromeDriver();
        driver.get("https://example.com/login");
        logger.info("Ввод логина");
        driver.findElement(By.id("nonexistent")).sendKeys("testuser");
    } catch (NoSuchElementException e) {
        logger.error("Элемент не найден: {}", e.getMessage());
        throw e;
    } finally {
        if (driver != null) {
            logger.info("Закрытие WebDriver");
            driver.quit();
        }
    }
}
```

## Интеграция с Allure

Allure интегрируется с логами для формирования отчётов. Используйте `Allure.step` для явного указания шагов.

### Пример интеграции с Allure (JUnit 5)

```java
import io.qameta.allure.Allure;
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class AllureLoginTest {
    private static final Logger logger = LoggerFactory.getLogger(AllureLoginTest.class);

    @Test
    void loginTest() {
        Allure.step("Инициализация WebDriver", () -> {
            logger.info("Инициализация WebDriver");
            WebDriver driver = new ChromeDriver();
            driver.get("https://example.com/login");
        });

        Allure.step("Ввод логина", () -> {
            logger.info("Ввод логина");
            driver.findElement(By.id("username")).sendKeys("testuser");
        });

        Allure.step("Проверка результата", () -> {
            logger.info("Проверка успешного входа");
            Assertions.assertTrue(driver.findElement(By.id("welcome")).isDisplayed());
        });
    }
}
```

## Page Object Model и логирование

Используйте Page Object для структурирования тестов. Логируйте действия на уровне страниц.

### Пример Page Object с логированием

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LoginPage {
    private static final Logger logger = LoggerFactory.getLogger(LoginPage.class);
    private WebDriver driver;

    @FindBy(id = "username")
    private WebElement usernameField;

    @FindBy(id = "password")
    private WebElement passwordField;

    @FindBy(id = "loginButton")
    private WebElement loginButton;

    public LoginPage(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this);
    }

    public void login(String username, String password) {
        logger.info("Ввод логина: {}", username);
        usernameField.sendKeys(username);
        logger.info("Ввод пароля");
        passwordField.sendKeys(password);
        logger.info("Клик по кнопке входа");
        loginButton.click();
    }
}
```

### Пример теста с Page Object (JUnit 5)

```java
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

public class LoginPageTest {
    private static final Logger logger = LoggerFactory.getLogger(LoginPageTest.class);

    @Test
    void loginTest() {
        // Arrange
        logger.info("Инициализация WebDriver");
        WebDriver driver = new ChromeDriver();
        driver.get("https://example.com/login");
        LoginPage loginPage = new LoginPage(driver);

        // Act
        loginPage.login("testuser", "password");

        // Assert
        logger.info("Проверка успешного входа");
        Assertions.assertTrue(driver.findElement(By.id("welcome")).isDisplayed());
        driver.quit();
    }
}
```

## Интеграция с Jenkins

Jenkins собирает логи тестов и отображает их в консоли. Настройте Allure в Jenkins для генерации отчётов.

### Пример Jenkinsfile

```groovy
pipeline {
    agent any
    stages {
        stage('Run Tests') {
            steps {
                sh 'mvn clean test'
            }
        }
        stage('Generate Allure Report') {
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

## Отладка в IntelliJ IDEA

Для отладки используйте точки останова в IntelliJ IDEA:
1. Установите точку останова в коде теста (например, перед `findElement`).
2. Запустите тест в режиме Debug.
3. Просмотрите значения переменных (например, локаторы или текст элемента).
4. Проверьте логи в консоли или файле `logs/test.log`.

## Дополнительно

- **Стабильные локаторы**: Используйте атрибуты `id`, `name` или уникальные `data-*` для надёжности. Избегайте длинных XPath.
- **Ожидания**: Применяйте `WebDriverWait` для синхронизации действий.
  ```java
  WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
  wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("welcome")));
  ```
- **Распространённые ошибки**:
  - Неправильная конфигурация логгера: проверьте `logback.xml`.
  - Игнорирование исключений: всегда логируйте их с контекстом.
  - Перегруженные логи: избегайте избыточного логирования на уровне `DEBUG`.

## Источники
- [SLF4J Documentation](https://www.slf4j.org/manual.html)
- [Logback Documentation](https://logback.qos.ch/manual/)
- [Allure Framework](https://docs.qameta.io/allure/)

---
[**← Назад к оглавлению**](../README.md)