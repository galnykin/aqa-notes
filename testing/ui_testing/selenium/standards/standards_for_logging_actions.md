# Стандарты логирования действий

Логирование в автоматизации тестирования — важный аспект для отслеживания действий теста, упрощения отладки и интеграции с отчётными системами. В этой статье описаны стандарты логирования ключевых шагов, таких как переходы, клики и ввод данных, с использованием SLF4J с Logback, а также интеграция логов с Allure и Jenkins.

## Настройка логирования

Для логирования используется SLF4J с бэкендом Logback. Это позволяет централизовать управление логами и интегрировать их с отчётными системами.

### Зависимости в Maven

Для работы с SLF4J и Logback добавьте следующие зависимости в `pom.xml`:

```xml
<dependencies>
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

## Логирование в Page Object

В Page Object Model (POM) логирование применяется для фиксации ключевых действий, таких как переходы, клики и ввод данных. Это помогает отслеживать последовательность шагов и упрощает отладку.

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
    private final WebDriver driver;

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

    public void enterCredentials(String username, String password) {
        logger.info("Entering username: {}", username);
        usernameField.sendKeys(username);
        logger.info("Entering password: [PROTECTED]");
        passwordField.sendKeys(password);
    }

    public void clickLoginButton() {
        logger.info("Clicking login button");
        loginButton.click();
    }

    public void navigateTo(String url) {
        logger.info("Navigating to URL: {}", url);
        driver.get(url);
    }
}
```

### Объяснение
- **Логгер**: Инициализируется через `LoggerFactory.getLogger` для текущего класса.
- **Логирование действий**: Каждое действие (ввод данных, клик, переход) сопровождается лог-сообщением с уровнем `INFO`.
- **Защита данных**: Пароли не логируются в явном виде, используется заглушка `[PROTECTED]`.

## Логирование в тестах

Тесты следуют шаблону AAA (Arrange-Act-Assert) или Given-When-Then, с логированием ключевых шагов.

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
    private LoginPage loginPage;

    @BeforeEach
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        loginPage = new LoginPage(driver);
        logger.info("WebDriver initialized and LoginPage created");
    }

    @Test
    void testSuccessfulLogin() {
        // Arrange
        String url = "https://example.com/login";
        String username = "testuser";
        String password = "testpass";

        // Act
        loginPage.navigateTo(url);
        loginPage.enterCredentials(username, password);
        loginPage.clickLoginButton();
        logger.info("Login attempt completed");

        // Assert
        assertThat(driver.getCurrentUrl()).contains("dashboard");
        logger.info("Login successful, redirected to dashboard");
    }

    @AfterEach
    void tearDown() {
        logger.info("Closing WebDriver");
        driver.quit();
    }
}
```

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

import static org.assertj.core.api.Assertions.assertThat;

public class LoginTestNG {
    private static final Logger logger = LoggerFactory.getLogger(LoginTestNG.class);
    private WebDriver driver;
    private LoginPage loginPage;

    @BeforeMethod
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        loginPage = new LoginPage(driver);
        logger.info("WebDriver initialized and LoginPage created");
    }

    @Test
    void testSuccessfulLogin() {
        // Given
        String url = "https://example.com/login";
        String username = "testuser";
        String password = "testpass";

        // When
        loginPage.navigateTo(url);
        loginPage.enterCredentials(username, password);
        loginPage.clickLoginButton();
        logger.info("Login attempt completed");

        // Then
        assertThat(driver.getCurrentUrl()).contains("dashboard");
        logger.info("Login successful, redirected to dashboard");
    }

    @AfterMethod
    void tearDown() {
        logger.info("Closing WebDriver");
        driver.quit();
    }
}
```

## Интеграция с Allure

Allure позволяет интегрировать логи с отчётами, добавляя шаги и вложения.

### Настройка Allure

