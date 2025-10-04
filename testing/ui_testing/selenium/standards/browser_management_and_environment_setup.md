# Управление браузером и настройка окружения

Эффективное управление браузером и настройка окружения в автоматизации тестирования UI обеспечивают стабильность тестов и их переносимость между различными конфигурациями. В статье описаны подходы к поддержке разных браузеров (Chrome, Firefox, Edge), использованию WebDriverManager и Docker/Selenoid, а также настройке разрешений экрана, таймаутов, прокси и языков.

## Поддержка разных браузеров

Для обеспечения кросс-браузерного тестирования используется Selenium WebDriver с поддержкой Chrome, Firefox и Edge. Выбор браузера задаётся через конфигурацию или параметры.

### Использование WebDriverManager

**WebDriverManager** (Bonigarcia) упрощает управление драйверами браузеров, автоматически загружая и настраивая их.

#### Зависимости в Maven

Добавьте зависимость в `pom.xml`:

```xml
<dependencies>
    <dependency>
        <groupId>io.github.bonigarcia</groupId>
        <artifactId>webdrivermanager</artifactId>
        <version>5.9.2</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.seleniumhq.selenium</groupId>
        <artifactId>selenium-java</artifactId>
        <version>4.26.0</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

#### Пример настройки браузеров

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.firefox.FirefoxDriver;
import org.openqa.selenium.firefox.FirefoxOptions;
import org.openqa.selenium.edge.EdgeDriver;
import org.openqa.selenium.edge.EdgeOptions;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class BrowserFactory {
    private static final Logger logger = LoggerFactory.getLogger(BrowserFactory.class);

    public static WebDriver createDriver(String browser) {
        WebDriver driver;
        switch (browser.toLowerCase()) {
            case "chrome":
                WebDriverManager.chromedriver().setup();
                ChromeOptions chromeOptions = new ChromeOptions();
                chromeOptions.addArguments("--headless", "--disable-gpu");
                driver = new ChromeDriver(chromeOptions);
                logger.info("Chrome browser initialized");
                break;
            case "firefox":
                WebDriverManager.firefoxdriver().setup();
                FirefoxOptions firefoxOptions = new FirefoxOptions();
                firefoxOptions.addArguments("--headless");
                driver = new FirefoxDriver(firefoxOptions);
                logger.info("Firefox browser initialized");
                break;
            case "edge":
                WebDriverManager.edgedriver().setup();
                EdgeOptions edgeOptions = new EdgeOptions();
                edgeOptions.addArguments("--headless");
                driver = new EdgeDriver(edgeOptions);
                logger.info("Edge browser initialized");
                break;
            default:
                throw new IllegalArgumentException("Unsupported browser: " + browser);
        }
        return driver;
    }
}
```

### Использование Docker/Selenoid

**Selenoid** — это сервер Selenium, работающий в Docker, который упрощает масштабируемое тестирование в разных браузерах.

#### Настройка Selenoid

1. Установите Docker.
2. Скачайте конфигурацию Selenoid:

```bash
curl -s https://raw.githubusercontent.com/aerokube/selenoid/master/config/browsers.json > browsers.json
```

Пример `browsers.json`:

```json
{
  "chrome": {
    "default": "126.0",
    "versions": {
      "126.0": {
        "image": "selenoid/chrome:126.0",
        "port": "4444"
      }
    }
  },
  "firefox": {
    "default": "130.0",
    "versions": {
      "130.0": {
        "image": "selenoid/firefox:130.0",
        "port": "4444"
      }
    }
  },
  "edge": {
    "default": "126.0",
    "versions": {
      "126.0": {
        "image": "selenoid/edge:126.0",
        "port": "4444"
      }
    }
  }
}
```

3. Запустите Selenoid:

```bash
docker run -d --name selenoid -p 4444:4444 -v $(pwd)/browsers.json:/etc/selenoid/browsers.json aerokube/selenoid:latest-release
```

#### Пример подключения к Selenoid

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.remote.DesiredCapabilities;
import org.openqa.selenium.remote.RemoteWebDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.net.URL;

public class SelenoidFactory {
    private static final Logger logger = LoggerFactory.getLogger(SelenoidFactory.class);

