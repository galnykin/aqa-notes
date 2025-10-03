# Группировка тестов в Allure

Группировка тестов с использованием аннотаций `@Epic`, `@Feature` и `@Story` обеспечивает структурированную организацию тестов, упрощая навигацию в отчётах Allure и поиск нужных сценариев. Все тесты (UI и API) должны быть сгруппированы по `@Epic` и `@Feature`, а внутри `@Feature` — логически разбиты по `@Story`. Это улучшает читаемость и анализ отчётов для QA, разработчиков и бизнеса. Используется технологический стек: Java 17, Maven, SLF4J с Logback, JUnit 5, TestNG, Selenium WebDriver, RestAssured, Allure и Jenkins.

## Принципы группировки тестов

### Все тесты сгруппированы по @Epic и @Feature

- **@Epic**: Высокоуровневый бизнес-блок, отражающий крупный модуль приложения (например, «Авторизация», «Оплата»).
- **@Feature**: Функциональный раздел внутри эпика, описывающий конкретную функциональность (например, «Модальное окно входа», «API пользователей»).
- **Требование**: Каждый тест (UI или API) должен быть помечен аннотациями `@Epic` и `@Feature` для создания иерархической структуры в отчёте.

### Логическая разбивка по @Story внутри @Feature

- **@Story**: Конкретный пользовательский сценарий или тест-кейс внутри фичи (например, «Успешный вход», «Неверный формат email»).
- **Назначение**: Обеспечивает детализацию тестов внутри `@Feature`, упрощая поиск и анализ конкретных сценариев.
- **Логика**: Сценарии внутри одной `@Feature` должны быть связаны общей функциональностью, но различаться по конкретным действиям или проверкам.

### Преимущества группировки

- **Навигация**: Иерархия `@Epic > @Feature > @Story` позволяет быстро находить тесты в Allure-отчётах.
- **Поиск сценариев**: Логическая структура помогает QA и бизнесу идентифицировать тесты по функциональности.
- **Анализ**: Упрощает фильтрацию и приоритизацию тестов в CI/CD.

## Примеры реализации

### UI-тест с группировкой (JUnit 5)

```java
import io.qameta.allure.Description;
import io.qameta.allure.Epic;
import io.qameta.allure.Feature;
import io.qameta.allure.Owner;
import io.qameta.allure.Severity;
import io.qameta.allure.SeverityLevel;
import io.qameta.allure.Story;
import io.qameta.allure.Tag;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import static org.assertj.core.api.Assertions.assertThat;

@Epic("Авторизация")
@Feature("Модальное окно входа")
@Owner("qa_team")
@Tag("ui")
public class UiGroupedTest {
    private static final Logger logger = LoggerFactory.getLogger(UiGroupedTest.class);
    private WebDriver driver;

    @BeforeEach
    void setUp() {
        logger.info("Инициализация UI-теста");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
    }

    @Test
    @Story("Успешный вход")
    @Severity(SeverityLevel.BLOCKER)
    @Tag("smoke")
    @DisplayName("Проверка успешного входа с корректными данными")
    @Description("Тест проверяет вход в систему с корректными учетными данными")
    void testSuccessfulLogin() {
        logger.info("Открытие страницы логина");
        driver.get("https://example.com/login");
        logger.info("Проверка URL после входа");
        assertThat(driver.getCurrentUrl()).contains("login");
    }

    @Test
    @Story("Неверный формат email")
    @Severity(SeverityLevel.CRITICAL)
    @Tag("regression")
    @DisplayName("Проверка ошибки при неверном формате email")
    @Description("Тест проверяет отображение ошибки при вводе неверного email")
    void testInvalidEmailFormat() {
        logger.info("Открытие страницы логина");
        driver.get("https://example.com/login");
        logger.info("Ввод неверного email");
        // driver.findElement(By.id("email")).sendKeys("invalid-email");
        logger.info("Проверка сообщения об ошибке");
        assertThat(driver.getPageSource()).contains("Неверный формат email");
    }

    @AfterEach
    void tearDown() {
        logger.info("Завершение UI-теста");
        if (driver != null) {
            driver.quit();
        }
    }
}
```

### API-тест с группировкой (TestNG)

