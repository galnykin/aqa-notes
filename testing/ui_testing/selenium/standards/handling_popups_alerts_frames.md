# Обработка всплывающих окон, алертов и фреймов

В автоматизации UI-тестирования с использованием Selenium WebDriver часто требуется работать с всплывающими окнами, алертами и фреймами. В этой статье описаны подходы к обработке JavaScript-алертов, переключению между фреймами и управлению окнами/вкладками, включая лучшие практики, отладку и интеграцию с логами и отчётами Allure.

## Обработка алертов

JavaScript-алерты (всплывающие окна браузера) обрабатываются через интерфейс `Alert` в Selenium. Для этого используется метод `driver.switchTo().alert()`.

### Пример обработки алерта

```java
import io.qameta.allure.Step;
import org.openqa.selenium.Alert;
import org.openqa.selenium.WebDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class AlertHandler {
    private static final Logger logger = LoggerFactory.getLogger(AlertHandler.class);
    private final WebDriver driver;

    public AlertHandler(WebDriver driver) {
        this.driver = driver;
    }

    @Step("Accepting alert with text: {0}")
    public void acceptAlert() {
        Alert alert = driver.switchTo().alert();
        String alertText = alert.getText();
        logger.info("Accepting alert with text: {}", alertText);
        alert.accept();
    }

    @Step("Dismissing alert with text: {0}")
    public void dismissAlert() {
        Alert alert = driver.switchTo().alert();
        String alertText = alert.getText();
        logger.info("Dismissing alert with text: {}", alertText);
        alert.dismiss();
    }

    @Step("Entering text '{0}' into prompt alert")
    public void enterTextInPrompt(String text) {
        Alert alert = driver.switchTo().alert();
        logger.info("Entering text into prompt: {}", text);
        alert.sendKeys(text);
        alert.accept();
    }
}
```

### Особенности
- **Типы алертов**: Простые алерты (`accept()`), подтверждения (`accept()` или `dismiss()`) и промпты (с вводом текста через `sendKeys()`).
- **Ожидание алерта**: Используйте `WebDriverWait` для устойчивости, так как алерты могут появляться с задержкой.

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.time.Duration;

public class AlertWaitUtil {
    private static final Logger logger = LoggerFactory.getLogger(AlertWaitUtil.class);

    public static Alert waitForAlert(WebDriver driver, int timeoutInSeconds) {
        logger.info("Waiting for alert to appear");
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(timeoutInSeconds));
        Alert alert = wait.until(ExpectedConditions.alertIsPresent());
        logger.info("Alert appeared");
        return alert;
    }
}
```

## Переключение между фреймами

Фреймы (iframe или frame) требуют переключения контекста WebDriver с помощью `driver.switchTo().frame()`. После работы с фреймом необходимо вернуться в основной контент с помощью `driver.switchTo().defaultContent()`.

### Пример работы с фреймами

```java
import io.qameta.allure.Step;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class FramePage {
    private static final Logger logger = LoggerFactory.getLogger(FramePage.class);
    private final WebDriver driver;

    @FindBy(id = "frame-id")
    private WebElement frame;

    @FindBy(id = "element-in-frame")
    private WebElement frameElement;

    public FramePage(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this);
    }

    @Step("Switching to frame by element")
    public void switchToFrame() {
        logger.info("Switching to frame with id: frame-id");
        driver.switchTo().frame(frame);
    }

    @Step("Interacting with element in frame")
    public void interactWithFrameElement() {
        logger.info("Clicking element in frame");
        frameElement.click();
    }

    @Step("Switching back to default content")
    public void switchToDefaultContent() {
        logger.info("Switching back to default content");
        driver.switchTo().defaultContent();
    }
}
```

### Особенности
- **Способы переключения**: Фрейм можно указать по `WebElement`, `id/name` или индексу (`driver.switchTo().frame(0)`).
- **Ожидание фрейма**: Используйте `WebDriverWait` для проверки доступности фрейма.

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.time.Duration;

public class FrameWaitUtil {
    private static final Logger logger = LoggerFactory.getLogger(FrameWaitUtil.class);

    public static void waitForFrameAndSwitch(WebDriver driver, WebElement frameElement, int timeoutInSeconds) {
        logger.info("Waiting for frame to be available");
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(timeoutInSeconds));
        wait.until(ExpectedConditions.frameToBeAvailableAndSwitchToIt(frameElement));
        logger.info("Switched to frame");
    }
}
```

