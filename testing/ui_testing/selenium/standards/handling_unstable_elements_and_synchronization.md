# Обработка нестабильных элементов и синхронизация

Нестабильные элементы в веб-страницах возникают из-за динамического обновления DOM, асинхронной загрузки контента или изменений в структуре страницы. Это приводит к ошибкам, таким как StaleElementReferenceException, и требует механизмов синхронизации для обеспечения стабильности тестов. В статье рассмотрены проверки видимости элементов, повторные попытки при исключениях и кастомные ожидания на базе WebDriverWait.

## Проверка видимости, кликабельности и наличия в DOM

Проверка видимости элемента осуществляется с помощью метода `isDisplayed()`, который возвращает `true`, если элемент видим пользователю (учитывая CSS-свойства visibility, display и opacity). Метод `isEnabled()` проверяет, доступен ли элемент для взаимодействия (не отключён). Для наличия элемента в DOM достаточно успешного поиска, но для надёжности комбинируют с ожиданием и проверками.

В Page Object Model эти методы интегрируются в страницы для верификации состояний перед действиями.

### Пример кода (JUnit 5)

```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.openqa.selenium.support.ui.ExpectedConditions;
import io.github.bonigarcia.wdm.WebDriverManager;
import static org.assertj.core.api.Assertions.assertThat;

import java.time.Duration;

public class ElementVisibilityTest {
    private WebDriver driver;
    private WebDriverWait wait;

    @FindBy(id = "submit-button")
    private WebElement submitButton;

    @BeforeEach
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        driver.get("https://example.com/login");
        PageFactory.initElements(driver, this);
    }

    @Test
    void checkElementVisibility() {
        // Arrange: Ожидание загрузки элемента
        wait.until(ExpectedConditions.visibilityOf(submitButton));

        // Act: Проверка видимости и кликабельности
        boolean isVisible = submitButton.isDisplayed();
        boolean isEnabled = submitButton.isEnabled();

        // Assert: Верификация
        assertThat(isVisible).isTrue();
        assertThat(isEnabled).isTrue();
    }
}
```

### Пример кода (TestNG)

```java
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.openqa.selenium.support.ui.ExpectedConditions;
import io.github.bonigarcia.wdm.WebDriverManager;
import static org.assertj.core.api.Assertions.assertThat;

import java.time.Duration;

public class ElementVisibilityTestNG {
    private WebDriver driver;
    private WebDriverWait wait;

    @FindBy(id = "submit-button")
    private WebElement submitButton;

    @BeforeMethod
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        driver.get("https://example.com/login");
        PageFactory.initElements(driver, this);
    }

    @Test
    void checkElementVisibility() {
        // Arrange: Ожидание загрузки элемента
        wait.until(ExpectedConditions.visibilityOf(submitButton));

        // Act: Проверка видимости и кликабельности
        boolean isVisible = submitButton.isDisplayed();
        boolean isEnabled = submitButton.isEnabled();

        // Assert: Верификация
        assertThat(isVisible).isTrue();
        assertThat(isEnabled).isTrue();
    }
}
```

Для стабильности рекомендуется использовать локаторы с `id` или `data-*` атрибутами, чтобы минимизировать влияние на изменения в DOM.

## Повторные попытки при ошибках StaleElementReferenceException

StaleElementReferenceException возникает, когда ссылка на элемент устаревает из-за обновления DOM. Для обработки применяют повторные попытки (retry) с перепоискм элемента в цикле или с использованием FluentWait. Это предотвращает сбои в тестах на динамических страницах.

В архитектуре тестов retry логика интегрируется в утилитные классы или обёртки над действиями WebDriver.

### Пример кода (JUnit 5)

```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.StaleElementReferenceException;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import io.github.bonigarcia.wdm.WebDriverManager;
import static org.assertj.core.api.Assertions.assertThat;

public class StaleElementRetryTest {
    private WebDriver driver;

    @FindBy(id = "dynamic-button")
    private WebElement dynamicButton;

    @BeforeEach
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        driver.get("https://example.com/dynamic");
        PageFactory.initElements(driver, this);
    }

    @Test
    void retryOnStaleElement() {
        // Arrange: Переменная для попыток
        int maxAttempts = 3;
        boolean success = false;

        // Act: Цикл повторных попыток
        for (int attempt = 1; attempt <= maxAttempts; attempt++) {
            try {
                // Перепоиск элемента
                PageFactory.initElements(driver, this);
                dynamicButton.click();
                success = true;
                break;
            } catch (StaleElementReferenceException e) {
                if (attempt == maxAttempts) {
                    throw e; // Переброс исключения после всех попыток
                }
            }
        }

        // Assert: Проверка успеха
        assertThat(success).isTrue();
    }
}
```

### Пример кода (TestNG)

```java
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import org.openqa.selenium.StaleElementReferenceException;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import io.github.bonigarcia.wdm.WebDriverManager;
import static org.assertj.core.api.Assertions.assertThat;

public class StaleElementRetryTestNG {
    private WebDriver driver;

    @FindBy(id = "dynamic-button")
    private WebElement dynamicButton;

    @BeforeMethod
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        driver.get("https://example.com/dynamic");
        PageFactory.initElements(driver, this);
    }

    @Test
    void retryOnStaleElement() {
        // Arrange: Переменная для попыток
        int maxAttempts = 3;
        boolean success = false;

        // Act: Цикл повторных попыток
        for (int attempt = 1; attempt <= maxAttempts; attempt++) {
            try {
                // Перепоиск элемента
                PageFactory.initElements(driver, this);
                dynamicButton.click();
                success = true;
                break;
            } catch (StaleElementReferenceException e) {
                if (attempt == maxAttempts) {
                    throw e; // Переброс исключения после всех попыток
                }
            }
        }

        // Assert: Проверка успеха
        assertThat(success).isTrue();
    }
}
```

