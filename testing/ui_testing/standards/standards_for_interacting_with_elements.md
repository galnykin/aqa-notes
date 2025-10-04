# Стандарты взаимодействия с элементами

## Введение в стандарты взаимодействия

Стандарты взаимодействия с элементами в UI-тестировании обеспечивают стабильность, читаемость и поддерживаемость тестов. Основные принципы включают использование обёрток над действиями (`click`, `sendKeys`, `getText`), проверку состояния элементов перед взаимодействием и обработку исключений. Это позволяет минимизировать ошибки, связанные с динамическим поведением веб-приложений, и упростить отладку.

### Обёртки над действиями

Обёртки — это методы, которые инкапсулируют стандартные действия Selenium WebDriver (`click`, `sendKeys`, `getText`) с добавлением логирования, ожиданий и обработки ошибок. Они улучшают читаемость кода и упрощают диагностику проблем.

Пример кода (JUnit 5):

```java
import org.junit.jupiter.api.Test;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;

public class ElementInteractionTest {
    private WebDriver driver;
    private WebDriverWait wait;

    public ElementInteractionTest() {
        // Инициализация WebDriver и WebDriverWait
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    }

    // Обёртка для клика по элементу
    public void safeClick(By locator) {
        WebElement element = wait.until(ExpectedConditions.elementToBeClickable(locator));
        try {
            element.click();
            System.out.println("Clicked on element: " + locator);
        } catch (Exception e) {
            throw new RuntimeException("Failed to click on element: " + locator, e);
        }
    }

    // Обёртка для ввода текста
    public void safeSendKeys(By locator, String text) {
        WebElement element = wait.until(ExpectedConditions.visibilityOfElementLocated(locator));
        try {
            element.clear();
            element.sendKeys(text);
            System.out.println("Entered text '" + text + "' into element: " + locator);
        } catch (Exception e) {
            throw new RuntimeException("Failed to enter text into element: " + locator, e);
        }
    }

    @Test
    void testLogin() {
        // Arrange
        By usernameField = By.id("username");
        By loginButton = By.cssSelector(".login-button");

        // Act
        safeSendKeys(usernameField, "testuser");
        safeClick(loginButton);

        // Assert
        // Проверка результата
    }
}
```

Пример кода (TestNG):

```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.testng.annotations.Test;
import java.time.Duration;

public class ElementInteractionTestNG {
    private WebDriver driver;
    private WebDriverWait wait;

    public ElementInteractionTestNG() {
        // Инициализация WebDriver и WebDriverWait
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    }

    // Обёртка для клика по элементу
    public void safeClick(By locator) {
        WebElement element = wait.until(ExpectedConditions.elementToBeClickable(locator));
        try {
            element.click();
            System.out.println("Clicked on element: " + locator);
        } catch (Exception e) {
            throw new RuntimeException("Failed to click on element: " + locator, e);
        }
    }

    // Обёртка для ввода текста
    public void safeSendKeys(By locator, String text) {
        WebElement element = wait.until(ExpectedConditions.visibilityOfElementLocated(locator));
        try {
            element.clear();
            element.sendKeys(text);
            System.out.println("Entered text '" + text + "' into element: " + locator);
        } catch (Exception e) {
            throw new RuntimeException("Failed to enter text into element: " + locator, e);
        }
    }

    @Test
    public void testLogin() {
        // Arrange
        By usernameField = By.id("username");
        By loginButton = By.cssSelector(".login-button");

        // Act
        safeSendKeys(usernameField, "testuser");
        safeClick(loginButton);

        // Assert
        // Проверка результата
    }
}
```

### Проверка состояния элемента

Перед взаимодействием с элементом необходимо проверять его состояние: видимость (`isDisplayed`), кликабельность (`isEnabled`, `elementToBeClickable`). Это предотвращает исключения вроде `ElementNotInteractableException`.

Пример проверки состояния в Page Object:

```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;

public class LoginPage {
    private WebDriver driver;
    private WebDriverWait wait;
    private By usernameField = By.id("username");
    private By loginButton = By.cssSelector(".login-button");

    public LoginPage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    }

    public void enterUsername(String username) {
        WebElement element = wait.until(ExpectedConditions.visibilityOfElementLocated(usernameField));
        if (element.isDisplayed() && element.isEnabled()) {
            element.clear();
            element.sendKeys(username);
        } else {
            throw new IllegalStateException("Username field is not interactable");
        }
    }

    public void clickLoginButton() {
        WebElement element = wait.until(ExpectedConditions.elementToBeClickable(loginButton));
        element.click();
    }
}
```

