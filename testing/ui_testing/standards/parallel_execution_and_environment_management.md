# Параллельный запуск и управление окружением

## Параллельный запуск тестов

Параллельный запуск тестов сокращает время выполнения тестовой сборки. Используются TestNG, JUnit 5, Selenide Grid и Selenoid для организации параллельного выполнения UI-тестов.

### Параллельный запуск с TestNG
TestNG поддерживает параллельное выполнение через конфигурацию `testng.xml`.

#### Пример `testng.xml`
```xml
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="ParallelSuite" parallel="tests" thread-count="2">
    <test name="Test1">
        <classes>
            <class name="com.example.ParallelTest"/>
        </classes>
    </test>
    <test name="Test2">
        <classes>
            <class name="com.example.ParallelTest"/>
        </classes>
    </test>
</suite>
```

#### Пример теста
```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.Test;

public class ParallelTest {
    private WebDriver driver;

    @Test
    public void testExample() {
        driver = new ChromeDriver();
        driver.get("https://example.com");
        // Логика теста
    }

    @AfterMethod
    public void tearDown() {
        if (driver != null) {
            driver.quit();
        }
    }
}
```

### Параллельный запуск с JUnit 5
JUnit 5 поддерживает параллельное выполнение через конфигурацию `junit-platform.properties`.

#### Пример `junit-platform.properties`
```properties
junit.jupiter.execution.parallel.enabled=true
junit.jupiter.execution.parallel.mode.default=concurrent
junit.jupiter.execution.parallel.config.strategy=fixed
junit.jupiter.execution.parallel.config.fixed.parallelism=2
```

#### Пример теста
```java
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

public class ParallelTest {
    private WebDriver driver;

    @Test
    void testExample() {
        driver = new ChromeDriver();
        driver.get("https://example.com");
        // Логика теста
    }

    @AfterEach
    void tearDown() {
        if (driver != null) {
            driver.quit();
        }
    }
}
```

### Параллельный запуск с Selenide и Selenoid
Selenide упрощает работу с Selenium, а Selenoid обеспечивает масштабируемый запуск тестов в Docker-контейнерах.

#### Настройка Selenoid
1. Установите Selenoid: `docker pull aerokube/selenoid`.
2. Настройте `browsers.json`:
```json
{
  "chrome": {
    "default": "latest",
    "versions": {
      "latest": {
        "image": "selenoid/chrome:latest",
        "port": "4444"
      }
    }
  }
}
```
3. Запустите Selenoid: `docker run -d -p 4444:4444 -v /path/to/browsers.json:/etc/selenoid/browsers.json aerokube/selenoid`.

#### Пример теста с Selenide
```java
import com.codeborne.selenide.Configuration;
import com.codeborne.selenide.Selenide;
import org.junit.jupiter.api.Test;

import static com.codeborne.selenide.Selenide.open;

public class SelenideParallelTest {
    static {
        Configuration.remote = "http://localhost:4444/wd/hub";
        Configuration.browser = "chrome";
    }

    @Test
    void testWithSelenoid() {
        open("https://example.com");
        // Логика теста
    }
}
```

### Лучшие практики
- Ограничивайте количество параллельных потоков в зависимости от ресурсов CI/CD.
- Используйте Selenoid для масштабирования UI-тестов в контейнерах.
- Настройте таймауты (`Configuration.timeout` в Selenide) для предотвращения зависаний.

## Управление браузерами и конфигурацией

Для управления браузерами используются параметры WebDriver: headless-режим, удалённый запуск и настройка разрешения экрана.

### Пример конфигурации (Chrome)
```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.junit.jupiter.api.Test;

public class BrowserConfigTest {
    @Test
    void configureBrowser() {
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--headless"); // Headless-режим
        options.addArguments("--window-size=1920,1080"); // Разрешение
        WebDriver driver = new ChromeDriver(options);
        driver.get("https://example.com");
        driver.quit();
    }
}
```

### Удалённый запуск
```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.remote.RemoteWebDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.junit.jupiter.api.Test;
import java.net.URL;

public class RemoteBrowserTest {
    @Test
    void remoteBrowser() throws Exception {
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--headless");
        WebDriver driver = new RemoteWebDriver(new URL("http://localhost:4444/wd/hub"), options);
        driver.get("https://example.com");
        driver.quit();
    }
}
```

