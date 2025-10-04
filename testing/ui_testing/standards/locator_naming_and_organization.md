# Именование и организация локаторов

Локаторы — ключевой элемент UI-тестирования, обеспечивающий взаимодействие с элементами веб-страницы. Неправильное именование или организация локаторов снижает читаемость кода, усложняет поддержку тестов и увеличивает вероятность ошибок. Этот раздел описывает лучшие практики по именованию и организации локаторов в UI-тестах с использованием Selenium WebDriver и Selenide.

## Стабильные и читаемые локаторы

Стабильность локаторов критически важна для устойчивости тестов. Используйте селекторы, которые редко меняются при обновлении интерфейса, такие как `id`, `name` или `data-testid`. Избегайте хрупких локаторов, таких как длинные XPath или CSS-селекторы, зависящие от структуры DOM.

- **Рекомендации по выбору селекторов**:
  - `id`: уникальный идентификатор элемента, предпочтительный выбор.
  - `name`: подходит для форм, если атрибут уникален.
  - `data-testid`: кастомный атрибут, часто используемый разработчиками для тестирования.
  - CSS-селекторы: используйте короткие и специфичные, например, `.button.submit`.
  - XPath: применяйте только для сложных случаев, где другие селекторы невозможны, например, `//input[@aria-label='Search']`.

- **Проверки стабильности**:
  - Проверяйте локаторы в DevTools браузера перед использованием.
  - Используйте методы `isDisplayed()` и `isEnabled()` для проверки состояния элемента перед взаимодействием.

Пример кода (JUnit 5):
```java
import org.junit.jupiter.api.Test;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import io.github.bonigarcia.wdm.WebDriverManager;

public class LoginTest {
    @Test
    void testLoginButton() {
        WebDriverManager.chromedriver().setup();
        WebDriver driver = new ChromeDriver();
        
        // Arrange
        driver.get("https://example.com/login");
        By loginButton = By.id("login-submit"); // Стабильный локатор по id
        
        // Act
        boolean isButtonDisplayed = driver.findElement(loginButton).isDisplayed();
        
        // Assert
        assert isButtonDisplayed : "Login button is not displayed";
        driver.quit();
    }
}
```

Пример кода (TestNG):
```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.Test;
import io.github.bonigarcia.wdm.WebDriverManager;

public class LoginTest {
    @Test
    public void testLoginButton() {
        WebDriverManager.chromedriver().setup();
        WebDriver driver = new ChromeDriver();
        
        // Arrange
        driver.get("https://example.com/login");
        By loginButton = By.id("login-submit"); // Стабильный локатор по id
        
        // Act
        boolean isButtonDisplayed = driver.findElement(loginButton).isDisplayed();
        
        // Assert
        org.testng.Assert.assertTrue(isButtonDisplayed, "Login button is not displayed");
        driver.quit();
    }
}
```

## Хранение локаторов

Локаторы должны быть организованы так, чтобы поддерживать читаемость и повторное использование кода. Используйте модель Page Object для структурирования локаторов и связанных с ними действий.

- **Хранение локаторов**:
  - Храните локаторы как `private final By` в классах Page Object.
  - Используйте аннотации `@FindBy` (Selenium) или `@$` (Selenide) для упрощения поиска элементов.
  - Выносите сложные локаторы в отдельные классы или константы, если они используются в нескольких местах.

- **Именование локаторов**:
  - Называйте локаторы по их функциональному назначению: `loginButton`, `searchInput`, `errorMessage`.
  - Избегайте общих имен, таких как `button` или `input`, без контекста.
  - Используйте camelCase для имен переменных.

Пример Page Object с локаторами (Selenium):
```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;

public class LoginPage {
    private final WebDriver driver;
    
    // Локаторы
    private final By usernameInput = By.name("username");
    private final By passwordInput = By.name("password");
    private final By loginButton = By.id("login-submit");
    private final By errorMessage = By.cssSelector(".error-message");

    public LoginPage(WebDriver driver) {
        this.driver = driver;
    }

    public void enterUsername(String username) {
        driver.findElement(usernameInput).sendKeys(username);
    }

    public void enterPassword(String password) {
        driver.findElement(passwordInput).sendKeys(password);
    }

    public void clickLoginButton() {
        driver.findElement(loginButton).click();
    }

    public boolean isErrorMessageDisplayed() {
        return driver.findElement(errorMessage).isDisplayed();
    }
}
```

