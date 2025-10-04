# Использование явных и неявных ожиданий

Ожидания в автотестах необходимы для синхронизации тестов с динамически загружаемыми веб-страницами. Неправильное управление ожиданиями может привести к нестабильным (flaky) тестам или ненужным задержкам. В этой статье рассматриваются неявные и явные ожидания в Selenium WebDriver, их правильное использование, а также лучшие практики для повышения стабильности тестов.

## Неявные ожидания

Неявные ожидания (`implicitlyWait`) задают глобальный таймаут для всех операций поиска элементов (`findElement` и `findElements`). Если элемент не найден сразу, WebDriver будет ждать указанное время, прежде чем выбросить исключение `NoSuchElementException`.

### Настройка неявных ожиданий

Неявные ожидания настраиваются в базовой конфигурации тестов, обычно в `BaseTest` или при инициализации `WebDriver`.

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import java.time.Duration;

public class BaseTest {
    protected WebDriver driver;

    public void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        // Установка неявного ожидания на 10 секунд
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
        driver.get("https://example.com");
    }

    public void tearDown() {
        if (driver != null) {
            driver.quit();
        }
    }
}
```

### Особенности неявных ожиданий
- **Простота**: Глобальная настройка, не требующая дополнительного кода для каждого элемента.
- **Ограничения**: Применяется ко всем операциям поиска, что может замедлить тесты, если элементы появляются быстрее или, наоборот, не появляются вовсе.
- **Рекомендация**: Используйте неявные ожидания только для базовой настройки с минимальным таймаутом (5–10 секунд). Для точечной работы с нестабильными элементами применяйте явные ожидания.

## Явные ожидания

Явные ожидания (`WebDriverWait` и `ExpectedConditions`) позволяют ждать определённого условия для конкретного элемента, например, его видимости, кликабельности или изменения атрибута. Это делает тесты более устойчивыми к асинхронным операциям на странице.

### Пример использования WebDriverWait

Класс `BasePage` включает методы с явными ожиданиями для работы с элементами. Используется `PageFactory` для инициализации локаторов.

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.PageFactory;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;

public class BasePage {
    protected WebDriver driver;
    protected WebDriverWait wait;

    public BasePage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        PageFactory.initElements(driver, this);
    }

    // Ожидание видимости элемента
    protected WebElement waitForElementVisible(WebElement element) {
        return wait.until(ExpectedConditions.visibilityOf(element));
    }

    // Ожидание кликабельности элемента
    protected WebElement waitForElementClickable(WebElement element) {
        return wait.until(ExpectedConditions.elementToBeClickable(element));
    }

    // Клик с ожиданием
    protected void click(WebElement element) {
        waitForElementClickable(element).click();
    }

    // Проверка отображения элемента
    protected boolean isElementDisplayed(WebElement element) {
        return waitForElementVisible(element).isDisplayed();
    }
}
```

### Пример Page Object с явными ожиданиями

Класс `LoginPage` использует методы из `BasePage` для работы с элементами страницы логина.

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;

public class LoginPage extends BasePage {
    @FindBy(id = "username")
    private WebElement usernameField;

    @FindBy(id = "password")
    private WebElement passwordField;

    @FindBy(id = "loginButton")
    private WebElement loginButton;

    @FindBy(id = "errorMessage")
    private WebElement errorMessage;

    public LoginPage(WebDriver driver) {
        super(driver);
    }

    public LoginPage enterUsername(String username) {
        waitForElementVisible(usernameField).sendKeys(username);
        return this;
    }

    public LoginPage enterPassword(String password) {
        waitForElementVisible(passwordField).sendKeys(password);
        return this;
    }

    public void clickLoginButton() {
        click(loginButton);
    }

    public boolean isErrorMessageDisplayed() {
        return isElementDisplayed(errorMessage);
    }
}
```

### Пример теста

#### JUnit 5
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
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(5)); // Неявное ожидание
        loginPage = new LoginPage(driver);
        driver.get("https://example.com/login");
    }

    @Test
    void testInvalidLogin() {
        // Arrange
        String username = "invalidUser";
        String password = "invalidPass";

        // Act
        loginPage.enterUsername(username)
                .enterPassword(password)
                .clickLoginButton();

        // Assert
        assertThat(loginPage.isErrorMessageDisplayed()).isTrue();
    }

    @AfterEach
    void tearDown() {
        driver.quit();
    }
}
```

#### TestNG
```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import static org.assertj.core.api.Assertions.assertThat;

public class LoginTest {
    private WebDriver driver;
    private LoginPage loginPage;

    @BeforeMethod
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(5)); // Неявное ожидание
        loginPage = new LoginPage(driver);
        driver.get("https://example.com/login");
    }

    @Test
    void testInvalidLogin() {
        // Given
        String username = "invalidUser";
        String password = "invalidPass";

        // When
        loginPage.enterUsername(username)
                .enterPassword(password)
                .clickLoginButton();

        // Then
        assertThat(loginPage.isErrorMessageDisplayed()).isTrue();
    }

    @AfterMethod
    void tearDown() {
        driver.quit();
    }
}
```

## FluentWait для сложных случаев