    public static WebDriver createDriver(String browser) {
        try {
            DesiredCapabilities capabilities = new DesiredCapabilities();
            capabilities.setBrowserName(browser);
            capabilities.setVersion("126.0");
            capabilities.setCapability("enableVNC", true);
            capabilities.setCapability("enableVideo", false);
            URL selenoidUrl = new URL("http://localhost:4444/wd/hub");
            WebDriver driver = new RemoteWebDriver(selenoidUrl, capabilities);
            logger.info("Selenoid initialized with browser: {}", browser);
            return driver;
        } catch (Exception e) {
            logger.error("Failed to initialize Selenoid: {}", e.getMessage());
            throw new RuntimeException("Selenoid initialization failed", e);
        }
    }
}
```

## Настройка окружения

Настройка окружения включает управление разрешением экрана, таймаутами, прокси и языками браузера.

### Настройка разрешения экрана

Разрешение задаётся через `WebDriver` или опции браузера.

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class ResolutionSetup {
    private static final Logger logger = LoggerFactory.getLogger(ResolutionSetup.class);

    public static void setResolution(WebDriver driver, int width, int height) {
        logger.info("Setting browser resolution to {}x{}", width, height);
        driver.manage().window().setSize(new org.openqa.selenium.Dimension(width, height));
    }

    public static WebDriver createChromeWithResolution() {
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--window-size=1920,1080");
        WebDriver driver = new ChromeDriver(options);
        logger.info("Chrome initialized with resolution 1920x1080");
        return driver;
    }
}
```

### Настройка таймаутов

Для устойчивости тестов задаются таймауты для загрузки страниц, выполнения скриптов и неявных ожиданий.

```java
import org.openqa.selenium.WebDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.time.Duration;

public class TimeoutSetup {
    private static final Logger logger = LoggerFactory.getLogger(TimeoutSetup.class);

    public static void configureTimeouts(WebDriver driver) {
        driver.manage().timeouts()
                .implicitlyWait(Duration.ofSeconds(10))
                .pageLoadTimeout(Duration.ofSeconds(30))
                .scriptTimeout(Duration.ofSeconds(5));
        logger.info("Timeouts configured: implicit=10s, pageLoad=30s, script=5s");
    }
}
```

Для явных ожиданий используйте `WebDriverWait`:

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.time.Duration;

public class WaitUtil {
    private static final Logger logger = LoggerFactory.getLogger(WaitUtil.class);

    public static void waitForElementVisible(WebDriver driver, WebElement element) {
        logger.info("Waiting for element to be visible");
        new WebDriverWait(driver, Duration.ofSeconds(10))
                .until(d -> element.isDisplayed());
        logger.info("Element is visible");
    }
}
```

### Настройка прокси

Прокси настраивается через опции браузера.

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class ProxySetup {
    private static final Logger logger = LoggerFactory.getLogger(ProxySetup.class);

    public static WebDriver createChromeWithProxy(String proxyServer) {
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--proxy-server=" + proxyServer);
        WebDriver driver = new ChromeDriver(options);
        logger.info("Chrome initialized with proxy: {}", proxyServer);
        return driver;
    }
}
```

Пример использования: `createChromeWithProxy("http://proxy.example.com:8080")`.

### Настройка языка браузера