Распространённая ошибка — отсутствие retry в динамических сценариях, что приводит к flaky тестам. Для отладки используйте точки останова в IntelliJ IDEA, чтобы проверить состояние элемента после обновления DOM.

## Использование кастомных ожиданий и обёрток над WebDriverWait

WebDriverWait позволяет создавать кастомные ExpectedConditions для специфических сценариев, таких как ожидание нестабильного элемента. Обёртки упрощают повторное использование, инкапсулируя логику в утилитные классы.

Это улучшает читаемость и поддерживаемость тестов, особенно в клиент-серверной модели, где UI зависит от API-ответов.

### Пример кода (JUnit 5)

```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.openqa.selenium.support.ui.ExpectedCondition;
import org.openqa.selenium.support.ui.FluentWait;
import io.github.bonigarcia.wdm.WebDriverManager;
import static org.assertj.core.api.Assertions.assertThat;

import java.time.Duration;

public class CustomWaitTest {
    private WebDriver driver;
    private WebDriverWait wait;

    @BeforeEach
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        driver.get("https://example.com/unstable");
    }

    @Test
    void customExpectation() {
        // Arrange: Кастомное ожидание (элемент стабилен и видим)
        ExpectedCondition<Boolean> stableAndVisible = element -> {
            try {
                WebElement el = driver.findElementById("unstable-element");
                return el.isDisplayed() && el.isEnabled();
            } catch (Exception e) {
                return false;
            }
        };

        // Act: Ожидание
        boolean result = wait.until(stableAndVisible);

        // Assert
        assertThat(result).isTrue();
    }

    @Test
    void fluentWaitWrapper() {
        // Arrange: FluentWait с игнором StaleElement
        FluentWait<WebDriver> fluentWait = new FluentWait<>(driver)
                .withTimeout(Duration.ofSeconds(10))
                .pollingEvery(Duration.ofMillis(500))
                .ignoring(StaleElementReferenceException.class);

        // Act: Ожидание кликабельности
        WebElement element = fluentWait.until(driver -> driver.findElementById("retry-element"));

        // Assert
        assertThat(element.isDisplayed()).isTrue();
    }
}
```

### Пример кода (TestNG)

```java
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.openqa.selenium.support.ui.ExpectedCondition;
import org.openqa.selenium.support.ui.FluentWait;
import org.openqa.selenium.StaleElementReferenceException;
import io.github.bonigarcia.wdm.WebDriverManager;
import static org.assertj.core.api.Assertions.assertThat;

import java.time.Duration;

public class CustomWaitTestNG {
    private WebDriver driver;
    private WebDriverWait wait;

    @BeforeMethod
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        driver.get("https://example.com/unstable");
    }

    @Test
    void customExpectation() {
        // Arrange: Кастомное ожидание (элемент стабилен и видим)
        ExpectedCondition<Boolean> stableAndVisible = element -> {
            try {
                WebElement el = driver.findElementById("unstable-element");
                return el.isDisplayed() && el.isEnabled();
            } catch (Exception e) {
                return false;
            }
        };

        // Act: Ожидание
        boolean result = wait.until(stableAndVisible);

        // Assert
        assertThat(result).isTrue();
    }

    @Test
    void fluentWaitWrapper() {
        // Arrange: FluentWait с игнором StaleElement
        FluentWait<WebDriver> fluentWait = new FluentWait<>(driver)
                .withTimeout(Duration.ofSeconds(10))
                .pollingEvery(Duration.ofMillis(500))
                .ignoring(StaleElementReferenceException.class);

        // Act: Ожидание кликабельности
        WebElement element = fluentWait.until(driver -> driver.findElementById("retry-element"));

        // Assert
        assertThat(element.isDisplayed()).isTrue();
    }
}
```

Лучшая практика — комбинировать с Selenide для упрощения, но базовая реализация на WebDriver обеспечивает контроль. В CI/CD, например, в Jenkins, такие ожидания снижают количество ложных сбоев.

## Источники
- [Устраняем ошибку stale element reference в Selenium](https://sky.pro/wiki/java/ustranyaem-oshibku-stale-element-reference-v-selenium/)
- [Как побороть Stale Element Reference Exception при E2E](https://habr.com/ru/companies/bimeister/articles/698652/)
- [StaleElementReferenceException в Selenium: как исправить](https://external.software/archives/19154)
- [Контроль за ходом теста. Кастомные ожидания](https://comaqa.gitbook.io/selenium-webdriver-lectures/selenium-webdriver.-slozhnye-voprosy./kontrol-za-hodom-testa.-kastomnye-ozhidaniya-popapy-alerty-iframes.)
- [Способы ожидания в Java и Selenium](https://tproger.ru/articles/sposoby-ozhidanija-v-java-i-selenium)
- [How does Selenium isDisplayed() method work?](https://www.browserstack.com/guide/isdisplayed-method-in-selenium)
- [isDisplayed, isEnabled, isSelected Methods in Selenium](https://www.scientecheasy.com/2020/08/selenium-isdisplayed-isenabled-isselected-methods.html/)

---
[**← Назад к оглавлению**](../README.md)