## Управление окнами и вкладками

Selenium позволяет управлять окнами и вкладками через `driver.getWindowHandles()` и `driver.switchTo().window()`.

### Пример управления окнами

```java
import io.qameta.allure.Step;
import org.openqa.selenium.WebDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.Set;

public class WindowHandler {
    private static final Logger logger = LoggerFactory.getLogger(WindowHandler.class);
    private final WebDriver driver;

    public WindowHandler(WebDriver driver) {
        this.driver = driver;
    }

    @Step("Switching to new window")
    public void switchToNewWindow() {
        String originalWindow = driver.getWindowHandle();
        Set<String> allWindows = driver.getWindowHandles();
        for (String windowHandle : allWindows) {
            if (!windowHandle.equals(originalWindow)) {
                driver.switchTo().window(windowHandle);
                logger.info("Switched to new window: {}", windowHandle);
                break;
            }
        }
    }

    @Step("Closing current window and switching back to original")
    public void closeAndSwitchBack(String originalWindow) {
        logger.info("Closing current window");
        driver.close();
        driver.switchTo().window(originalWindow);
        logger.info("Switched back to original window: {}", originalWindow);
    }
}
```

### Особенности
- **Получение дескрипторов**: Метод `getWindowHandles()` возвращает множество всех открытых окон/вкладок.
- **Переключение**: Переключайтесь на нужное окно по его дескриптору.
- **Закрытие окон**: Используйте `driver.close()` для текущего окна и `driver.quit()` для завершения всех окон.

## Пример теста с JUnit 5

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

public class AlertFrameWindowTest {
    private static final Logger logger = LoggerFactory.getLogger(AlertFrameWindowTest.class);
    private WebDriver driver;
    private AlertHandler alertHandler;
    private FramePage framePage;
    private WindowHandler windowHandler;

