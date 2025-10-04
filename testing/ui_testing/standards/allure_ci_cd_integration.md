# Интеграция с Allure и CI/CD

## Добавление шагов в тесты с Allure

Allure предоставляет возможность структурировать тесты через шаги, используя аннотацию `@Step` или метод `Allure.step()`. Это улучшает читаемость отчётов, позволяя разбивать тест на логические блоки.

### Пример с `@Step` (JUnit 5)
```java
import io.qameta.allure.Step;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

public class AllureStepTest {
    private WebDriver driver;

    @Test
    void testWithSteps() {
        driver = new ChromeDriver();
        try {
            openPage("https://example.com");
            verifyPageTitle("Example Domain");
        } finally {
            driver.quit();
        }
    }

    @Step("Открытие страницы {url}")
    void openPage(String url) {
        driver.get(url);
    }

    @Step("Проверка заголовка страницы: {expectedTitle}")
    void verifyPageTitle(String expectedTitle) {
        String actualTitle = driver.getTitle();
        assert actualTitle.equals(expectedTitle) : "Title mismatch";
    }
}
```

### Пример с `Allure.step` (TestNG)
```java
import io.qameta.allure.Allure;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.Test;

public class AllureStepTest {
    private WebDriver driver;

    @Test
    public void testWithSteps() {
        driver = new ChromeDriver();
        try {
            Allure.step("Открытие страницы https://example.com", () -> {
                driver.get("https://example.com");
            });
            Allure.step("Проверка заголовка страницы", () -> {
                String actualTitle = driver.getTitle();
                assert actualTitle.equals("Example Domain") : "Title mismatch";
            });
        } finally {
            driver.quit();
        }
    }
}
```

### Лучшие практики
- Используйте `@Step` для методов, которые повторно используются в тестах.
- Применяйте `Allure.step()` для динамических шагов внутри теста.
- Описывайте шаги кратко, но информативно, включая входные данные и ожидаемые результаты.

## Скриншоты, логи и вложения при падении теста

Allure позволяет прикреплять скриншоты, логи и другие артефакты для анализа ошибок. Это особенно полезно при интеграции с CI/CD, где отчёты хранятся централизованно.

### Пример с JUnit 5
```java
import io.qameta.allure.Allure;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class AllureAttachmentTest {
    private static final Logger logger = LoggerFactory.getLogger(AllureAttachmentTest.class);
    private WebDriver driver;

    @AfterEach
    void tearDown() {
        if (driver != null) {
            Allure.step("Захват скриншота при падении", () -> {
                byte[] screenshot = ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
                Allure.addAttachment("Screenshot", "image/png", screenshot, ".png");
            });
            Allure.step("Сохранение исходного кода страницы", () -> {
                String pageSource = driver.getPageSource();
                Allure.addAttachment("Page Source", "text/html", pageSource, ".html");
            });
            Allure.addAttachment("Логи теста", "text/plain", "Test failed\nCheck logs for details", ".txt");
            driver.quit();
        }
    }

    @Test
    void testWithAttachments() {
        driver = new ChromeDriver();
        logger.info("Test started");
        driver.get("https://example.com");
        throw new AssertionError("Test failed intentionally");
    }
}
```

### Пример с TestNG
```java
import io.qameta.allure.Allure;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.Test;

public class AllureAttachmentTest {
    private static final Logger logger = LoggerFactory.getLogger(AllureAttachmentTest.class);
    private WebDriver driver;

    @AfterMethod
    public void tearDown() {
        if (driver != null) {
            Allure.step("Захват скриншота при падении", () -> {
                byte[] screenshot = ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
                Allure.addAttachment("Screenshot", "image/png", screenshot, ".png");
            });
            Allure.step("Сохранение исходного кода страницы", () -> {
                String pageSource = driver.getPageSource();
                Allure.addAttachment("Page Source", "text/html", pageSource, ".html");
            });
            Allure.addAttachment("Логи теста", "text/plain", "Test failed\nCheck logs for details", ".txt");
            driver.quit();
        }
    }

    @Test
    public void testWithAttachments() {
        driver = new ChromeDriver();
        logger.info("Test started");
        driver.get("https://example.com");
        throw new AssertionError("Test failed intentionally");
    }
}
```

### Настройка логирования (`pom.xml`)
```xml
<dependencies>
    <!-- Allure JUnit 5 -->
    <dependency>
        <groupId>io.qameta.allure</groupId>
        <artifactId>allure-junit5</artifactId>
        <version>2.30.0</version>
        <scope>test</scope>
    </dependency>
    <!-- Allure TestNG -->
    <dependency>
        <groupId>io.qameta.allure</groupId>
        <artifactId>allure-testng</artifactId>
        <version>2.30.0</version>
        <scope>test</scope>
    </dependency>
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
</dependencies>
```

