# Стандарты навигации и переходов между страницами

Эффективная навигация и переходы между страницами в UI-тестировании — ключевые аспекты для создания читаемых, стабильных и масштабируемых тестов. В этой статье описаны стандарты для реализации явных переходов с использованием методов `openPage()` и `navigateToSection()`, проверки загрузки страниц через наличие ключевых элементов, а также работы с URL-шаблонами и параметрами. Все примеры основаны на Java, Selenium WebDriver, Page Object Model (POM) и других инструментах из заданного стека.

## Явные переходы между страницами

Для управления навигацией в тестах рекомендуется использовать методы `openPage()` и `navigateToSection()` в классах Page Object. Это обеспечивает читаемость кода и инкапсуляцию логики переходов.

### Метод `openPage()`

Метод `openPage()` используется для прямого перехода на страницу по URL. Он инкапсулирует логику открытия страницы и проверки её загрузки.

```java
import io.qameta.allure.Step;
import org.openqa.selenium.WebDriver;
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

    protected abstract void verifyPageLoaded();
}
```

### Метод `navigateToSection()`

Метод `navigateToSection()` используется для перехода к определённой секции или странице через взаимодействие с элементами интерфейса (например, клик по ссылке или кнопке).

```java
import io.qameta.allure.Step;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HomePage extends BasePage {
    private static final Logger logger = LoggerFactory.getLogger(HomePage.class);

    @FindBy(id = "login-link")
    private WebElement loginLink;

    public HomePage(WebDriver driver) {
        super(driver);
        PageFactory.initElements(driver, this);
    }

    @Step("Navigating to login section")
    public LoginPage navigateToSection() {
        logger.info("Clicking login link");
        loginLink.click();
        return new LoginPage(driver);
    }

    @Override
    protected void verifyPageLoaded() {
        wait.until(d -> loginLink.isDisplayed());
        logger.info("Home page loaded, login link is visible");
    }
}
```

### Пример использования в тесте

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

public class NavigationTest {
    private static final Logger logger = LoggerFactory.getLogger(NavigationTest.class);
    private WebDriver driver;
    private HomePage homePage;

