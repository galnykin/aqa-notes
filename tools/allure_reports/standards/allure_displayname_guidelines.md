# Формат и язык аннотации @DisplayName

Аннотация `@DisplayName` в JUnit 5 и TestNG задаёт человекочитаемое название теста, которое отображается в отчётах, таких как Allure, и упрощает понимание сути теста. В статье описываются рекомендации по формату и языку `@DisplayName`, включая понятность, краткость, отражение сути теста, использование единого языка и избегание технических терминов. Используется технологический стек: Java 17, Maven, SLF4J с Logback, JUnit 5, TestNG, Selenium WebDriver, RestAssured, Allure и Jenkins.

## Рекомендации по использованию @DisplayName

### Название должно быть понятным, кратким и отражать суть теста

- **Понятность**: Название должно быть доступно не только техническим специалистам, но и бизнес-пользователям.
- **Краткость**: Избегайте лишних слов, сохраняя суть.
- **Суть теста**: Название должно описывать, что именно проверяет тест (например, конкретный сценарий или функциональность).

**Примеры**:
- Хорошо: `@DisplayName("Проверка успешного входа с корректными данными")`
- Плохо: `@DisplayName("Тест логина с валидными credentials")` (технический термин и смешение языков).

### Использовать единый язык

- Выберите один язык для всех тестов (например, только русский или только английский).
- Единый язык упрощает чтение отчётов и поддерживает консистентность.

**Примеры**:
- Русский: `@DisplayName("Проверка обработки неверного формата email")`
- Английский: `@DisplayName("Verify handling of invalid email format")`
- Плохо: `@DisplayName("Verify вход с invalid email")` (смешение языков).

### Не использовать технические термины без необходимости

- Избегайте терминов, таких как "валидация", "endpoint", "assert", если они не добавляют ценности.
- Ориентируйтесь на бизнес-логику или пользовательский сценарий.

**Примеры**:
- Хорошо: `@DisplayName("Проверка отображения ошибки при неверном пароле")`
- Плохо: `@DisplayName("Assert error message on invalid password input")` (технические термины).

## Примеры реализации

### UI-тест с @DisplayName (JUnit 5)

```java
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
@Story("Проверка входа")
@Owner("qa_team")
@Tag("ui")
@Tag("smoke")
@Severity(SeverityLevel.CRITICAL)
public class UiDisplayNameTest {
    private static final Logger logger = LoggerFactory.getLogger(UiDisplayNameTest.class);
    private WebDriver driver;

    @BeforeEach
    void setUp() {
        logger.info("Инициализация UI-теста");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
    }

    @Test
    @DisplayName("Проверка успешного входа с корректными данными")
    void testSuccessfulLogin() {
        logger.info("Открытие страницы логина");
        driver.get("https://example.com/login");
        logger.info("Проверка URL после входа");
        assertThat(driver.getCurrentUrl()).contains("login");
    }

    @Test
    @DisplayName("Проверка ошибки при неверном формате email")
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

### API-тест с @DisplayName (TestNG)

```java
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
@Feature("Получение данных пользователя")
@Story("Проверка ответа API")
@Owner("api_team")
@Tag("api")
@Tag("regression")
@Severity(SeverityLevel.NORMAL)
public class ApiDisplayNameTest {
    private static final Logger logger = LoggerFactory.getLogger(ApiDisplayNameTest.class);

    @Test
    @DisplayName("Проверка получения данных пользователя по ID")
    void testApiUserData() {
        logger.info("Отправка GET-запроса: /users/1");
        RestAssured.baseURI = "https://api.example.com";
        Response response = given()
                .when()
                .get("/users/1")
                .then()
                .extract().response();
        logger.info("Проверка статуса ответа: {}", response.getStatusCode());
        assertEquals(response.getStatusCode(), 200, "Ожидался статус 200");
    }
}
```

## Интеграция с CI/CD

Allure интегрируется с Jenkins для автоматической генерации отчётов, где `@DisplayName` улучшает читаемость.

- **@DisplayName в отчётах**: Названия тестов отображаются в Allure, упрощая анализ.
- **Фильтрация**: Используйте `@Tag` и `@Severity` для выборочного запуска тестов.

## Лучшие практики и отладка

### Формат @DisplayName

- **Понятность**: Используйте простые фразы, например, «Проверка успешного входа» вместо «Тест логина».
- **Краткость**: Ограничивайтесь 5–10 словами, избегая лишних деталей.
- **Суть теста**: Описывайте сценарий, например, «Проверка ошибки при неверном пароле».

### Единый язык

- Договоритесь в команде о языке (например, только русский).
- Проверяйте консистентность в отчётах Allure.

### Избегание технических терминов

- Заменяйте «валидация» на «проверка», «endpoint» на «запрос» и т.д.
- Ориентируйтесь на бизнес-термины, понятные всем.

### Отладка

- **Локальная проверка**: Используйте `mvn allure:serve` для просмотра отчётов и проверки `@DisplayName`.
- **Логи**: Анализируйте `logs/tests.log` в IntelliJ IDEA для корреляции с названиями тестов.
- **CI/CD**: Убедитесь, что Jenkins корректно отображает `@DisplayName` в Allure-отчётах.

### Распространённые ошибки

- **Смешение языков**: Например, `@DisplayName("Verify вход с корректными данными")`.
- **Слишком длинные названия**: Например, `@DisplayName("Проверка функциональности входа в систему с корректными учетными данными пользователя")`.
- **Технические термины**: Например, `@DisplayName("Assert response code 200 for API endpoint")`.

## Дополнительно

### Пример с обработкой ошибок

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
@Story("Обработка ошибок")
@Owner("qa_team")
@Tag("ui")
@Tag("regression")
@Severity(SeverityLevel.NORMAL)
public class UiDisplayNameErrorTest {
    private static final Logger logger = LoggerFactory.getLogger(UiDisplayNameErrorTest.class);
    private WebDriver driver;

    @BeforeEach
    void setUp() {
        logger.info("Инициализация UI-теста");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
    }

    @Test
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
@Story("Проверка отправки логов")
@Owner("devops_team")
@Tag("integration")
@Severity(SeverityLevel.MINOR)
public class AllureContainerDisplayNameTest {
    private static final Logger logger = LoggerFactory.getLogger(AllureContainerDisplayNameTest.class);

    @Container
    private static final GenericContainer<?> logstash = new GenericContainer<>("logstash:8.15.2")
            .withExposedPorts(5000);

    @Test
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