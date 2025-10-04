# Ожидания в UI-тестировании

Ожидания в UI-тестировании решают проблему синхронизации между тестом и приложением. UI-элементы могут загружаться асинхронно, поэтому тесты должны дожидаться их появления или изменения состояния. Использование правильных механизмов ожиданий повышает стабильность тестов, минимизируя ложные сбои.

## Типы ожиданий

### Неявные ожидания
Неявные ожидания задают глобальный таймаут для всех операций поиска элементов. Если элемент не найден сразу, WebDriver ждёт указанное время перед выбросом исключения. Используются для базовой настройки, но не рекомендуются для сложных сценариев из-за отсутствия гибкости.

```java
// Настройка неявного ожидания
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import java.time.Duration;

WebDriver driver = new ChromeDriver();
driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
```

### Явные ожидания
Явные ожидания (WebDriverWait, ExpectedConditions) применяются для конкретных условий, таких как видимость элемента, кликабельность или изменение текста. Они позволяют точно контролировать момент, когда тест продолжит выполнение.

```java
// Пример явного ожидания с JUnit 5
import org.junit.jupiter.api.Test;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;

public class ExplicitWaitTest {
    @Test
    void testExplicitWait() {
        // Arrange
        WebDriver driver = new ChromeDriver();
        driver.get("https://example.com");
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));

        // Act
        wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("submit-button")));
        driver.findElement(By.id("submit-button")).click();

        // Assert
        wait.until(ExpectedConditions.titleContains("Success"));
        assert driver.getTitle().contains("Success");

        driver.quit();
    }
}
```

```java
// Пример явного ожидания с TestNG
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.testng.annotations.Test;
import java.time.Duration;

public class ExplicitWaitTestNG {
    @Test
    public void testExplicitWait() {
        // Arrange
        WebDriver driver = new ChromeDriver();
        driver.get("https://example.com");
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));

        // Act
        wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("submit-button")));
        driver.findElement(By.id("submit-button")).click();

        // Assert
        wait.until(ExpectedConditions.titleContains("Success"));
        assert driver.getTitle().contains("Success");

        driver.quit();
    }
}
```

### FluentWait
FluentWait предоставляет гибкость для кастомных условий. Позволяет задавать интервал опроса, максимальное время ожидания и игнорировать определённые исключения. Используется для нестандартных сценариев, например, ожидания сложных состояний элементов.

```java
// Пример FluentWait с JUnit 5
import org.junit.jupiter.api.Test;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.ui.FluentWait;
import org.openqa.selenium.support.ui.Wait;
import java.time.Duration;
import java.util.function.Function;

public class FluentWaitTest {
    @Test
    void testFluentWait() {
        // Arrange
        WebDriver driver = new ChromeDriver();
        driver.get("https://example.com");
        Wait<WebDriver> wait = new FluentWait<>(driver)
                .withTimeout(Duration.ofSeconds(10))
                .pollingEvery(Duration.ofMillis(500))
                .ignoring(org.openqa.selenium.NoSuchElementException.class);

        // Act
        wait.until(driver1 -> driver1.findElement(By.id("dynamic-element")).isDisplayed());
        driver.findElement(By.id("dynamic-element")).click();

        // Assert
        assert driver.findElement(By.id("result")).isDisplayed();

        driver.quit();
    }
}
```

```java
// Пример FluentWait с TestNG
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.ui.FluentWait;
import org.openqa.selenium.support.ui.Wait;
import org.testng.annotations.Test;
import java.time.Duration;
import java.util.function.Function;

public class FluentWaitTestNG {
    @Test
    public void testFluentWait() {
        // Arrange
        WebDriver driver = new ChromeDriver();
        driver.get("https://example.com");
        Wait<WebDriver> wait = new FluentWait<>(driver)
                .withTimeout(Duration.ofSeconds(10))
                .pollingEvery(Duration.ofMillis(500))
                .ignoring(org.openqa.selenium.NoSuchElementException.class);

        // Act
        wait.until(driver1 -> driver1.findElement(By.id("dynamic-element")).isDisplayed());
        driver.findElement(By.id("dynamic-element")).click();

        // Assert
        assert driver.findElement(By.id("result")).isDisplayed();

        driver.quit();
    }
}
```