    @BeforeEach
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        alertHandler = new AlertHandler(driver);
        framePage = new FramePage(driver);
        windowHandler = new WindowHandler(driver);
        logger.info("WebDriver initialized");
    }

    @Test
    @Step("Test handling alert, frame, and new window")
    void testAlertFrameAndWindow() {
        // Arrange
        String originalWindow = driver.getWindowHandle();
        driver.get("https://example.com/test-page");
        logger.info("Navigated to test page");

        // Act: Handle alert
        AlertWaitUtil.waitForAlert(driver, 10);
        alertHandler.acceptAlert();

        // Act: Handle frame
        FrameWaitUtil.waitForFrameAndSwitch(driver, framePage.frame, 10);
        framePage.interactWithFrameElement();
        framePage.switchToDefaultContent();

        // Act: Handle new window
        driver.findElement(By.id("new-window-link")).click();
        windowHandler.switchToNewWindow();
        assertThat(driver.getCurrentUrl()).contains("new-page");
        windowHandler.closeAndSwitchBack(originalWindow);

        // Assert
        assertThat(driver.getCurrentUrl()).contains("test-page");
        logger.info("Test completed successfully");
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

## Пример теста с TestNG

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import io.qameta.allure.Step;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;

import static org.assertj.core.api.Assertions.assertThat;

public class AlertFrameWindowTestNG {
    private static final Logger logger = LoggerFactory.getLogger(AlertFrameWindowTestNG.class);
    private WebDriver driver;
    private AlertHandler alertHandler;
    private FramePage framePage;
    private WindowHandler windowHandler;

    @BeforeMethod
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        alertHandler = new AlertHandler(driver);
        framePage = new FramePage(driver);
        windowHandler = new WindowHandler(driver);
        logger.info("WebDriver initialized");
    }

    @Test
    @Step("Test handling alert, frame, and new window")
    void testAlertFrameAndWindow() {
        // Given
        String originalWindow = driver.getWindowHandle();
        driver.get("https://example.com/test-page");
        logger.info("Navigated to test page");

        // When: Handle alert
        AlertWaitUtil.waitForAlert(driver, 10);
        alertHandler.acceptAlert();

        // When: Handle frame
        FrameWaitUtil.waitForFrameAndSwitch(driver, framePage.frame, 10);
        framePage.interactWithFrameElement();
        framePage.switchToDefaultContent();

        // When: Handle new window
        driver.findElement(By.id("new-window-link")).click();
        windowHandler.switchToNewWindow();
        assertThat(driver.getCurrentUrl()).contains("new-page");
        windowHandler.closeAndSwitchBack(originalWindow);

        // Then
        assertThat(driver.getCurrentUrl()).contains("test-page");
        logger.info("Test completed successfully");
    }

    @AfterMethod
    void tearDown() {
        logger.info("Closing WebDriver");
        if (driver != null) {
            driver.quit();
        }
    }
}
```

## Интеграция с Allure

Для отчётности используйте аннотацию `@Step` (как в примерах выше) и добавьте зависимости в `pom.xml`:

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

## Интеграция с Jenkins

Настройте запуск тестов в Jenkins с поддержкой Allure.

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
   - Установите точки останова в методах обработки алертов, фреймов или окон.
   - Используйте DevTools браузера для проверки `id` или `name` фреймов и ссылок для новых окон.
   - Логируйте действия с помощью SLF4J для отслеживания переключений.

2. **Устойчивость тестов**:
   - Используйте стабильные локаторы (`id`, `data-*`) для элементов фреймов и ссылок.
   - Проверяйте доступность алертов и фреймов с помощью `isDisplayed()` или `ExpectedConditions`.
   - Обрабатывайте исключения, такие как `NoAlertPresentException` или `NoSuchFrameException`:

```java
public void safeAcceptAlert(WebDriver driver) {
    try {
        AlertWaitUtil.waitForAlert(driver, 10);
        alertHandler.acceptAlert();
    } catch (Exception e) {
        logger.warn("No alert present: {}", e.getMessage());
    }
}
```

## Распространённые ошибки

1. **Отсутствие ожидания**: Игнорирование `WebDriverWait` для алертов или фреймов приводит к `NoSuchElementException`.
2. **Неправильное переключение**: Забытое возвращение в `defaultContent()` после работы с фреймом.
3. **Управление окнами**: Ошибки при работе с дескрипторами окон из-за их неправильного сохранения.
4. **Логирование**: Недостаточное логирование шагов, что затрудняет отладку.

## Дополнительно

- **Selenide для упрощения**:
  - Selenide автоматически обрабатывает алерты и фреймы:

```java
import com.codeborne.selenide.Configuration;
import com.codeborne.selenide.Selenide;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import static com.codeborne.selenide.Selenide.*;

public class SelenideExample {
    private static final Logger logger = LoggerFactory.getLogger(SelenideExample.class);

    public static void handleAlertAndFrame() {
        Configuration.browser = "chrome";
        Configuration.timeout = 10000;
        open("https://example.com/test-page");
        logger.info("Navigated to test page");

        // Handle alert
        confirm("Are you sure?");
        logger.info("Alert confirmed");

        // Handle frame
        switchTo().frame("frame-id");
        $("#element-in-frame").click();
        switchTo().defaultContent();
        logger.info("Interacted with frame and switched back");

        // Handle new window
        $("#new-window-link").click();
        switchTo().window(1);
        logger.info("Switched to new window");
        closeWindow();
        switchTo().window(0);
        logger.info("Switched back to original window");
    }
}
```

- **Testcontainers для тестирования**:
  - Используйте Testcontainers для локального запуска браузеров в Docker:

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

public class TestcontainersExample {
    private static final Logger logger = LoggerFactory.getLogger(TestcontainersExample.class);

    public void testWithContainer() {
        try (BrowserWebDriverContainer<?> container = new BrowserWebDriverContainer<>()
                .withCapabilities(new ChromeOptions())) {
            container.start();
            WebDriver driver = container.getWebDriver();
            logger.info("Testcontainers browser started");
            driver.get("https://example.com/test-page");
            // Логика теста
            driver.quit();
            logger.info("Testcontainers browser closed");
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