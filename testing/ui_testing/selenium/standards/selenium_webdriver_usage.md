# Использование Selenium WebDriver для автоматизации тестирования

Selenium WebDriver — это инструмент для автоматизации тестирования веб-приложений, позволяющий эмулировать действия пользователя в браузере. Он широко используется для проверки пользовательских сценариев, кросс-браузерного тестирования и интеграции с CI/CD системами. В этой статье описаны ключевые аспекты использования Selenium WebDriver, лучшие практики и примеры реализации с применением Java, JUnit 5, TestNG, Selenide и Allure для отчётности.

## Основы работы с Selenium WebDriver

Selenium WebDriver взаимодействует с браузером через драйверы (например, ChromeDriver, GeckoDriver). Для упрощения управления драйверами используется WebDriverManager, который автоматически загружает и настраивает нужные версии драйверов. WebDriver позволяет выполнять действия, такие как клики, ввод текста, навигация по страницам, а также проверять состояние элементов интерфейса.

### Зависимости Maven
Для начала работы добавьте зависимости в `pom.xml`:

```xml
<dependencies>
    <!-- Selenium WebDriver -->
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
    <!-- TestNG -->
    <dependency>
        <groupId>org.testng</groupId>
        <artifactId>testng</artifactId>
        <version>7.10.2</version>
        <scope>test</scope>
    </dependency>
    <!-- AssertJ -->
    <dependency>
        <groupId>org.assertj</groupId>
        <artifactId>assertj-core</artifactId>
        <version>3.26.3</version>
        <scope>test</scope>
    </dependency>
    <!-- Selenide -->
    <dependency>
        <groupId>com.codeborne</groupId>
        <artifactId>selenide</artifactId>
        <version>7.5.0</version>
    </dependency>
    <!-- Allure -->
    <dependency>
        <groupId>io.qameta.allure</groupId>
        <artifactId>allure-junit5</artifactId>
        <version>2.29.0</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>io.qameta.allure</groupId>
        <artifactId>allure-testng</artifactId>
        <version>2.29.0</version>
        <scope>test</scope>
    </dependency>
    <!-- SLF4J + Logback -->
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.5.11</version>
    </dependency>
</dependencies>
```

## Page Object Model

Page Object Model (POM) — это паттерн проектирования, который помогает сделать тесты читаемыми и поддерживаемыми. Каждый веб-страница представлена отдельным классом, содержащим локаторы и методы для взаимодействия с элементами.

### Пример Page Object с JUnit 5
```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;

public class LoginPage {
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

    public void login(String username, String password) {
        usernameField.sendKeys(username);
        passwordField.sendKeys(password);
        loginButton.click();
    }
}
```

### Тест с JUnit 5
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
        loginPage = new LoginPage(driver);
        driver.get("https://example.com/login");
    }

    @Test
    void shouldLoginSuccessfully() {
        // Arrange
        String username = "testuser";
        String password = "testpass";

        // Act
        loginPage.login(username, password);

        // Assert
        assertThat(driver.getCurrentUrl()).contains("dashboard");
    }

    @AfterEach
    void tearDown() {
        driver.quit();
    }
}
```

### Тест с TestNG
```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import static org.testng.Assert.assertTrue;

public class LoginTestNG {
    private WebDriver driver;
    private LoginPage loginPage;

    @BeforeMethod
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        loginPage = new LoginPage(driver);
        driver.get("https://example.com/login");
    }

    @Test
    void shouldLoginSuccessfully() {
        // Given
        String username = "testuser";
        String password = "testpass";

        // When
        loginPage.login(username, password);

        // Then
        assertTrue(driver.getCurrentUrl().contains("dashboard"));
    }

    @AfterMethod
    void tearDown() {
        driver.quit();
    }
}
```

## Использование Selenide для упрощения тестов

Selenide — это обёртка над Selenium WebDriver, которая упрощает написание UI-тестов благодаря встроенным ожиданиям и лаконичному синтаксису.

### Пример теста с Selenide
```java
import com.codeborne.selenide.Selenide;
import com.codeborne.selenide.SelenideElement;
import org.junit.jupiter.api.Test;
import static com.codeborne.selenide.Selenide.$;
import static com.codeborne.selenide.Selenide.open;
import static org.assertj.core.api.Assertions.assertThat;