```java
import io.qameta.allure.Description;
import io.qameta.allure.Epic;
import io.qameta.allure.Feature;
import io.qameta.allure.Owner;
import io.qameta.allure.Severity;
import io.qameta.allure.SeverityLevel;
import io.qameta.allure.Story;
import io.qameta.allure.Tag;
import io.restassured.RestAssured;
import io.restassured.response.Response;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testng.annotations.Test;

import static io.restassured.RestAssured.given;
import static org.testng.Assert.assertEquals;

@Epic("API-тестирование")
@Feature("API пользователей")
@Owner("api_team")
@Tag("api")
public class ApiGroupedTest {
    private static final Logger logger = LoggerFactory.getLogger(ApiGroupedTest.class);

    @Test
    @Story("Получение данных пользователя")
    @Severity(SeverityLevel.CRITICAL)
    @Tag("regression")
    @Description("Тест проверяет получение данных пользователя по ID")
    @DisplayName("Проверка получения данных пользователя по ID")
    void testGetUserData() {
        logger.info("Отправка GET-запроса: /users/1");
        RestAssured.baseURI = "https://api.example.com";
        Response response = given()
                .when()
                .get("/users/1")
                .then()
                .extract().response();
        logger.info("Проверка статуса ответа: {}", response.getStatusCode());
        assertEquals(response.getStatusCode(), 200);
    }

    @Test
    @Story("Фильтрация пользователей")
    @Severity(SeverityLevel.NORMAL)
    @Tag("regression")
    @Description("Тест проверяет фильтрацию пользователей по параметру")
    @DisplayName("Проверка фильтрации пользователей по роли")
    void testUserFiltering() {
        logger.info("Отправка GET-запроса: /users?role=admin");
        RestAssured.baseURI = "https://api.example.com";
        Response response = given()
                .queryParam("role", "admin")
                .when()
                .get("/users")
                .then()
                .extract().response();
        logger.info("Проверка статуса ответа: {}", response.getStatusCode());
        assertEquals(response.getStatusCode(), 200);
    }
}
```

## Интеграция с CI/CD

Allure интегрируется с Jenkins для отображения иерархии `@Epic > @Feature > @Story`, упрощая навигацию.

- **Навигация в Allure**: Иерархия `@Epic > @Feature > @Story` отображается в отчёте, позволяя фильтровать тесты.
- **Логи**: Архивируются в `logs/*.log` для корреляции с отчётами.

## Лучшие практики и отладка

### Группировка тестов

- **@Epic**: Используйте для крупных модулей (например, «Авторизация», «Оплата»).
- **@Feature**: Указывайте конкретные функциональные области внутри эпика (например, «Модальное окно входа»).
- **@Story**: Разбивайте фичи на логические сценарии, такие как «Успешный вход» или «Неверный пароль».
- **Консистентность**: Убедитесь, что все тесты имеют аннотации `@Epic`, `@Feature` и `@Story`.

### Отладка

- **Локальная проверка**: Используйте `mvn allure:serve` для просмотра отчётов и проверки структуры.
- **Логи**: Анализируйте `logs/tests.log` в IntelliJ IDEA для корреляции с иерархией тестов.
- **CI/CD**: Убедитесь, что Jenkins корректно отображает иерархию в Allure-отчётах.

### Распространённые ошибки

- **Пропуск аннотаций**: Отсутствие `@Epic`, `@Feature` или `@Story` нарушает структуру отчёта.
- **Нелогичная разбивка**: Сценарии в `@Story` не соответствуют `@Feature`, что затрудняет навигацию.
- **Слишком общие эпики**: Например, `@Epic("Тестирование")` неинформативно.

## Дополнительно

### Пример с обработкой ошибок и скриншотами

```java
import io.qameta.allure.Allure;
import io.qameta.allure.Attachment;
import io.qameta.allure.Epic;
import io.qameta.allure.Feature;
import io.qameta.allure.Owner;
import io.qameta.allure.Severity;
import io.qameta.allure.SeverityLevel;
import io.qameta.allure.Story;
import io.qameta.allure.Tag;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import static org.assertj.core.api.Assertions.assertThat;

@Epic("Авторизация")
@Feature("Модальное окно входа")
@Owner("qa_team")
@Tag("ui")
@Tag("regression")
public class UiGroupedErrorTest {
    private static final Logger logger = LoggerFactory.getLogger(UiGroupedErrorTest.class);
    private WebDriver driver;

    @BeforeEach
    void setUp() {
        logger.info("Инициализация UI-теста");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
    }

    @Test
    @Story("Неверный формат email")
    @Severity(SeverityLevel.CRITICAL)
    @DisplayName("Проверка ошибки при неверном формате email")
    void testInvalidEmailFormat() {
        logger.info("Открытие страницы логина");
        driver.get("https://example.com/login");
        logger.info("Ввод неверного email");
        // driver.findElement(By.id("email")).sendKeys("invalid-email");
        try {
            logger.info("Проверка сообщения об ошибке");
            assertThat(driver.getPageSource()).contains("Invalid email");
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

    @AfterEach
    void tearDown() {
        logger.info("Завершение UI-теста");
        if (driver != null) {
            driver.quit();
        }
    }
}
```

### Интеграция с Testcontainers

```java
import io.qameta.allure.Epic;
import io.qameta.allure.Feature;
import io.qameta.allure.Owner;
import io.qameta.allure.Severity;
import io.qameta.allure.SeverityLevel;
import io.qameta.allure.Story;
import io.qameta.allure.Tag;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testcontainers.containers.GenericContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

@Testcontainers
@Epic("Интеграционное тестирование")
@Feature("Логирование в Logstash")
@Owner("devops_team")
@Tag("integration")
public class AllureGroupedContainerTest {
    private static final Logger logger = LoggerFactory.getLogger(AllureGroupedContainerTest.class);

    @Container
    private static final GenericContainer<?> logstash = new GenericContainer<>("logstash:8.15.2")
            .withExposedPorts(5000);

    @Test
    @Story("Отправка логов")
    @Severity(SeverityLevel.MINOR)
    @DisplayName("Проверка отправки логов в Logstash")
    void testLogstashIntegration() {
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