    @BeforeEach
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        homePage = new HomePage(driver);
        logger.info("WebDriver initialized");
    }

    @Test
    @Step("Test navigation to login page")
    void testNavigation() {
        // Arrange
        String homeUrl = "https://example.com";

        // Act
        homePage.openPage(homeUrl);
        LoginPage loginPage = homePage.navigateToSection();
        logger.info("Navigated to login page");

        // Assert
        assertThat(driver.getCurrentUrl()).contains("login");
        logger.info("Login page URL verified");
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
- **Явные переходы**: Методы `openPage()` и `navigateToSection()` делают навигацию прозрачной и читаемой.
- **Возврат Page Object**: Метод `navigateToSection()` возвращает объект следующей страницы для цепочного вызова.
- **Логирование**: Каждое действие логируется через SLF4J для отслеживания шагов.

## Проверка загрузки страницы

Для устойчивости тестов необходимо проверять, что страница полностью загружена, прежде чем выполнять действия. Это достигается через проверку наличия ключевых элементов с использованием `WebDriverWait`.

### Пример проверки загрузки

```java
import io.qameta.allure.Step;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LoginPage extends BasePage {
    private static final Logger logger = LoggerFactory.getLogger(LoginPage.class);

    @FindBy(id = "username")
    private WebElement usernameField;

    public LoginPage(WebDriver driver) {
        super(driver);
        PageFactory.initElements(driver, this);
    }

    @Override
    @Step("Verifying login page loaded")
    protected void verifyPageLoaded() {
        wait.until(ExpectedConditions.visibilityOf(usernameField));
        logger.info("Login page loaded, username field is visible");
    }

    @Step("Entering credentials: username={username}")
    public void enterCredentials(String username, String password) {
        logger.info("Entering username: {}", username);
        usernameField.sendKeys(username);
        logger.info("Entering password: [PROTECTED]");
        // Дополнительные действия
    }
}
```

### Лучшие практики
- **Стабильные локаторы**: Используйте `id` или `data-*` атрибуты для ключевых элементов.
- **Ожидания**: Применяйте `WebDriverWait` с условиями, такими как `visibilityOf` или `elementToBeClickable`.
- **Логирование**: Логируйте успешную загрузку страницы и возможные ошибки.

### Обработка ошибок

```java
public void verifyPageLoadedWithRetry() {
    try {
        wait.until(ExpectedConditions.visibilityOf(usernameField));
        logger.info("Login page loaded successfully");
    } catch (Exception e) {
        logger.error("Failed to load login page: {}", e.getMessage());
        throw new IllegalStateException("Login page did not load", e);
    }
}
```

## Использование URL-шаблонов и параметров

URL-шаблоны и параметры позволяют гибко формировать адреса страниц, особенно при работе с динамическими URL.

### Формирование URL с параметрами

```java
import io.qameta.allure.Step;
import org.openqa.selenium.WebDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.net.URLEncoder;
import java.nio.charset.StandardCharsets;

public class DynamicUrlPage extends BasePage {
    private static final Logger logger = LoggerFactory.getLogger(DynamicUrlPage.class);
    private static final String BASE_URL = "https://example.com/search";

    @FindBy(id = "search-results")
    private WebElement searchResults;

    public DynamicUrlPage(WebDriver driver) {
        super(driver);
    }

    @Step("Opening search page with query: {query}")
    public void openSearchPage(String query) {
        String encodedQuery = URLEncoder.encode(query, StandardCharsets.UTF_8);
        String url = String.format("%s?q=%s", BASE_URL, encodedQuery);
        logger.info("Navigating to search URL: {}", url);
        driver.get(url);
        verifyPageLoaded();
    }

    @Override
    protected void verifyPageLoaded() {
        wait.until(ExpectedConditions.visibilityOf(searchResults));
        logger.info("Search page loaded, results are visible");
    }
}
```

### Пример теста с динамическим URL

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

public class DynamicUrlTest {
    private static final Logger logger = LoggerFactory.getLogger(DynamicUrlTest.class);
    private WebDriver driver;
    private DynamicUrlPage searchPage;

    @BeforeEach
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        searchPage = new DynamicUrlPage(driver);
        logger.info("WebDriver initialized");
    }

    @Test
    void testSearchWithDynamicUrl() {
        // Arrange
        String query = "test query";

        // Act
        searchPage.openSearchPage(query);
        logger.info("Navigated to search page with query: {}", query);

        // Assert
        assertThat(driver.getCurrentUrl()).contains("q=test+query");
        logger.info("Search page URL verified");
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
- **Кодирование параметров**: Используйте `URLEncoder` для безопасной обработки специальных символов.
- **Гибкость**: URL-шаблоны позволяют легко адаптировать тесты под разные окружения.
- **Проверка URL**: Используйте AssertJ для проверки корректности сформированного URL.

## Интеграция с Allure

Для отчётности используйте аннотацию `@Step` для документирования шагов навигации и проверки загрузки страниц. Зависимости для Allure:

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

## Интеграция с Jenkins

Настройте запуск тестов в Jenkins с поддержкой Allure для отображения шагов навигации.

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
   - Установите точки останова в методах `openPage()` или `navigateToSection()`.
   - Проверяйте локаторы ключевых элементов через DevTools браузера.
   - Используйте логи SLF4J для отслеживания переходов и ошибок.

2. **Устойчивость тестов**:
   - Используйте стабильные локаторы (`id`, `data-*`) для ключевых элементов.
   - Проверяйте элементы через `isDisplayed()` или `isEnabled()` перед действиями.
   - Обрабатывайте исключения, такие как `TimeoutException`:

```java
public void safeOpenPage(String url) {
    try {
        openPage(url);
    } catch (Exception e) {
        logger.error("Failed to open page {}: {}", url, e.getMessage());
        throw new IllegalStateException("Page navigation failed", e);
    }
}
```

## Распространённые ошибки

1. **Отсутствие проверки загрузки**: Игнорирование `verifyPageLoaded()` приводит к ошибкам при взаимодействии с элементами.
2. **Некорректные URL**: Ошибки в формировании динамических URL из-за неправильной обработки параметров.
3. **Неустойчивые локаторы**: Использование ненадёжных локаторов, таких как `xpath` или `css`, вместо `id`.
4. **Недостаточное логирование**: Отсутствие логов затрудняет отладку ошибок навигации.

## Дополнительно

- **Selenide для упрощения навигации**:
  - Selenide предоставляет удобные методы для работы с URL и проверкой элементов:

```java
import com.codeborne.selenide.Configuration;
import com.codeborne.selenide.Selenide;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import static com.codeborne.selenide.Selenide.$;
import static com.codeborne.selenide.Selenide.open;

public class SelenideNavigation {
    private static final Logger logger = LoggerFactory.getLogger(SelenideNavigation.class);

    public static void navigateToLogin(String baseUrl) {
        Configuration.browser = "chrome";
        Configuration.timeout = 10000;
        String url = baseUrl + "/login";
        open(url);
        $("#username").shouldBe(visible);
        logger.info("Navigated to login page: {}", url);
    }
}
```

- **Testcontainers для тестирования**:
  - Используйте Testcontainers для проверки навигации в изолированном окружении:

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
import org.openqa.selenium.WebDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testcontainers.containers.BrowserWebDriverContainer;

public class TestcontainersNavigation {
    private static final Logger logger = LoggerFactory.getLogger(TestcontainersNavigation.class);

    public void testNavigation() {
        try (BrowserWebDriverContainer<?> container = new BrowserWebDriverContainer<>()
                .withCapabilities(new ChromeOptions())) {
            container.start();
            WebDriver driver = container.getWebDriver();
            HomePage homePage = new HomePage(driver);
            homePage.openPage("https://example.com");
            logger.info("Navigated to home page in Testcontainers");
            driver.quit();
        }
    }
}
```

## Источники
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/)
- [Selenide Documentation](https://selenide.org/)
- [Testcontainers Documentation](https://www.testcontainers.org/)

---
[**← Назад к оглавлению**](../README.md)