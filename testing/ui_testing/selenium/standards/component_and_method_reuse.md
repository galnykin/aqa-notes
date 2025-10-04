# Повторное использование компонентов и методов

Повторное использование компонентов и методов в автоматизации тестирования повышает читаемость, масштабируемость и удобство поддержки тестов. В этой статье описаны подходы к выносу общих действий в базовые классы или утилиты, созданию отдельных объектов для компонентов (Header, Sidebar, Modal) и реализации переиспользуемых методов, таких как `loginAs(user)` и `fillForm(data)`, с использованием Java, Selenium WebDriver, Page Object Model (POM) и других инструментов из заданного стека.

## Вынос общих действий в базовые классы и утилиты

Общие действия, такие как настройка WebDriver, ожидания, логирование или проверки, выносятся в базовые классы или утилиты для устранения дублирования кода.

### Базовый класс для страниц

Базовый класс `BasePage` содержит общую функциональность для всех страниц, включая настройку ожиданий, логирование и проверки.

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

Утилитный класс `WebDriverUtils` содержит методы для работы с элементами, окнами или алертами.

```java
import io.qameta.allure.Step;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.time.Duration;

public class WebDriverUtils {
    private static final Logger logger = LoggerFactory.getLogger(WebDriverUtils.class);
    private static final WebDriverWait wait;

    public WebDriverUtils(WebDriver driver) {
        wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    }

    @Step("Clicking element safely")
    public static void safeClick(WebElement element) {
        wait.until(ExpectedConditions.elementToBeClickable(element));
        logger.info("Clicking element");
        element.click();
    }

    @Step("Entering text: {text} into element")
    public static void enterText(WebElement element, String text) {
        wait.until(ExpectedConditions.visibilityOf(element));
        logger.info("Entering text: {}", text);
        element.clear();
        element.sendKeys(text);
    }
}
```

### Особенности
- **Базовый класс**: `BasePage` предоставляет общие методы для навигации и ожиданий, которые наследуются всеми страницами.
- **Утилиты**: `WebDriverUtils` содержит статические методы для повторяющихся действий, таких как клики или ввод текста.
- **Логирование**: Все действия логируются через SLF4J для отладки.
- **Allure**: Аннотации `@Step` добавляют шаги в отчёты.

## Использование компонентов как отдельных объектов

Компоненты пользовательского интерфейса, такие как Header, Sidebar или Modal, выделяются в отдельные классы для инкапсуляции их логики и повторного использования.

### Пример компонента Header

```java
import io.qameta.allure.Step;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HeaderComponent {
    private static final Logger logger = LoggerFactory.getLogger(HeaderComponent.class);
    private final WebDriver driver;

    @FindBy(id = "login-link")
    private WebElement loginLink;

    @FindBy(id = "profile-link")
    private WebElement profileLink;

    public HeaderComponent(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this);
    }

    @Step("Navigating to login page from header")
    public LoginPage navigateToLogin() {
        logger.info("Clicking login link in header");
        WebDriverUtils.safeClick(loginLink);
        return new LoginPage(driver);
    }

    @Step("Navigating to profile page from header")
    public ProfilePage navigateToProfile() {
        logger.info("Clicking profile link in header");
        WebDriverUtils.safeClick(profileLink);
        return new ProfilePage(driver);
    }
}
```

### Пример компонента Modal

```java
import io.qameta.allure.Step;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class ModalComponent {
    private static final Logger logger = LoggerFactory.getLogger(ModalComponent.class);
    private final WebDriver driver;

    @FindBy(id = "modal-confirm-button")
    private WebElement confirmButton;

    @FindBy(id = "modal-cancel-button")
    private WebElement cancelButton;

    public ModalComponent(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this);
    }

    @Step("Confirming modal action")
    public void confirmModal() {
        logger.info("Clicking confirm button in modal");
        WebDriverUtils.safeClick(confirmButton);
    }

    @Step("Cancelling modal action")
    public void cancelModal() {
        logger.info("Clicking cancel button in modal");
        WebDriverUtils.safeClick(cancelButton);
    }
}
```

### Интеграция компонентов в Page Object

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class DashboardPage extends BasePage {
    private static final Logger logger = LoggerFactory.getLogger(DashboardPage.class);
    private final HeaderComponent header;
    private final ModalComponent modal;

    @FindBy(id = "dashboard-title")
    private WebElement dashboardTitle;

    public DashboardPage(WebDriver driver) {
        super(driver);
        this.header = new HeaderComponent(driver);
        this.modal = new ModalComponent(driver);
    }

    public HeaderComponent getHeader() {
        return header;
    }

    public ModalComponent getModal() {
        return modal;
    }

    @Override
    protected void verifyPageLoaded() {
        waitForElementVisible(dashboardTitle);
        logger.info("Dashboard page loaded, title is visible");
    }
}
```

### Особенности
- **Инкапсуляция**: Компоненты (Header, Modal) содержат только свою логику, что упрощает поддержку.
- **Повторное использование**: Компоненты можно использовать на разных страницах, если они присутствуют в DOM.
- **Читаемость**: Логика навигации и взаимодействия с компонентами отделена от страниц.

## Переиспользуемые методы

Переиспользуемые методы, такие как `loginAs(user)` и `fillForm(data)`, упрощают выполнение типичных сценариев.

### Метод `loginAs(user)`

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

    @Step("Logging in as user: {user.username}")
    public DashboardPage loginAs(User user) {
        logger.info("Logging in as user: {}", user.getUsername());
        WebDriverUtils.enterText(usernameField, user.getUsername());
        WebDriverUtils.enterText(passwordField, user.getPassword());
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

### Класс `User` (с использованием Lombok)

```java
import lombok.Data;