### Лучшие практики
- Прикрепляйте скриншоты только при падении теста, чтобы не перегружать отчёты.
- Используйте `WebDriverWait` перед захватом скриншота для стабилизации страницы.
- Логируйте ключевые действия с помощью SLF4J для последующего анализа.

## Отображение входных данных и ожидаемых результатов

Allure позволяет отображать входные данные и ожидаемые результаты через аннотации `@Description`, `@Step` или метод `Allure.parameter`.

### Пример с JUnit 5
```java
import io.qameta.allure.Allure;
import io.qameta.allure.Description;
import io.qameta.allure.Step;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

public class AllureParametersTest {
    private WebDriver driver;

    @Test
    @Description("Проверка заголовка страницы с параметрами")
    void testWithParameters() {
        driver = new ChromeDriver();
        try {
            String url = "https://example.com";
            String expectedTitle = "Example Domain";
            Allure.parameter("URL", url);
            Allure.parameter("Ожидаемый заголовок", expectedTitle);
            openPage(url);
            verifyPageTitle(expectedTitle);
        } finally {
            driver.quit();
        }
    }

    @Step("Открытие страницы {url}")
    void openPage(String url) {
        driver.get(url);
    }

    @Step("Проверка заголовка: {expectedTitle}")
    void verifyPageTitle(String expectedTitle) {
        String actualTitle = driver.getTitle();
        Allure.parameter("Фактический заголовок", actualTitle);
        assert actualTitle.equals(expectedTitle) : "Title mismatch";
    }
}
```

### Пример с TestNG
```java
import io.qameta.allure.Allure;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.Test;

public class AllureParametersTest {
    private WebDriver driver;

    @Test(description = "Проверка заголовка страницы с параметрами")
    public void testWithParameters() {
        driver = new ChromeDriver();
        try {
            String url = "https://example.com";
            String expectedTitle = "Example Domain";
            Allure.parameter("URL", url);
            Allure.parameter("Ожидаемый заголовок", expectedTitle);
            Allure.step("Открытие страницы " + url, () -> {
                driver.get(url);
            });
            Allure.step("Проверка заголовка: " + expectedTitle, () -> {
                String actualTitle = driver.getTitle();
                Allure.parameter("Фактический заголовок", actualTitle);
                assert actualTitle.equals(expectedTitle) : "Title mismatch";
            });
        } finally {
            driver.quit();
        }
    }
}
```

### Лучшие практики
- Используйте `Allure.parameter` для явного указания входных и выходных данных.
- Включайте ожидаемые результаты в описание шагов через `@Step` или `Allure.step`.

## Поддержка запуска в Jenkins, GitLab CI и GitHub Actions

Allure интегрируется с CI/CD системами для автоматической генерации и публикации отчётов. Пример настройки для каждой системы:

### Jenkins
1. Установите плагин **Allure Jenkins Plugin**.
2. Настройте `pom.xml` для генерации отчётов:
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
3. Добавьте шаг в Jenkins pipeline:
```groovy
pipeline {
    agent any
    stages {
        stage('Test') {
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

### GitLab CI
```yaml
stages:
  - test
  - report

test:
  stage: test
  script:
    - mvn clean test
  artifacts:
    paths:
      - target/allure-results

allure_report:
  stage: report
  script:
    - mvn allure:report
  artifacts:
    paths:
      - target/site/allure-maven
```

### GitHub Actions
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
- Сохраняйте результаты Allure в папке `target/allure-results` для последующей обработки.
- Используйте Allure TestOps для централизованного хранения и анализа отчётов.
- Настройте архивирование артефактов в CI/CD для доступа к скриншотам и логам.

## Дополнительно

### Отладка
- **IntelliJ IDEA Debug**: Устанавливайте точки останова в методах `@Step` или `Allure.step` для проверки входных данных и состояния теста.
- **Локаторы**: Перед захватом скриншотов проверяйте их стабильность с помощью `driver.findElement().isDisplayed()`.
- **Ожидания**: Используйте `WebDriverWait` для синхронизации тестов перед добавлением вложений.

### Распространённые ошибки
- **Отсутствие отчётов**: Проверьте наличие папки `target/allure-results` и корректность плагина Allure.
- **Ошибки в CI/CD**: Убедитесь, что зависимости Maven установлены и версии Allure совместимы.
- **Перегрузка отчётов**: Ограничивайте размер вложений (например, сжимайте скриншоты).

## Источники
- [Allure Framework](https://allurereport.org/docs/)
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/)
- [Jenkins Allure Plugin](https://plugins.jenkins.io/allure/)

---
[**← Назад к оглавлению**](../README.md)