## Page Object Model и ожидания
Ожидания интегрируются в Page Object для повышения читаемости и повторного использования кода. Пример показывает страницу с использованием явных ожиданий.

```java
// Page Object: LoginPage.java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;

public class LoginPage {
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

    public void login(String username, String password) {
        wait.until(ExpectedConditions.visibilityOf(usernameField)).sendKeys(username);
        wait.until(ExpectedConditions.visibilityOf(passwordField)).sendKeys(password);
        wait.until(ExpectedConditions.elementToBeClickable(loginButton)).click();
    }
}
```

```java
// Тест с Page Object и JUnit 5
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

public class LoginTest {
    @Test
    void testLogin() {
        // Arrange
        WebDriver driver = new ChromeDriver();
        driver.get("https://example.com/login");
        LoginPage loginPage = new LoginPage(driver);

        // Act
        loginPage.login("user", "password");

        // Assert
        assert driver.findElement(org.openqa.selenium.By.id("welcome-message")).isDisplayed();
        driver.quit();
    }
}
```

```java
// Тест с Page Object и TestNG
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.Test;

public class LoginTestNG {
    @Test
    public void testLogin() {
        // Arrange
        WebDriver driver = new ChromeDriver();
        driver.get("https://example.com/login");
        LoginPage loginPage = new LoginPage(driver);

        // Act
        loginPage.login("user", "password");

        // Assert
        assert driver.findElement(org.openqa.selenium.By.id("welcome-message")).isDisplayed();
        driver.quit();
    }
}
```

## Локаторы и стабильность
Для ожиданий важны стабильные локаторы. Рекомендации:
- Используй `id`, `name` или уникальные `data-*` атрибуты.
- Избегай длинных XPath или CSS-селекторов, зависящих от структуры DOM.
- Проверяй элементы с помощью `isDisplayed()` и `isEnabled()` перед взаимодействием.

Пример проверки локатора в IntelliJ IDEA:
1. Установи точку останова на строке с локатором.
2. Запусти тест в режиме Debug.
3. В окне Debug проверь значение переменной `WebElement` или выполни `driver.findElement(By.id("element-id"))` в консоли.

## Лучшие практики и отладка
- **Избегай Thread.sleep**: Фиксированные задержки увеличивают время выполнения тестов и не гарантируют стабильности.
- **Retry-механизмы**: Используй FluentWait или библиотеки вроде Awaitility для повторных попыток.
- **Логирование**: Настрой SLF4J с Logback для записи событий, связанных с ожиданиями.
- **CI/CD интеграция**: В Jenkins настрой запуск тестов с параметром таймаута, чтобы учесть задержки сети.

Пример `pom.xml` для зависимостей:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>ui-tests</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!-- Selenium WebDriver -->
        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-java</artifactId>
            <version>4.16.1</version>
        </dependency>
        <!-- JUnit 5 -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.10.2</version>
            <scope>test</scope>
        </dependency>
        <!-- TestNG -->
        <dependency>
            <groupId>org.testng</groupId>
            <artifactId>testng</artifactId>
            <version>7.10.2</version>
            <scope>test</scope>
        </dependency>
        <!-- SLF4J и Logback -->
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.4.14</version>
        </dependency>
    </dependencies>
</project>
```

## Распространённые ошибки
- **Слишком короткий таймаут**: Увеличивай время ожидания для медленных приложений, но не превышай разумных пределов (10-15 секунд).
- **Зависимость от нестабильных локаторов**: Проверяй локаторы на уникальность в DevTools браузера.
- **Игнорирование исключений**: Логируй исключения в FluentWait для диагностики.

## Источники
- [Selenium Documentation](https://www.selenium.dev/documentation/)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [TestNG Documentation](https://testng.org/doc/)

---
[**← Назад к оглавлению**](../README.md)