Добавьте зависимости в `pom.xml`:

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
</dependencies>
```

### Логирование с Allure

Используйте аннотацию `@Step` для логирования шагов:

```java
import io.qameta.allure.Step;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LoginPageWithAllure {
    private static final Logger logger = LoggerFactory.getLogger(LoginPageWithAllure.class);
    private final WebDriver driver;

    @FindBy(id = "username")
    private WebElement usernameField;

    @FindBy(id = "password")
    private WebElement passwordField;

    @FindBy(id = "loginButton")
    private WebElement loginButton;

    public LoginPageWithAllure(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this);
    }

    @Step("Navigating to URL: {url}")
    public void navigateTo(String url) {
        logger.info("Navigating to URL: {}", url);
        driver.get(url);
    }

    @Step("Entering credentials: username={username}, password=[PROTECTED]")
    public void enterCredentials(String username, String password) {
        logger.info("Entering username: {}", username);
        usernameField.sendKeys(username);
        logger.info("Entering password: [PROTECTED]");
        passwordField.sendKeys(password);
    }

    @Step("Clicking login button")
    public void clickLoginButton() {
        logger.info("Clicking login button");
        loginButton.click();
    }
}
```

Allure автоматически добавляет шаги в отчёт, а SLF4J логирует их в консоль и файл.

## Интеграция с Jenkins

Для интеграции логов и Allure-отчётов в Jenkins настройте задачу Maven.

### Настройка Jenkins

1. **Создайте задачу Maven**:
   - Укажите путь к `pom.xml`.
   - Задайте команду: `mvn clean test`.

2. **Добавьте плагин Allure**:
   - Установите плагин Allure Jenkins.
   - В настройках задачи добавьте пост-шаг "Allure Report".
   - Укажите путь к папке `allure-results`.

3. **Логирование в Jenkins**:
   - Логи из `logback.xml` автоматически выводятся в консоль Jenkins.
   - Для сохранения логов в файл укажите путь, доступный Jenkins (например, `${WORKSPACE}/logs/test.log`).

### Пример конфигурации `pom.xml` для Jenkins

```xml
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

## Отладка логов

Для отладки используйте IntelliJ IDEA:

1. **Точки останова**: Установите их в методах Page Object или тестах.
2. **Просмотр логов**: Включите уровень логирования `DEBUG` в `logback.xml` для более детальной информации.
3. **Проверка локаторов**: Используйте DevTools браузера для проверки `id` или `data-*` локаторов, добавляя логирование для каждого действия с элементом.

### Пример изменения уровня логирования

В `logback.xml`:

```xml
<root level="debug">
    <appender-ref ref="CONSOLE"/>
    <appender-ref ref="FILE"/>
</root>
```

## Распространённые ошибки

1. **Перегруженные логи**: Логируйте только ключевые действия, избегая избыточной информации.
2. **Незащищённые данные**: Не логируйте чувствительные данные (пароли, токены).
3. **Отсутствие интеграции**: Убедитесь, что логи доступны в CI/CD и отчётах Allure.
4. **Неправильные локаторы**: Используйте стабильные локаторы (`id`, `data-*`) и логируйте их поиск.

## Дополнительно

- Для API-тестирования с RestAssured добавляйте логирование запросов и ответов:
  ```java
  import io.restassured.RestAssured;
  import org.slf4j.Logger;
  import org.slf4j.LoggerFactory;

  public class ApiTest {
      private static final Logger logger = LoggerFactory.getLogger(ApiTest.class);

      public void testApi() {
          logger.info("Sending GET request to /api/endpoint");
          RestAssured.get("/api/endpoint")
              .then()
              .log().all()
              .statusCode(200);
          logger.info("API request completed successfully");
      }
  }
  ```
- Для устойчивости тестов используйте `WebDriverWait` и логируйте ожидания:
  ```java
  import org.openqa.selenium.support.ui.WebDriverWait;
  import org.slf4j.Logger;
  import org.slf4j.LoggerFactory;

  public class WaitExample {
      private static final Logger logger = LoggerFactory.getLogger(WaitExample.class);

      public void waitForElement(WebDriver driver, WebElement element) {
          logger.info("Waiting for element to be visible");
          new WebDriverWait(driver, Duration.ofSeconds(10))
              .until(d -> element.isDisplayed());
          logger.info("Element is visible");
      }
  }
  ```

## Источники
- [SLF4J Documentation](https://www.slf4j.org/)
- [Logback Documentation](https://logback.qos.ch/)
- [Allure Framework](https://allurereport.org/)

---
[**← Назад к оглавлению**](../README.md)