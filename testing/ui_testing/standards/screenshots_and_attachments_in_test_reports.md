# Скриншоты и вложения в отчёты тестирования

## Захват скриншотов при падении теста

Скриншоты при падении тестов помогают в отладке, предоставляя визуальный контекст ошибки. В UI-тестировании для этого используется интерфейс `TakesScreenshot` из Selenium WebDriver. Скриншоты можно прикреплять к отчётам Allure для анализа.

### Реализация с JUnit 5
```java
import io.qameta.allure.Allure;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

public class ScreenshotTest {
    private WebDriver driver;

    @AfterEach
    void tearDown() {
        if (driver != null) {
            // Захват скриншота при падении теста
            byte[] screenshot = ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
            Allure.addAttachment("Screenshot on failure", "image/png", screenshot, ".png");
            driver.quit();
        }
    }

    @Test
    void testWithScreenshot() {
        driver = new ChromeDriver();
        driver.get("https://example.com");
        // Симуляция ошибки
        throw new AssertionError("Test failed intentionally");
    }
}
```

### Реализация с TestNG
```java
import io.qameta.allure.Allure;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.Test;

public class ScreenshotTest {
    private WebDriver driver;

    @AfterMethod
    public void tearDown() {
        if (driver != null) {
            // Захват скриншота при падении теста
            byte[] screenshot = ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
            Allure.addAttachment("Screenshot on failure", "image/png", screenshot, ".png");
            driver.quit();
        }
    }

    @Test
    public void testWithScreenshot() {
        driver = new ChromeDriver();
        driver.get("https://example.com");
        // Симуляция ошибки
        throw new AssertionError("Test failed intentionally");
    }
}
```

### Лучшие практики
- Используйте `try-catch` в методе `@AfterEach` или `@AfterMethod`, чтобы гарантировать закрытие браузера даже при сбоях.
- Сохраняйте скриншоты только при падении теста, чтобы не перегружать отчёты.
- Убедитесь, что WebDriver корректно инициализирован перед вызовом `TakesScreenshot`.

## Вложение HTML, JSON, логов и curl-запросов

Для анализа ошибок в отчётах полезно прикреплять исходный код страницы, JSON-ответы API, логи и curl-запросы. Allure поддерживает вложения в различных форматах через метод `Allure.addAttachment`.

### Пример вложения исходного кода страницы
```java
import io.qameta.allure.Allure;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

public class PageSourceTest {
    @Test
    void savePageSource() {
        WebDriver driver = new ChromeDriver();
        try {
            driver.get("https://example.com");
            // Сохранение исходного кода страницы
            String pageSource = driver.getPageSource();
            Allure.addAttachment("Page Source", "text/html", pageSource, ".html");
            // Симуляция ошибки
            throw new AssertionError("Test failed");
        } finally {
            driver.quit();
        }
    }
}
```

### Пример вложения JSON-ответа (API-тестирование)
```java
import io.qameta.allure.Allure;
import io.restassured.RestAssured;
import io.restassured.response.Response;
import org.junit.jupiter.api.Test;

public class ApiAttachmentTest {
    @Test
    void attachJsonResponse() {
        Response response = RestAssured.get("https://api.example.com/data");
        // Вложение JSON-ответа
        Allure.addAttachment("API Response", "application/json", response.asString(), ".json");
    }
}
```

### Пример вложения curl-запроса
```java
import io.qameta.allure.Allure;
import io.restassured.RestAssured;
import io.restassured.specification.RequestSpecification;
import org.junit.jupiter.api.Test;

public class CurlAttachmentTest {
    @Test
    void attachCurlRequest() {
        RequestSpecification request = RestAssured.given().baseUri("https://api.example.com");
        // Формирование curl-запроса
        String curl = "curl -X GET 'https://api.example.com/data'";
        Allure.addAttachment("Curl Request", "text/plain", curl, ".txt");
        request.get("/data");
    }
}
```

### Логирование
Для логов используйте SLF4J с Logback. Пример настройки `pom.xml`:
```xml
<dependencies>
    <!-- SLF4J API -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>2.0.16</version>
    </dependency>
    <!-- Logback -->
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.5.8</version>
    </dependency>
    <!-- Allure JUnit 5 -->
    <dependency>
        <groupId>io.qameta.allure</groupId>
        <artifactId>allure-junit5</artifactId>
        <version>2.30.0</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### Пример логирования
```java
import io.qameta.allure.Allure;
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LogAttachmentTest {
    private static final Logger logger = LoggerFactory.getLogger(LogAttachmentTest.class);

    @Test
    void attachLogs() {
        logger.info("Test started");
        // Вложение логов
        Allure.addAttachment("Test Logs", "text/plain", "Test started\nTest step executed", ".txt");
    }
}
```

## Автоматизация вложений в CI/CD пайплайне

Для автоматизации вложений в CI/CD (например, Jenkins или GitHub Actions) настройте Allure для генерации отчётов. Пример конфигурации для Maven и Allure:

### Настройка Allure в `pom.xml`
```xml
<build>
    <plugins>
        <plugin>
            <groupId>io.qameta.allure</groupId>
            <artifactId>allure-maven</artifactId>
            <version>2.12.0</version>
            <configuration>
                <reportVersion>2.30.0</reportVersion>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### Пример GitHub Actions
```yaml
name: CI Pipeline
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          java-version: '17'
      - name: Run tests
        run: mvn clean test
      - name: Generate Allure report
        run: mvn allure:report
      - name: Upload Allure report
        uses: actions/upload-artifact@v4
        with:
          name: allure-report
          path: target/site/allure-maven
```

### Лучшие практики
- Используйте уникальные имена для вложений, чтобы избежать перезаписи.
- Настройте Allure TestOps для централизованного хранения отчётов.
- В CI/CD добавьте шаг для архивирования артефактов (скриншотов, логов).

## Дополнительно

### Отладка
- **IntelliJ IDEA Debug**: Устанавливайте точки останова в местах вызова `TakesScreenshot` или `addAttachment`. Проверяйте содержимое переменных (например, `pageSource` или `response.asString()`) в дебаггере.
- **Локаторы**: Для UI-тестов проверяйте стабильность локаторов перед захватом скриншота с помощью `driver.findElement().isDisplayed()`.
- **Ожидания**: Используйте `WebDriverWait` для стабилизации тестов перед захватом данных.

### Распространённые ошибки
- **Скриншоты пустые**: Убедитесь, что WebDriver инициализирован и страница загружена.
- **Перегрузка отчётов**: Избегайте прикрепления больших файлов (например, видео) без сжатия.
- **Ошибки Allure**: Проверьте наличие зависимости `allure-junit5` или `allure-testng` и их совместимость с версией Allure.

## Источники
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/)
- [Allure Framework](https://allurereport.org/docs/)
- [RestAssured Documentation](http://rest-assured.io/)

---
[**← Назад к оглавлению**](../README.md)