public class SelenideLoginTest {
    @Test
    void shouldLoginSuccessfully() {
        // Arrange
        open("https://example.com/login");
        SelenideElement usernameField = $("#username");
        SelenideElement passwordField = $("#password");
        SelenideElement loginButton = $("#loginButton");

        // Act
        usernameField.sendKeys("testuser");
        passwordField.sendKeys("testpass");
        loginButton.click();

        // Assert
        assertThat(Selenide.webdriver().driver().getWebDriver().getCurrentUrl()).contains("dashboard");
    }
}
```

## Устойчивость тестов

Для предотвращения нестабильных (flaky) тестов используйте:

- **Стабильные локаторы**: Предпочитайте `id`, `data-*` атрибуты или уникальные CSS-селекторы.
- **Ожидания**: Используйте `WebDriverWait` или `FluentWait` для обработки асинхронных элементов.
- **Проверки состояния**: Проверяйте `isDisplayed()` и `isEnabled()` перед взаимодействием с элементами.

### Пример ожидания
```java
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;

public void waitForElement(WebDriver driver, WebElement element) {
    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    wait.until(ExpectedConditions.visibilityOf(element));
}
```

## Интеграция с CI/CD (Jenkins)

Для запуска тестов в Jenkins настройте задачу Maven:

1. Создайте файл `Jenkinsfile`:
```groovy
pipeline {
    agent any
    stages {
        stage('Build and Test') {
            steps {
                sh 'mvn clean test'
            }
        }
        stage('Allure Report') {
            steps {
                allure includeProperties: false, jdk: '', results: [[path: 'target/allure-results']]
            }
        }
    }
}
```

2. Убедитесь, что Allure Jenkins Plugin установлен.
3. Настройте Allure в `pom.xml` для генерации отчётов:
```xml
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
```

## Отладка тестов

Для отладки используйте IntelliJ IDEA:
- Установите точки останова в тестовом коде или Page Object.
- Проверяйте локаторы через DevTools браузера.
- Логируйте действия с помощью SLF4J:
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LoginPage {
    private static final Logger logger = LoggerFactory.getLogger(LoginPage.class);

    public void login(String username, String password) {
        logger.info("Attempting login with username: {}", username);
        // Логика входа
    }
}
```

## Распространённые ошибки

- **Нестабильные локаторы**: Использование хрупких локаторов, таких как длинные XPath. Решение: используйте `id` или `data-*`.
- **Игнорирование ожиданий**: Отсутствие `WebDriverWait` приводит к ошибкам из-за асинхронности. Решение: добавляйте явные ожидания.
- **Закрытие браузера**: Неправильное завершение сессии WebDriver. Решение: всегда вызывайте `driver.quit()` в `@AfterEach`/`@AfterMethod`.

## Связь с клиент-серверной моделью

Selenium WebDriver взаимодействует с веб-приложением через клиент-серверную модель: тесты (клиент) отправляют HTTP-запросы через драйвер браузера (сервер). Это позволяет эмулировать действия пользователя, такие как клики и ввод данных, а также проверять ответы сервера, отображаемые в интерфейсе.

## Дополнительно

- **Allure TestOps**: Используйте для централизованного хранения и анализа отчётов.
- **Testcontainers**: Подходит для тестирования API или интеграции с базами данных в контейнерах.
- **Checkstyle/PMD/SpotBugs**: Настройте для проверки качества кода тестов.

## Источники
- [Selenium Documentation](https://www.selenium.dev/documentation/)
- [Selenide Documentation](https://selenide.org/documentation.html)
- [Allure Framework](https://allurereport.org/docs/)

---
[**← Назад к оглавлению**](../README.md)