# Цели Allure-репортинга

Allure — мощный инструмент для создания структурированных и наглядных отчётов по тестам, который повышает прозрачность, упрощает анализ ошибок и поддерживает коммуникацию между командами. В статье рассматриваются цели Allure-репортинга: повышение прозрачности тестов, упрощение анализа падений, коммуникация между QA, разработкой и бизнесом, а также интеграция с CI/CD для автоматических отчётов. Используется технологический стек: Java 17, Maven, SLF4J с Logback, JUnit 5, TestNG, Selenium WebDriver, RestAssured, Allure и Jenkins.

## Повышение прозрачности тестов

Allure помогает сделать тесты понятными, показывая, что проверяется, как и почему. Это достигается через структурированные шаги, аннотации и прикреплённые данные.

- **Что проверяется**: Аннотации `@Description` и `@Step` описывают цель теста и его шаги.
- **Как проверяется**: Логи, скриншоты и входные данные прикрепляются к отчёту.
- **Почему**: Категории (теги, severity) указывают на приоритет и контекст теста.

**Пример теста с Allure (JUnit 5)**:

```java
import io.qameta.allure.Description;
import io.qameta.allure.Severity;
import io.qameta.allure.SeverityLevel;
import io.qameta.allure.Step;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import static org.assertj.core.api.Assertions.assertThat;

public class TransparentTest {
    private static final Logger logger = LoggerFactory.getLogger(TransparentTest.class);
    private WebDriver driver;

    @BeforeEach
    void setUp() {
        logger.info("Инициализация теста");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
    }

    @Test
    @Description("Проверка страницы логина для прозрачности")
    @Severity(SeverityLevel.CRITICAL)
    void testLoginTransparency() {
        openLoginPage();
        verifyPageUrl();
    }

    @Step("Открытие страницы логина")
    private void openLoginPage() {
        logger.info("Открытие страницы: https://example.com/login");
        driver.get("https://example.com/login");
    }

    @Step("Проверка URL страницы")
    private void verifyPageUrl() {
        String currentUrl = driver.getCurrentUrl();
        logger.info("Проверка URL: {}", currentUrl);
        assertThat(currentUrl).contains("login");
    }

    @AfterEach
    void tearDown() {
        logger.info("Завершение теста");
        if (driver != null) {
            driver.quit();
        }
    }
}
```

**Лог и отчёт Allure**:

- Лог: Записывает шаги и результаты в `logs/tests.log`.
- Allure: Отображает шаги, их статус и описание в отчёте.

## Упрощение анализа падений и ошибок

Allure упрощает анализ падений, предоставляя подробную информацию об ошибках, включая скриншоты, логи и стек вызовов.

- **Что прикреплять**: Скриншоты при сбоях, HTTP-запросы, ответы API, логи.
- **Как использовать**: Аннотация `@Attachment` для добавления данных в отчёт.

**Пример с обработкой ошибок (TestNG)**:

```java
import io.qameta.allure.Allure;
import io.qameta.allure.Attachment;
import io.qameta.allure.Description;
import io.qameta.allure.Severity;
import io.qameta.allure.SeverityLevel;
import io.qameta.allure.Step;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;

import static org.testng.Assert.assertTrue;

public class FailureAnalysisTest {
    private static final Logger logger = LoggerFactory.getLogger(FailureAnalysisTest.class);
    private WebDriver driver;

    @BeforeMethod
    void setUp() {
        logger.info("Инициализация теста");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
    }

    @Test
    @Description("Проверка страницы логина с анализом ошибок")
    @Severity(SeverityLevel.CRITICAL)
    void testLoginFailure() {
        openLoginPage();
        verifyPageUrl();
    }

    @Step("Открытие страницы логина")
    private void openLoginPage() {
        logger.info("Открытие страницы: https://example.com/login");
        driver.get("https://example.com/login");
    }

    @Step("Проверка URL страницы")
    private void verifyPageUrl() {
        String currentUrl = driver.getCurrentUrl();
        logger.info("Проверка URL: {}", currentUrl);
        try {
            assertTrue(currentUrl.contains("invalid"), "URL не содержит 'invalid'");
        } catch (AssertionError e) {
            logger.error("Ошибка в тесте: {}", e.getMessage(), e);
            attachScreenshot();
            throw e;
        }
    }

    @Attachment(value = "Скриншот ошибки", type = "image/png")
    private byte[] attachScreenshot() {
        return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
    }

    @AfterMethod
    void tearDown() {
        logger.info("Завершение теста");
        if (driver != null) {
            driver.quit();
        }
    }
}
```

**Allure-отчёт**:

- Показывает шаг с ошибкой, текст исключения и прикреплённый скриншот.
- Логи из Logback дополняют отчёт контекстом.

## Поддержка коммуникации между QA, разработкой и бизнесом

Allure-отчёты упрощают коммуникацию, предоставляя понятные данные для всех заинтересованных сторон.

- **QA**: Анализируют шаги и результаты тестов.
- **Разработчики**: Используют скриншоты и логи для отладки.
- **Бизнес**: Понимают покрытие тестов и критичность ошибок через теги `@Severity`.

**Пример с категоризацией**:

```java
import io.qameta.allure.Allure;
import io.qameta.allure.Epic;
import io.qameta.allure.Feature;
import io.qameta.allure.Story;
import io.qameta.allure.Step;
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class CommunicationTest {
    private static final Logger logger = LoggerFactory.getLogger(CommunicationTest.class);

    @Test
    @Epic("Авторизация")
    @Feature("Логин")
    @Story("Проверка успешного входа")
    void testForCommunication() {
        logBusinessStep("Проверка страницы логина");
        Allure.addAttachment("Контекст", "Тест проверяет вход пользователя");
    }

    @Step("Бизнес-шаг: {step}")
    private void logBusinessStep(String step) {
        logger.info("Бизнес: {}", step);
        Allure.label("owner", "QA-Team");
    }
}
```

**Allure-отчёт**:

- Категории (`Epic`, `Feature`, `Story`) помогают бизнесу понять назначение теста.
- Логи и метки (`owner`) улучшают коммуникацию.

## Интеграция с CI/CD для автоматических отчётов

Allure интегрируется с CI/CD (например, Jenkins) для автоматической генерации и публикации отчётов.

### Настройка Jenkins

**Пример `Jenkinsfile`**:

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
                allure([
                    includeProperties: false,
                    jdk: '',
                    results: [[path: 'target/allure-results']]
                ])
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'logs/*.log', allowEmptyArchive: true
        }
    }
}
```

- Тесты генерируют данные в `target/allure-results`.
- Плагин Allure в Jenkins формирует отчёт, доступный через веб-интерфейс.

### Конфигурация Logback

Логи дополняют Allure-отчёты:

```xml
<configuration>
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/tests.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>logs/tests.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>10MB</maxFileSize>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="info">
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

**Зависимости Maven**:

```xml
<dependency>
    <groupId>io.qameta.allure</groupId>
    <artifactId>allure-junit5</artifactId>
    <version>2.29.0</version>
</dependency>
<dependency>
    <groupId>io.qameta.allure</groupId>
    <artifactId>allure-testng</artifactId>
    <version>2.29.0</version>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.5.8</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>2.0.16</version>
</dependency>
```

## Лучшие практики и отладка

### Прозрачность

- Используйте `@Step` и `@Description` для чёткого описания тестов.
- Прикрепляйте входные данные, скриншоты и логи через `@Attachment`.

### Анализ ошибок

- Добавляйте скриншоты и логи при сбоях.
- Используйте `try-catch` для обработки ошибок и их логирования.

### Коммуникация

- Применяйте теги (`@Epic`, `@Feature`, `@Story`) для категоризации тестов.
- Добавляйте метки (`owner`, `severity`) для ясности ролей и приоритетов.

### CI/CD

- Настройте автоматическую генерацию Allure-отчётов в Jenkins.
- Архивируйте логи для последующего анализа.

### Отладка

- В IntelliJ IDEA проверяйте Allure-отчёты локально с помощью `mvn allure:serve`.
- Анализируйте логи в `logs/tests.log` для корреляции с Allure.

### Распространённые ошибки

- **Недостаточная детализация**: Пропуск `@Step` снижает прозрачность.
- **Отсутствие вложений**: Не прикреплены скриншоты или логи при сбоях.
- **Сложные отчёты**: Избыточные шаги затрудняют коммуникацию.

## Дополнительно

### Интеграция с API-тестами

Allure поддерживает API-тесты с RestAssured.

**Пример API-теста**:

```java
import io.qameta.allure.Step;
import io.restassured.RestAssured;
import io.restassured.response.Response;
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import static io.restassured.RestAssured.given;
import static org.assertj.core.api.Assertions.assertThat;

public class ApiAllureTest {
    private static final Logger logger = LoggerFactory.getLogger(ApiAllureTest.class);

    @Test
    void testApiTransparency() {
        logger.info("Запуск API-теста");
        sendGetRequest();
    }

    @Step("Отправка GET-запроса")
    private void sendGetRequest() {
        RestAssured.baseURI = "https://api.example.com";
        Response response = given()
                .when()
                .get("/users/1")
                .then()
                .extract().response();
        logger.info("Статус ответа: {}", response.getStatusCode());
        assertThat(response.getStatusCode()).isEqualTo(200);
    }
}
```

### Использование Testcontainers

Testcontainers помогает тестировать интеграцию с системами логирования.

```java
import io.qameta.allure.Step;
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testcontainers.containers.GenericContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

@Testcontainers
public class AllureContainerTest {
    private static final Logger logger = LoggerFactory.getLogger(AllureContainerTest.class);

    @Container
    private static final GenericContainer<?> logstash = new GenericContainer<>("logstash:8.15.2")
            .withExposedPorts(5000);

    @Test
    void testLogstashIntegration() {
        logContainerStep();
    }

    @Step("Проверка интеграции с Logstash")
    private void logContainerStep() {
        logger.info("Отправка лога в Logstash на {}:{}", logstash.getHost(), logstash.getFirstMappedPort());
    }
}
```

## Источники

- [Allure Framework](https://allurereport.org/docs/)
- [SLF4J User Manual](http://www.slf4j.org/manual.html)
- [Logback Documentation](https://logback.qos.ch/documentation.html)
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/webdriver/)
- [RestAssured Documentation](https://rest-assured.io/)
- [Testcontainers Documentation](https://testcontainers.org/)
- [Jenkins Pipeline Documentation](https://www.jenkins.io/doc/book/pipeline/)

---
[**← Назад к оглавлению**](../README.md)