Пример теста с Page Object (JUnit 5):
```java
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import io.github.bonigarcia.wdm.WebDriverManager;

public class LoginTest {
    @Test
    void testInvalidLogin() {
        WebDriverManager.chromedriver().setup();
        WebDriver driver = new ChromeDriver();
        
        // Arrange
        driver.get("https://example.com/login");
        LoginPage loginPage = new LoginPage(driver);
        
        // Act
        loginPage.enterUsername("invalidUser");
        loginPage.enterPassword("wrongPassword");
        loginPage.clickLoginButton();
        
        // Assert
        assert loginPage.isErrorMessageDisplayed() : "Error message is not displayed";
        driver.quit();
    }
}
```

Пример теста с Page Object (TestNG):
```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.Test;
import io.github.bonigarcia.wdm.WebDriverManager;

public class LoginTest {
    @Test
    public void testInvalidLogin() {
        WebDriverManager.chromedriver().setup();
        WebDriver driver = new ChromeDriver();
        
        // Arrange
        driver.get("https://example.com/login");
        LoginPage loginPage = new LoginPage(driver);
        
        // Act
        loginPage.enterUsername("invalidUser");
        loginPage.enterPassword("wrongPassword");
        loginPage.clickLoginButton();
        
        // Assert
        org.testng.Assert.assertTrue(loginPage.isErrorMessageDisplayed(), "Error message is not displayed");
        driver.quit();
    }
}
```

## Ожидания и устойчивость тестов

Для повышения стабильности тестов используйте явные ожидания (`WebDriverWait` в Selenium, встроенные ожидания в Selenide). Это позволяет избежать ошибок, связанных с асинхронной загрузкой элементов.

- **Типы ожиданий**:
  - Ожидание видимости (`visibilityOfElementLocated`).
  - Ожидание кликабельности (`elementToBeClickable`).
  - Ожидание изменения текста или атрибута.

Пример с явным ожиданием (Selenium):
```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;

public class LoginPage {
    private final WebDriver driver;
    private final WebDriverWait wait;
    private final By loginButton = By.id("login-submit");

    public LoginPage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    }

    public void clickLoginButton() {
        wait.until(ExpectedConditions.elementToBeClickable(loginButton)).click();
    }
}
```

Пример с Selenide:
```java
import com.codeborne.selenide.SelenideElement;
import static com.codeborne.selenide.Selenide.$;
import static com.codeborne.selenide.Condition.visible;

public class LoginPage {
    private final SelenideElement loginButton = $("#login-submit");

    public void clickLoginButton() {
        loginButton.shouldBe(visible).click();
    }
}
```

## Отладка локаторов

Для отладки локаторов используйте IntelliJ IDEA Debug:
- Устанавливайте точки останова в местах вызова локаторов.
- Проверяйте значение локаторов в окне Variables.
- Используйте DevTools браузера для проверки селекторов в реальном времени.

**Распространённые ошибки**:
- Использование неуникальных локаторов, приводящее к выбору неправильного элемента.
- Игнорирование динамических атрибутов, таких как `id`, генерируемый на лету.
- Отсутствие ожиданий, из-за чего тест падает при медленной загрузке страницы.

## Дополнительно

- **Вынос локаторов в отдельные классы**: Если проект большой, создавайте отдельные классы для хранения локаторов, например, `Locators.java`, чтобы централизовать управление.
- **Логирование**: Используйте SLF4J с Logback для логирования найденных элементов или ошибок. Например:
  ```java
  import org.slf4j.Logger;
  import org.slf4j.LoggerFactory;

  public class LoginPage {
      private static final Logger logger = LoggerFactory.getLogger(LoginPage.class);
      private final By loginButton = By.id("login-submit");

      public void clickLoginButton(WebDriver driver) {
          logger.info("Clicking login button with locator: {}", loginButton);
          driver.findElement(loginButton).click();
      }
  }
  ```

- **Пример `pom.xml` для зависимостей**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>ui-tests</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- Selenium -->
        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-java</artifactId>
            <version>4.11.0</version>
        </dependency>
        <!-- Selenide -->
        <dependency>
            <groupId>com.codeborne</groupId>
            <artifactId>selenide</artifactId>
            <version>6.19.1</version>
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
        <!-- SLF4J + Logback -->
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.4.11</version>
        </dependency>
    </dependencies>
</project>
```

## Источники
- [Selenium Documentation](https://www.selenium.dev/documentation/)
- [Selenide Documentation](https://selenide.org/documentation.html)
- [WebDriverManager GitHub](https://github.com/bonigarcia/webdrivermanager)

---
[**← Назад к оглавлению**](../README.md)