### Selenide для упрощения конфигурации
```java
import com.codeborne.selenide.Configuration;
import org.junit.jupiter.api.Test;

import static com.codeborne.selenide.Selenide.open;

public class SelenideConfigTest {
    static {
        Configuration.browser = "chrome";
        Configuration.headless = true;
        Configuration.browserSize = "1920x1080";
        Configuration.remote = "http://localhost:4444/wd/hub";
    }

    @Test
    void testWithConfig() {
        open("https://example.com");
        // Логика теста
    }
}
```

### Лучшие практики
- Используйте headless-режим для CI/CD, чтобы снизить нагрузку на ресурсы.
- Настраивайте разрешение экрана для соответствия целевому UI.
- Проверяйте стабильность локаторов с помощью `isDisplayed()` перед взаимодействием.

## Изоляция тестов

Изоляция тестов предотвращает взаимное влияние тестов через уникальные данные и очистку состояния.

### Уникальные данные
Используйте генерацию уникальных данных (например, UUID) для тестовых пользователей или записей.

#### Пример
```java
import org.junit.jupiter.api.Test;
import java.util.UUID;

public class UniqueDataTest {
    @Test
    void testWithUniqueData() {
        String uniqueEmail = "user_" + UUID.randomUUID() + "@example.com";
        // Использование uniqueEmail в тесте
    }
}
```

### Очистка после тестов
Очищайте данные после теста через API или базу данных.

#### Пример очистки
```java
import io.restassured.RestAssured;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.Test;

public class CleanupTest {
    private String userId;

    @Test
    void testWithCleanup() {
        userId = RestAssured.given()
                .body("{\"email\": \"test@example.com\"}")
                .post("https://api.example.com/users")
                .jsonPath().getString("id");
        // Логика теста
    }

    @AfterEach
    void cleanup() {
        if (userId != null) {
            RestAssured.delete("https://api.example.com/users/" + userId);
        }
    }
}
```

### Лучшие практики
- Генерируйте уникальные данные для каждого теста (например, email, ID).
- Используйте `@AfterEach` (JUnit 5) или `@AfterMethod` (TestNG) для очистки.
- Избегайте прямых запросов к базе данных, если доступен API.

## Поддержка разных окружений

Для поддержки окружений (dev, qa, prod) используйте конфигурационные файлы или системные переменные.

### Пример конфигурации Maven
```xml
<profiles>
    <profile>
        <id>dev</id>
        <properties>
            <env.url>https://dev.example.com</env.url>
        </properties>
    </profile>
    <profile>
        <id>qa</id>
        <properties>
            <env.url>https://qa.example.com</env.url>
        </properties>
    </profile>
    <profile>
        <id>prod</id>
        <properties>
            <env.url>https://prod.example.com</env.url>
        </properties>
    </profile>
</profiles>
```

### Пример теста с динамическим окружением
```java
import com.codeborne.selenide.Configuration;
import org.junit.jupiter.api.Test;

import static com.codeborne.selenide.Selenide.open;

public class EnvironmentTest {
    static {
        String envUrl = System.getProperty("env.url", "https://qa.example.com");
        Configuration.baseUrl = envUrl;
    }

    @Test
    void testOnEnvironment() {
        open("/");
        // Логика теста
    }
}
```

### Запуск в CI/CD
#### GitHub Actions
```yaml
name: CI Pipeline
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        env: [dev, qa, prod]
    steps:
      - uses: actions/checkout@v4
      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          java-version: '17'
      - name: Run tests
        run: mvn test -P${{ matrix.env }}
```

### Лучшие практики
- Используйте системные переменные или Maven-профили для управления окружениями.
- Настройте разные конфигурации для каждого окружения (URL, API-ключи).
- Проверяйте доступность окружения перед запуском тестов.

## Дополнительно

### Отладка
- **IntelliJ IDEA Debug**: Устанавливайте точки останова для проверки конфигурации WebDriver или Selenoid.
- **Локаторы**: Используйте стабильные локаторы (CSS, XPath) и проверяйте их с помощью `isDisplayed()`.
- **Ожидания**: Применяйте `WebDriverWait` для синхронизации тестов.

### Распространённые ошибки
- **Конфликты данных**: Убедитесь, что тесты используют уникальные данные.
- **Перегрузка Selenoid**: Ограничивайте количество контейнеров в зависимости от ресурсов.
- **Ошибки конфигурации**: Проверяйте правильность URL и параметров окружения.

## Источники
- [TestNG Documentation](https://testng.org/doc/)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [Selenoid Documentation](https://aerokube.com/selenoid/latest/)
- [Selenide Documentation](https://selenide.org/documentation.html)

---
[**← Назад к оглавлению**](../README.md)