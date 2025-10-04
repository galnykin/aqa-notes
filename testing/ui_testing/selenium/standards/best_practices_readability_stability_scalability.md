# Best Practices: читаемость, устойчивость, масштабируемость

Создание качественных автоматизированных тестов требует соблюдения лучших практик, обеспечивающих читаемость, устойчивость и масштабируемость. В этой статье описаны ключевые принципы: один тест — один сценарий, минимизация дублирования кода, чёткие названия методов и тестов, устойчивость к изменениям UI, а также возможность запуска тестов в разных окружениях и браузерах. Все примеры реализованы на Java с использованием Selenium WebDriver, JUnit 5, TestNG, Allure и других инструментов из заданного стека.

## Один тест — один сценарий

Каждый тест должен проверять один конкретный сценарий, чтобы упростить отладку и анализ результатов. Это улучшает читаемость и делает тесты более предсказуемыми.

### Пример теста с одним сценарием

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import io.qameta.allure.Step;
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
        logger.info("WebDriver initialized for login test");
    }

    @Test
    @Step("Verify successful login with valid credentials")
    void testSuccessfulLogin() {
        loginPage.openPage("https://example.com/login");
        loginPage.loginAs("validuser", "validpass");
        assertThat(driver.getCurrentUrl()).contains("dashboard");
        logger.info("Successful login verified");
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

### Особенности
- **Один сценарий**: Тест проверяет только успешный вход, без дополнительных проверок (например, отображения элементов).
- **Читаемость**: Название теста `testSuccessfulLogin` и аннотация `@Step` ясно описывают цель.
- **Логирование**: SLF4J фиксирует ключевые шаги для отладки.

## Минимизация дублирования кода

Для устранения дублирования кода общие действия выносятся в базовые классы, утилиты или компоненты Page Object Model (POM).

### Базовый класс для страниц

```java
import io.qameta.allure.Step;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.time.Duration;

public abstract class BasePage {
    protected static final Logger logger = LoggerFactory.getLogger(BasePage.class);
    protected final WebDriver driver;
    protected final WebDriverWait wait;

    public BasePage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    }

    @Step("Opening page: {url}")
    public void openPage(String url) {
        logger.info("Navigating to URL: {}", url);
        driver.get(url);
        verifyPageLoaded();
    }

    @Step("Waiting for element to be visible: {element}")
    protected void waitForElementVisible(WebElement element) {
        logger.info("Waiting for element to be visible");
        wait.until(ExpectedConditions.visibilityOf(element));
        logger.info("Element is visible");
    }

    protected abstract void verifyPageLoaded();
}
```

### Утилитный класс для общих действий

```java
import io.qameta.allure.Step;
import org.openqa.selenium.WebElement;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class WebDriverUtils {
    private static final Logger logger = LoggerFactory.getLogger(WebDriverUtils.class);

    @Step("Clicking element safely")
    public static void safeClick(WebElement element) {
        logger.info("Clicking element");
        element.click();
    }

    @Step("Entering text: {text} into element")
    public static void enterText(WebElement element, String text) {
        logger.info("Entering text: {}", text);
        element.clear();
        element.sendKeys(text);
    }
}
```

### Пример Page Object с минимизацией дублирования

```java
import io.qameta.allure.Step;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LoginPage extends BasePage {
    private static final Logger logger = LoggerFactory.getLogger(LoginPage.class);

    @FindBy(id = "username")
    private WebElement usernameField;

    @FindBy(id = "password")
    private WebElement passwordField;

    @FindBy(id = "login-button")
    private WebElement loginButton;

    public LoginPage(WebDriver driver) {
        super(driver);
        PageFactory.initElements(driver, this);
    }

    @Step("Logging in as user: {username}")
    public DashboardPage loginAs(String username, String password) {
        logger.info("Logging in as user: {}", username);
        WebDriverUtils.enterText(usernameField, username);
        WebDriverUtils.enterText(passwordField, password);
        WebDriverUtils.safeClick(loginButton);
        return new DashboardPage(driver);
    }

    @Override
    protected void verifyPageLoaded() {
        waitForElementVisible(usernameField);
        logger.info("Login page loaded, username field is visible");
    }
}
```

### Особенности
- **Базовый класс**: `BasePage` содержит общие методы для всех страниц.
- **Утилиты**: `WebDriverUtils` инкапсулирует повторяющиеся действия.
- **POM**: Логика страниц изолирована, что снижает дублирование.

## Чёткие названия методов и тестов

Названия методов и тестов должны быть понятными, описывать их цель и соответствовать бизнес-логике.

### Примеры названий
- Тест: `testSuccessfulLogin` (ясно, что проверяется успешный вход).
- Метод: `loginAs(String username, String password)` (описывает действие и параметры).
- Page Object: `LoginPage`, `DashboardPage` (соответствуют страницам приложения).

### Правила
- Используйте глаголы: `verify`, `check`, `navigateTo`, `fillForm`.
- Указывайте контекст: `testLoginWithInvalidCredentials`, `openSearchPage`.
- Избегайте общих слов: `test1`, `doSomething`.
- Следуйте camelCase для методов и тестов.

## Устойчивость к изменениям UI

Для минимизации влияния изменений UI тесты должны быть устойчивы к обновлениям DOM и дизайна.

### Использование стабильных локаторов

```java
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;

public class DashboardPage extends BasePage {
    @FindBy(id = "dashboard-title")
    private WebElement title;

    @FindBy(css = "[data-test-id='user-menu']")
    private WebElement userMenu;

    public DashboardPage(WebDriver driver) {
        super(driver);
        PageFactory.initElements(driver, this);
    }

    @Override
    protected void verifyPageLoaded() {
        waitForElementVisible(title);
        logger.info("Dashboard page loaded, title is visible");
    }
}
```

### Лучшие практики
- **Локаторы**: Используйте `id` или `data-*` атрибуты вместо `xpath` или сложных `css`.
- **Ожидания**: Применяйте `WebDriverWait` для проверки видимости или кликабельности элементов.
- **Проверки**: Используйте `isDisplayed()` и `isEnabled()` перед действиями.

```java
public void safeClickUserMenu() {
    try {
        wait.until(ExpectedConditions.elementToBeClickable(userMenu));
        WebDriverUtils.safeClick(userMenu);
    } catch (TimeoutException e) {
        logger.error("User menu not clickable: {}", e.getMessage());
        throw new IllegalStateException("Failed to click user menu", e);
    }
}
```

### Обработка нестабильных действий

Оберните нестабильные действия в `try-catch` для повышения устойчивости.

```java
@Step("Navigating to profile page")
public ProfilePage navigateToProfile() {
    try {
        waitForElementVisible(userMenu);
        WebDriverUtils.safeClick(userMenu);
        logger.info("Navigated to profile page");
        return new ProfilePage(driver);
    } catch (Exception e) {
        logger.error("Navigation to profile failed: {}", e.getMessage());
        throw new IllegalStateException("Profile navigation failed", e);
    }
}
```

## Возможность запуска в разных окружениях и браузерах

Тесты должны поддерживать запуск в разных окружениях (локально, CI/CD) и браузерах (Chrome, Firefox, Edge).

### Настройка BrowserFactory

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.firefox.FirefoxDriver;
import org.openqa.selenium.firefox.FirefoxOptions;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class BrowserFactory {
    private static final Logger logger = LoggerFactory.getLogger(BrowserFactory.class);

    public static WebDriver createDriver(String browser, String environment) {
        WebDriver driver;
        switch (browser.toLowerCase()) {
            case "chrome":
                WebDriverManager.chromedriver().setup();
                ChromeOptions chromeOptions = new ChromeOptions();
                if ("ci".equalsIgnoreCase(environment)) {
                    chromeOptions.addArguments("--headless", "--disable-gpu");
                }
                driver = new ChromeDriver(chromeOptions);
                logger.info("Chrome browser initialized for {} environment", environment);
                break;
            case "firefox":
                WebDriverManager.firefoxdriver().setup();
                FirefoxOptions firefoxOptions = new FirefoxOptions();
                if ("ci".equalsIgnoreCase(environment)) {
                    firefoxOptions.addArguments("--headless");
                }
                driver = new FirefoxDriver(firefoxOptions);
                logger.info("Firefox browser initialized for {} environment", environment);
                break;
            default:
                throw new IllegalArgumentException("Unsupported browser: " + browser);
        }
        return driver;
    }
}
```

### Конфигурация окружений

Используйте переменные окружения или файлы конфигурации для управления URL и настройками.

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class EnvironmentConfig {
    private static final Logger logger = LoggerFactory.getLogger(EnvironmentConfig.class);

    public static String getBaseUrl(String environment) {
        String baseUrl;
        switch (environment.toLowerCase()) {
            case "prod":
                baseUrl = "https://prod.example.com";
                break;
            case "stage":
                baseUrl = "https://stage.example.com";
                break;
            default:
                baseUrl = "https://localhost:8080";
                break;
        }
        logger.info("Using base URL: {} for environment: {}", baseUrl, environment);
        return baseUrl;
    }
}
```

### Пример теста с поддержкой разных браузеров

```java
import io.qameta.allure.Step;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import static org.assertj.core.api.Assertions.assertThat;

public class CrossBrowserTest {
    private static final Logger logger = LoggerFactory.getLogger(CrossBrowserTest.class);
    private WebDriver driver;
    private LoginPage loginPage;

    @BeforeEach
    void setUp() {
        String browser = System.getProperty("browser", "chrome");
        String environment = System.getProperty("env", "localhost");
        driver = BrowserFactory.createDriver(browser, environment);
        loginPage = new LoginPage(driver);
        logger.info("Test setup completed for browser: {} and environment: {}", browser, environment);
    }

    @Test
    @Step("Verify login in {0} environment")
    void testLoginAcrossEnvironments() {
        String env = System.getProperty("env", "localhost");
        String baseUrl = EnvironmentConfig.getBaseUrl(env);
        loginPage.openPage(baseUrl + "/login");
        loginPage.loginAs("validuser", "validpass");
        assertThat(driver.getCurrentUrl()).contains("dashboard");
        logger.info("Login test passed in {} environment", env);
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

### Запуск в CI/CD

Настройте Jenkins или GitHub Actions для запуска тестов в разных браузерах и окружениях.

#### Пример GitHub Actions Workflow

```yaml
name: Cross-Browser Tests
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        browser: [chrome, firefox]
        env: [stage, prod]
    steps:
      - uses: actions/checkout@v3
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
      - name: Run tests
        env:
          BROWSER: ${{ matrix.browser }}
          ENV: ${{ matrix.env }}
        run: mvn clean test
      - name: Publish Allure Report
        uses: actions/upload-artifact@v3
        with:
          name: allure-report-${{ matrix.browser }}-${{ matrix.env }}
          path: target/allure-results
```

## Интеграция с Allure

Для отчётности используйте аннотацию `@Step` и настройте Allure для отображения шагов и результатов.

```xml
<dependencies>
    <dependency>
        <groupId>io.qameta.allure</groupId>
        <artifactId>allure-junit5</artifactId>
        <version>2.30.0</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

## Отладка и устойчивость

1. **Отладка в IntelliJ IDEA**:
   - Установите точки останова в `BrowserFactory` или Page Object.
   - Проверяйте локаторы через DevTools браузера.
   - Используйте логи SLF4J для отслеживания выполнения тестов.

2. **Устойчивость тестов**:
   - Используйте стабильные локаторы (`id`, `data-*`).
   - Применяйте `WebDriverWait` для ожидания элементов.
   - Обрабатывайте исключения:

```java
public void safeLoginAs(String username, String password) {
    try {
        loginAs(username, password);
    } catch (Exception e) {
        logger.error("Login failed: {}", e.getMessage());
        throw new IllegalStateException("Login failed", e);
    }
}
```

## Распространённые ошибки

1. **Многосценарийные тесты**: Тесты, проверяющие несколько сценариев, усложняют анализ ошибок.
2. **Дублирование кода**: Неправильное использование POM или утилит приводит к избыточности.
3. **Нечёткие названия**: Общие названия, такие как `test1`, снижают читаемость.
4. **Нестабильные локаторы**: Использование `xpath` или `css` вместо `id` делает тесты хрупкими.
5. **Жёстко закодированные значения**: URL или параметры в коде затрудняют запуск в разных окружениях.

## Дополнительно

- **Selenide для упрощения**:
  - Selenide упрощает написание читаемых и устойчивых тестов:

```java
import com.codeborne.selenide.Configuration;
import com.codeborne.selenide.Selenide;
import com.codeborne.selenide.logevents.SelenideLogger;
import io.qameta.allure.selenide.AllureSelenide;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import static com.codeborne.selenide.Selenide.*;

public class SelenideBestPractices {
    private static final Logger logger = LoggerFactory.getLogger(SelenideBestPractices.class);

    public static void setupSelenide(String browser, String environment) {
        SelenideLogger.addListener("AllureSelenide", new AllureSelenide());
        Configuration.browser = browser;
        Configuration.timeout = 10000;
        Configuration.headless = "ci".equalsIgnoreCase(environment);
        logger.info("Selenide configured for browser: {} and environment: {}", browser, environment);
    }

    public static void testLogin(String baseUrl) {
        setupSelenide("chrome", "stage");
        open(baseUrl + "/login");
        $("#username").setValue("validuser");
        $("#password").setValue("validpass");
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
import io.qameta.allure.Step;
import org.openqa.selenium.WebDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testcontainers.containers.BrowserWebDriverContainer;

public class TestcontainersBestPractices {
    private static final Logger logger = LoggerFactory.getLogger(TestcontainersBestPractices.class);

    @Step("Test login in Testcontainers")
    public void testLoginInContainer(String baseUrl) {
        try (BrowserWebDriverContainer<?> container = new BrowserWebDriverContainer<>()
                .withCapabilities(new ChromeOptions())) {
            container.start();
            WebDriver driver = container.getWebDriver();
            LoginPage loginPage = new LoginPage(driver);
            loginPage.openPage(baseUrl + "/login");
            loginPage.loginAs("validuser", "validpass");
            logger.info("Login test completed in Testcontainers");
            driver.quit();
        }
    }
}
```

## Источники
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/)
- [Selenide Documentation](https://selenide.org/)
- [Testcontainers Documentation](https://www.testcontainers.org/)
- [Allure Framework](https://allurereport.org/)

---
[**← Назад к оглавлению**](../README.md)