Пример теста с использованием Page Object (JUnit 5):

```java
import org.junit.jupiter.api.Test;

public class LoginTest {
    private WebDriver driver;

    @Test
    void testLoginPage() {
        // Arrange
        LoginPage loginPage = new LoginPage(driver);

        // Act
        loginPage.enterUsername("testuser");
        loginPage.clickLoginButton();

        // Assert
        // Проверка результата
    }
}
```

## Локаторы и ожидания

### Стабильные локаторы

Для устойчивости тестов используй локаторы, которые устойчивы к изменениям интерфейса:
- Предпочитать `id`, `name`, `data-testid` над `class` или `xpath`.
- Избегать длинных XPath (`//div/div[2]/span`) и CSS-селекторов, зависящих от структуры DOM.
- Пример стабильного локатора: `By.id("username")` или `By.cssSelector("[data-testid='login-button']")`.

### Ожидания

Используй явные ожидания (`WebDriverWait`) вместо неявных (`implicitlyWait`) для повышения надёжности тестов. Пример ожидания видимости элемента:

```java
WebElement element = wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("username")));
```

### Обработка исключений

Обрабатывай распространённые исключения:
- `ElementNotInteractableException`: элемент невидим или некликабелен.
- `StaleElementReferenceException`: элемент обновился в DOM.
- Пример обработки в обёртке:

```java
public void safeClick(By locator) {
    try {
        WebElement element = wait.until(ExpectedConditions.elementToBeClickable(locator));
        element.click();
    } catch (StaleElementReferenceException e) {
        System.out.println("Retrying click due to stale element");
        WebElement element = wait.until(ExpectedConditions.elementToBeClickable(locator));
        element.click();
    } catch (ElementNotInteractableException e) {
        throw new RuntimeException("Element is not interactable: " + locator, e);
    }
}
```

## Сложные действия

Для выполнения сложных действий (drag&drop, hover, scroll) используй `Actions` из Selenium. Пример drag&drop:

```java
import org.openqa.selenium.interactions.Actions;

public void dragAndDrop(By source, By target) {
    WebElement sourceElement = wait.until(ExpectedConditions.elementToBeClickable(source));
    WebElement targetElement = wait.until(ExpectedConditions.elementToBeClickable(target));
    new Actions(driver).dragAndDrop(sourceElement, targetElement).perform();
}
```

Пример теста с drag&drop (TestNG):

```java
import org.openqa.selenium.By;
import org.testng.annotations.Test;

public class DragAndDropTest {
    private WebDriver driver;
    private WebDriverWait wait;

    public void dragAndDrop(By source, By target) {
        WebElement sourceElement = wait.until(ExpectedConditions.elementToBeClickable(source));
        WebElement targetElement = wait.until(ExpectedConditions.elementToBeClickable(target));
        new Actions(driver).dragAndDrop(sourceElement, targetElement).perform();
    }

    @Test
    public void testDragAndDrop() {
        // Arrange
        By source = By.id("draggable");
        By target = By.id("droppable");

        // Act
        dragAndDrop(source, target);

        // Assert
        // Проверка результата
    }
}
```

## Отладка в IntelliJ IDEA

Для отладки тестов:
- Устанавливай точки останова в местах вызова обёрток или проверок состояния.
- Используй вкладку "Variables" для просмотра значений локаторов и текста элементов.
- Проверяй локаторы в браузере через DevTools: `document.querySelector("[data-testid='login-button']")`.
- Логируй действия через SLF4J для анализа последовательности выполнения.

Пример настройки логирования в `pom.xml`:

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
        <version>1.5.12</version>
    </dependency>
</dependencies>
```

## Дополнительно

- Используй `Selenide` для упрощения работы с ожиданиями и локаторами, если проект позволяет.
- Для CI/CD настрой запуск тестов с логированием в Allure для анализа ошибок.
- Пример GitHub Actions workflow:

```yaml
name: UI Tests
on: [push]
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
      - name: Publish Allure report
        uses: actions/upload-artifact@v4
        with:
          name: allure-report
          path: target/allure-results
```

## Источники
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/webdriver/)
- [SLF4J User Manual](http://www.slf4j.org/manual.html)

---
[**← Назад к оглавлению**](../README.md)