@Data
public class User {
    private final String username;
    private final String password;
}
```

### Метод `fillForm(data)`

```java
import io.qameta.allure.Step;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.Map;

public class ProfilePage extends BasePage {
    private static final Logger logger = LoggerFactory.getLogger(ProfilePage.class);

    @FindBy(id = "name")
    private WebElement nameField;

    @FindBy(id = "email")
    private WebElement emailField;

    @FindBy(id = "submit-button")
    private WebElement submitButton;

    public ProfilePage(WebDriver driver) {
        super(driver);
        PageFactory.initElements(driver, this);
    }

    @Step("Filling profile form with data")
    public void fillForm(Map<String, String> data) {
        logger.info("Filling profile form with data: {}", data.keySet());
        if (data.containsKey("name")) {
            WebDriverUtils.enterText(nameField, data.get("name"));
        }
        if (data.containsKey("email")) {
            WebDriverUtils.enterText(emailField, data.get("email"));
        }
        WebDriverUtils.safeClick(submitButton);
    }

    @Override
    protected void verifyPageLoaded() {
        waitForElementVisible(nameField);
        logger.info("Profile page loaded, name field is visible");
    }
}
```

### Пример теста с JUnit 5

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

import java.util.HashMap;
import java.util.Map;

import static org.assertj.core.api.Assertions.assertThat;

public class ComponentReuseTest {
    private static final Logger logger = LoggerFactory.getLogger(ComponentReuseTest.class);
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
    @Step("Test login and form filling")
    void testLoginAndFormFilling() {
        // Arrange
        String homeUrl = "https://example.com";
        User user = new User("testuser", "testpass");
        Map<String, String> formData = new HashMap<>();
        formData.put("name", "John Doe");
        formData.put("email", "john.doe@example.com");

        // Act
        homePage.openPage(homeUrl);
        LoginPage loginPage = homePage.getHeader().navigateToLogin();
        DashboardPage dashboardPage = loginPage.loginAs(user);
        ProfilePage profilePage = dashboardPage.getHeader().navigateToProfile();
        profilePage.fillForm(formData);
        logger.info("Form filled and submitted");

        // Assert
        assertThat(driver.getCurrentUrl()).contains("profile");
        logger.info("Profile page URL verified");
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

## Интеграция с Allure

Для отчётности используйте аннотацию `@Step` для документирования действий. Зависимости для Allure:

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

Настройте запуск тестов в Jenkins с поддержкой Allure для отображения шагов.

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
   - Установите точки останова в методах `loginAs()`, `fillForm()` или компонентах.
   - Проверяйте локаторы через DevTools браузера.
   - Используйте логи SLF4J для отслеживания выполнения методов.

2. **Устойчивость тестов**:
   - Используйте стабильные локаторы (`id`, `data-*`) для элементов компонентов.
   - Проверяйте элементы через `isDisplayed()` или `isEnabled()` перед действиями.
   - Обрабатывайте исключения:

```java
public void safeLoginAs(User user) {
    try {
        loginAs(user);
    } catch (Exception e) {
        logger.error("Login failed for user {}: {}", user.getUsername(), e.getMessage());
        throw new IllegalStateException("Login failed", e);
    }
}
```

## Распространённые ошибки

1. **Дублирование кода**: Неправильное использование базовых классов или утилит приводит к повторению логики.
2. **Некорректные локаторы**: Использование ненадёжных локаторов в компонентах, таких как `xpath`.
3. **Отсутствие проверок**: Игнорирование `verifyPageLoaded()` приводит к ошибкам при взаимодействии с элементами.
4. **Недостаточное логирование**: Отсутствие логов затрудняет отладку.

## Дополнительно

- **Selenide для упрощения**:
  - Selenide упрощает работу с компонентами и переиспользуемыми методами:

```java
import com.codeborne.selenide.Configuration;
import com.codeborne.selenide.Selenide;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.Map;

import static com.codeborne.selenide.Selenide.*;

public class SelenideComponents {
    private static final Logger logger = LoggerFactory.getLogger(SelenideComponents.class);

    public static void loginAs(String username, String password) {
        Configuration.browser = "chrome";
        Configuration.timeout = 10000;
        open("https://example.com/login");
        $("#username").setValue(username);
        $("#password").setValue(password);
        $("#login-button").click();
        logger.info("Logged in as {}", username);
    }

    public static void fillForm(Map<String, String> data) {
        if (data.containsKey("name")) {
            $("#name").setValue(data.get("name"));
        }
        if (data.containsKey("email")) {
            $("#email").setValue(data.get("email"));
        }
        $("#submit-button").click();
        logger.info("Form filled with data: {}", data.keySet());
    }
}
```

- **Testcontainers для тестирования**:
  - Используйте Testcontainers для проверки компонентов в изолированном окружении:

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

public class TestcontainersComponents {
    private static final Logger logger = LoggerFactory.getLogger(TestcontainersComponents.class);

    public void testComponents() {
        try (BrowserWebDriverContainer<?> container = new BrowserWebDriverContainer<>()
                .withCapabilities(new ChromeOptions())) {
            container.start();
            WebDriver driver = container.getWebDriver();
            HomePage homePage = new HomePage(driver);
            homePage.openPage("https://example.com");
            LoginPage loginPage = homePage.getHeader().navigateToLogin();
            loginPage.loginAs(new User("testuser", "testpass"));
            logger.info("Test completed in Testcontainers");
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