Язык браузера задаётся через аргументы или профили.

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LanguageSetup {
    private static final Logger logger = LoggerFactory.getLogger(LanguageSetup.class);

    public static WebDriver createChromeWithLanguage(String language) {
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--lang=" + language);
        WebDriver driver = new ChromeDriver(options);
        logger.info("Chrome initialized with language: {}", language);
        return driver;
    }
}
```

Пример: `createChromeWithLanguage("ru-RU")` для русского языка.

## Пример теста с настройкой окружения

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import io.qameta.allure.Step;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import static org.assertj.core.api.Assertions.assertThat;

public class BrowserTest {
    private static final Logger logger = LoggerFactory.getLogger(BrowserTest.class);
    private WebDriver driver;

    @BeforeEach
    void setUp() {
        WebDriverManager.chromedriver().setup();
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--headless", "--lang=ru-RU", "--window-size=1920,1080");
        driver = BrowserFactory.createDriver("chrome");
        TimeoutSetup.configureTimeouts(driver);
        logger.info("Test environment set up");
    }

    @Test
    @Step("Test navigation to example page")
    void testNavigation() {
        // Arrange
        String url = "https://example.com";

        // Act
        driver.get(url);
        logger.info("Navigated to {}", url);

        // Assert
        assertThat(driver.getTitle()).contains("Example");
        logger.info("Page title verified");
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

## Интеграция с CI/CD

Для запуска тестов в Jenkins или GitHub Actions настройте конфигурацию Maven и Selenoid.

### Пример Jenkins Pipeline

```groovy
pipeline {
    agent {
        docker {
            image 'maven:3.9.9-eclipse-temurin-17'
            args '-v $HOME/.m2:/root/.m2'
        }
    }
    environment {
        SELENOID_URL = 'http://selenoid:4444/wd/hub'
    }
    stages {
        stage('Test') {
            steps {
                sh 'mvn clean test -Dbrowser=chrome'
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

### Пример GitHub Actions Workflow

```yaml
name: UI Tests
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      selenoid:
        image: aerokube/selenoid:latest-release
        ports:
          - 4444:4444
    steps:
      - uses: actions/checkout@v3
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
      - name: Run tests
        env:
          BROWSER: chrome
        run: mvn clean test
      - name: Publish Allure Report
        uses: actions/upload-artifact@v3
        with:
          name: allure-report
          path: target/allure-results
```

## Отладка и устойчивость

1. **Отладка в IntelliJ IDEA**:
   - Установите точки останова в `BrowserFactory` или тестах.
   - Проверяйте локаторы через DevTools браузера.
   - Используйте логи SLF4J для отслеживания инициализации драйвера и настроек.

2. **Устойчивость тестов**:
   - Используйте стабильные локаторы (`id`, `data-*`).
   - Проверяйте элементы через `isDisplayed()` и `isEnabled()` перед действиями.
   - Пример:

```java
import org.openqa.selenium.WebElement;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class ElementUtil {
    private static final Logger logger = LoggerFactory.getLogger(ElementUtil.class);

    public static void safeClick(WebElement element) {
        if (element.isDisplayed() && element.isEnabled()) {
            logger.info("Clicking element");
            element.click();
        } else {
            logger.error("Element is not clickable");
            throw new IllegalStateException("Element is not clickable");
        }
    }
}
```

## Распространённые ошибки

1. **Неправильные версии драйверов**: WebDriverManager устраняет эту проблему, но при использовании Selenoid проверяйте соответствие версий браузеров.
2. **Некорректные таймауты**: Слишком короткие таймауты приводят к ошибкам. Используйте разумные значения (10–30 секунд).
3. **Ошибки прокси**: Убедитесь, что прокси-сервер доступен и правильно настроен.
4. **Игнорирование headless-режима**: В CI/CD включайте `--headless` для экономии ресурсов.

## Дополнительно

- **Selenide для упрощения работы**:
  - Selenide автоматически управляет драйверами и ожиданиями:

```xml
<dependencies>
    <dependency>
        <groupId>com.codeborne</groupId>
        <artifactId>selenide</artifactId>
        <version>7.5.0</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

```java
import com.codeborne.selenide.Configuration;
import com.codeborne.selenide.Selenide;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import static com.codeborne.selenide.Selenide.$;

public class SelenideExample {
    private static final Logger logger = LoggerFactory.getLogger(SelenideExample.class);

    public static void configureSelenide() {
        Configuration.browser = "chrome";
        Configuration.headless = true;
        Configuration.browserSize = "1920x1080";
        Configuration.timeout = 10000;
        logger.info("Selenide configured with browser: chrome, headless: true");
    }

    public static void login(String username, String password) {
        configureSelenide();
        Selenide.open("https://example.com/login");
        $("#username").setValue(username);
        $("#password").setValue(password);
        $("#loginButton").click();
        logger.info("Login attempt completed");
    }
}
```

- **Allure для отчётов**:
  - Добавьте шаги с помощью `@Step` для отслеживания настроек браузера.

## Источники
- [Selenium WebDriver](https://www.selenium.dev/documentation/)
- [WebDriverManager](https://bonigarcia.dev/webdrivermanager/)
- [Selenoid Documentation](https://aerokube.com/selenoid/latest/)
- [Selenide Documentation](https://selenide.org/)

---
[**← Назад к оглавлению**](../README.md)