`FluentWait` предоставляет гибкость для настройки ожиданий, включая интервал опроса и игнорирование определённых исключений. Используйте `FluentWait` для нестандартных сценариев, например, ожидания исчезновения элемента.

### Пример FluentWait
```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.FluentWait;
import org.openqa.selenium.support.ui.Wait;
import java.time.Duration;
import java.util.NoSuchElementException;

public class BasePage {
    protected WebDriver driver;
    protected Wait<WebDriver> fluentWait;

    public BasePage(WebDriver driver) {
        this.driver = driver;
        this.fluentWait = new FluentWait<>(driver)
                .withTimeout(Duration.ofSeconds(10))
                .pollingEvery(Duration.ofMillis(500))
                .ignoring(NoSuchElementException.class);
    }

    // Ожидание исчезновения элемента
    public boolean waitForElementToDisappear(WebElement element) {
        return fluentWait.until(driver -> {
            try {
                return !element.isDisplayed();
            } catch (NoSuchElementException e) {
                return true;
            }
        });
    }
}
```

## Избежание Thread.sleep

Использование `Thread.sleep` приводит к ненужным задержкам и делает тесты хрупкими, так как фиксированное время ожидания может быть либо слишком длинным, либо недостаточным. Вместо этого применяйте:

- `WebDriverWait` с `ExpectedConditions` для явных ожиданий.
- `FluentWait` для нестандартных условий.
- Retry-механизмы (например, через библиотеки вроде `awaitility` или собственные реализации).

### Пример retry-механизма
```java
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.ExpectedConditions;

public class LoginPage extends BasePage {
    public boolean isErrorMessageDisplayedWithRetry(int maxAttempts, long intervalMillis) {
        int attempts = 0;
        while (attempts < maxAttempts) {
            try {
                return wait.until(ExpectedConditions.visibilityOf(errorMessage)).isDisplayed();
            } catch (Exception e) {
                attempts++;
                try {
                    Thread.sleep(intervalMillis); // Используется только внутри retry
                } catch (InterruptedException ie) {
                    Thread.currentThread().interrupt();
                }
            }
        }
        return false;
    }
}
```

**Примечание**: Используйте retry-механизмы только в крайних случаях, когда `WebDriverWait` или `FluentWait` не справляются с задачей.

## Maven зависимости

Для работы с Selenium, JUnit 5, TestNG и AssertJ добавьте зависимости в `pom.xml`:

```xml
<dependencies>
    <!-- Selenium -->
    <dependency>
        <groupId>org.seleniumhq.selenium</groupId>
        <artifactId>selenium-java</artifactId>
        <version>4.25.0</version>
    </dependency>
    <!-- WebDriverManager -->
    <dependency>
        <groupId>io.github.bonigarcia</groupId>
        <artifactId>webdrivermanager</artifactId>
        <version>5.9.2</version>
    </dependency>
    <!-- JUnit 5 -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>5.11.3</version>
        <scope>test</scope>
    </dependency>
    <!-- AssertJ -->
    <dependency>
        <groupId>org.assertj</groupId>
        <artifactId>assertj-core</artifactId>
        <version>3.26.3</version>
        <scope>test</scope>
    </dependency>
    <!-- TestNG (если используется) -->
    <dependency>
        <groupId>org.testng</groupId>
        <artifactId>testng</artifactId>
        <version>7.10.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

## Интеграция с CI/CD

Для запуска тестов с ожиданиями в Jenkins или GitHub Actions настройте Maven. Пример для GitHub Actions:

```yaml
name: CI Pipeline
on:
  push:
    branches:
      - main
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
      - name: Run tests
        run: mvn clean test
```

## Отладка

Для отладки ожиданий в IntelliJ IDEA:
- Установите точки останова в методах с `WebDriverWait` или `FluentWait`.
- Проверяйте состояние элементов через DevTools браузера.
- Логируйте действия с помощью SLF4J и Logback, например: `logger.info("Waiting for element visibility: " + element)`.

## Распространённые ошибки

- **Злоупотребление неявными ожиданиями**: Установка слишком большого таймаута замедляет тесты, а слишком маленького — приводит к ошибкам.
- **Использование Thread.sleep**: Фиксированные задержки увеличивают время выполнения тестов и не гарантируют стабильности.
- **Игнорирование исключений**: Не обрабатывая исключения в `FluentWait`, можно пропустить важные ошибки.
- **Сложные условия ожиданий**: Неправильное использование `ExpectedConditions` (например, ожидание неверного состояния) приводит к нестабильным тестам.

## Дополнительно

- **Selenide**: Рассмотрите Selenide для упрощения ожиданий. Оно автоматически обрабатывает ожидания видимости и кликабельности (например, `$(byId("username")).shouldBe(visible).setValue("user")`).
- **Allure**: Добавьте аннотации `@Step` для методов с ожиданиями, чтобы улучшить читаемость отчётов.
- **Testcontainers**: Используйте Testcontainers для запуска браузеров в Docker, чтобы тестировать ожидания в изолированных окружениях.

## Источники
- [Selenium Documentation](https://www.selenium.dev/documentation/)
- [WebDriverManager](https://bonigarcia.dev/webdrivermanager/)
- [Allure Framework](https://allurereport.org/)

---
[**← Назад к оглавлению**](../README.md)