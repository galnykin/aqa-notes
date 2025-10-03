# Логика назначения уровней Severity в Allure

Аннотация `@Severity` в Allure позволяет классифицировать тесты по их критичности, что помогает приоритизировать исправление ошибок и фокусироваться на важных сценариях. В статье описывается логика назначения уровней `BLOCKER`, `CRITICAL`, `NORMAL`, `MINOR` и `TRIVIAL` с примерами для UI и API тестов. Используется технологический стек: Java 17, Maven, SLF4J с Logback, JUnit 5, TestNG, Selenium WebDriver, RestAssured, Allure и Jenkins.

## Логика назначения уровней Severity

Уровень критичности теста определяется на основе влияния его сбоя на функциональность приложения и бизнес-процессы.

### BLOCKER — падение критичного функционала

- **Описание**: Тест проверяет функциональность, без которой приложение становится неработоспособным (например, невозможность входа в систему, оформить заказ или получить доступ к основным функциям).
- **Когда использовать**: Сбой теста блокирует ключевые сценарии пользователей.
- **Примеры**:
  - UI: Проверка входа в систему с корректными данными.
  - API: Проверка доступности основного endpoint для авторизации.

### CRITICAL — важные бизнес-сценарии

- **Описание**: Тест проверяет важные функции, которые влияют на бизнес-логику, но их сбой может иметь обходной путь.
- **Когда использовать**: Тест связан с ключевыми бизнес-процессами, такими как регистрация, оплата или обработка данных.
- **Примеры**:
  - UI: Проверка корректной обработки формы регистрации.
  - API: Проверка корректного ответа API для получения данных пользователя.

### NORMAL — стандартные проверки

- **Описание**: Тесты, проверяющие типичную функциональность, которая не является критической для работы системы.
- **Когда использовать**: Для стандартных сценариев, сбои которых не оказывают значительного влияния на пользователей.
- **Примеры**:
  - UI: Проверка отображения кнопки на странице.
  - API: Проверка фильтрации данных в API.

### MINOR — незначительные UI-ошибки

- **Описание**: Тесты, связанные с некритичными проблемами пользовательского интерфейса, которые не влияют на функциональность.
- **Когда использовать**: Для ошибок, которые не мешают основным сценариям, но могут повлиять на пользовательский опыт.
- **Примеры**:
  - UI: Проверка правильного выравнивания элементов интерфейса.
  - API: Проверка формата ответа API (например, порядок полей).

### TRIVIAL — косметика, тексты, отступы

- **Описание**: Тесты, проверяющие косметические детали, такие как орфография, отступы или стили, которые не влияют на функциональность или пользовательский опыт.
- **Когда использовать**: Для мелких дефектов, которые не требуют срочного исправления.
- **Примеры**:
  - UI: Проверка орфографии в тексте подсказки.
  - API: Проверка наличия необязательного поля в ответе API.

## Примеры реализации

### UI-тест с разными уровнями Severity (JUnit 5)

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
public class UiSeverityTest {
    private static final Logger logger = LoggerFactory.getLogger(UiSeverityTest.class);
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
    @Description("Тест проверяет возможность входа в систему с корректными учетными данными")
    void testSuccessfulLogin() {
        logger.info("Открытие страницы логина");
        driver.get("https://example.com/login");
        logger.info("Проверка URL после входа");
        assertThat(driver.getCurrentUrl()).contains("login");
    }

    @Test
    @Story("Обработка неверного пароля")
    @Severity(SeverityLevel.CRITICAL)
    @Tag("regression")
    @DisplayName("Проверка ошибки при неверном пароле")
    @Description("Тест проверяет отображение ошибки при вводе неверного пароля")
    void testInvalidPassword() {
        logger.info("Открытие страницы логина");
        driver.get("https://example.com/login");
        logger.info("Ввод неверного пароля");
        // driver.findElement(By.id("password")).sendKeys("wrong");
        logger.info("Проверка сообщения об ошибке");
        assertThat(driver.getPageSource()).contains("Неверный пароль");
    }

    @Test
    @Story("Отображение кнопки входа")
    @Severity(SeverityLevel.NORMAL)
    @Tag("regression")
    @DisplayName("Проверка отображения кнопки входа")
    @Description("Тест проверяет, что кнопка входа отображается на странице")
    void testLoginButtonVisibility() {
        logger.info("Открытие страницы логина");
        driver.get("https://example.com/login");
        logger.info("Проверка видимости кнопки");
        // assertThat(driver.findElement(By.id("loginButton")).isDisplayed()).isTrue();
    }

    @Test
    @Story("Выравнивание элементов формы")
    @Severity(SeverityLevel.MINOR)
    @Tag("regression")
    @DisplayName("Проверка выравнивания поля ввода email")
    @Description("Тест проверяет корректное выравнивание поля ввода email")
    void testEmailFieldAlignment() {
        logger.info("Открытие страницы логина");
        driver.get("https://example.com/login");
        logger.info("Проверка выравнивания поля email");
        // assertThat(driver.findElement(By.id("email")).getCssValue("text-align")).isEqualTo("left");
    }

    @Test
    @Story("Орфография в подсказке")
    @Severity(SeverityLevel.TRIVIAL)
    @Tag("regression")
    @DisplayName("Проверка текста подсказки для поля email")
    @Description("Тест проверяет орфографию в подсказке для поля email")
    void testEmailHintText() {
        logger.info("Открытие страницы логина");
        driver.get("https://example.com/login");
        logger.info("Проверка текста подсказки");
        // assertThat(driver.findElement(By.id("emailHint")).getText()).isEqualTo("Введите ваш email");
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

### API-тест с разными уровнями Severity (TestNG)

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
@Feature("Получение данных пользователя")
@Owner("api_team")
@Tag("api")
public class ApiSeverityTest {
    private static final Logger logger = LoggerFactory.getLogger(ApiSeverityTest.class);

    @Test
    @Story("Доступность API авторизации")
    @Severity(SeverityLevel.BLOCKER)
    @Tag("smoke")
    @Description("Тест проверяет доступность endpoint'а авторизации")
    void testAuthEndpointAvailability() {
        logger.info("Отправка POST-запроса: /auth");
        RestAssured.baseURI = "https://api.example.com";
        Response response = given()
                .body("{\"username\":\"user\",\"password\":\"pass\"}")
                .when()
                .post("/auth")
                .then()
                .extract().response();
        logger.info("Проверка статуса ответа: {}", response.getStatusCode());
        assertEquals(response.getStatusCode(), 200);
    }

    @Test
    @Story("Получение данных пользователя")
    @Severity(SeverityLevel.CRITICAL)
    @Tag("regression")
    @Description("Тест проверяет получение данных пользователя по ID")
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

    @Test
    @Story("Формат ответа API")
    @Severity(SeverityLevel.MINOR)
    @Tag("regression")
    @Description("Тест проверяет порядок полей в ответе API")
    void testResponseFormat() {
        logger.info("Отправка GET-запроса: /users/1");
        RestAssured.baseURI = "https://api.example.com";
        Response response = given()
                .when()
                .get("/users/1")
                .then()
                .extract().response();
        logger.info("Проверка формата ответа");
        assertEquals(response.jsonPath().getString("id"), "1");
    }

    @Test
    @Story("Необязательное поле в ответе")
    @Severity(SeverityLevel.TRIVIAL)
    @Tag("regression")
    @Description("Тест проверяет наличие необязательного поля в ответе API")
    void testOptionalField() {
        logger.info("Отправка GET-запроса: /users/1");
        RestAssured.baseURI = "https://api.example.com";
        Response response = given()
                .when()
                .get("/users/1")
                .then()
                .extract().response();
        logger.info("Проверка наличия поля description");
        assertEquals(response.jsonPath().getString("description"), "");
    }
}
```

## Интеграция с CI/CD

Allure интегрируется с Jenkins для автоматической генерации отчётов, где `@Severity` помогает фильтровать тесты.

- **Фильтрация по Severity**: В Allure-отчётах можно отфильтровать тесты, например, только `BLOCKER` или `CRITICAL`, для анализа ключевых проблем.
- **Логи**: Архивируются в `logs/*.log` для корреляции с отчётами.

## Лучшие практики и отладка

### Назначение Severity

- **BLOCKER**: Используйте для тестов, проверяющих критические функции, такие как вход в систему или основной API.
- **CRITICAL**: Применяйте к тестам ключевых бизнес-сценариев, например, регистрация или обработка платежей.
- **NORMAL**: Для стандартных проверок, не влияющих на ключевые функции.
- **MINOR**: Для UI-тестов, связанных с некритичными элементами интерфейса.
- **TRIVIAL**: Для косметических проверок, таких как орфография или отступы.

### Отладка

- **Локальная проверка**: Используйте `mvn allure:serve` для просмотра отчётов и проверки уровней `@Severity`.
- **Логи**: Анализируйте `logs/tests.log` в IntelliJ IDEA для корреляции с Allure-отчётами.
- **CI/CD**: Убедитесь, что Jenkins фильтрует тесты по `@Severity` для приоритизации анализа.

### Распространённые ошибки

- **Завышение критичности**: Присвоение `BLOCKER` некритичным тестам снижает значимость отчётов.
- **Пропуск `@Severity`**: Отсутствие аннотации затрудняет фильтрацию.
- **Неправильная категоризация**: Например, тест UI-выравнивания помечен как `CRITICAL`.

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
@Story("Обработка ошибок")
@Owner("qa_team")
@Tag("ui")
@Tag("regression")
public class UiSeverityErrorTest {
    private static final Logger logger = LoggerFactory.getLogger(UiSeverityErrorTest.class);
    private WebDriver driver;

    @BeforeEach
    void setUp() {
        logger.info("Инициализация UI-теста");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
    }

    @Test
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
@Story("Проверка отправки логов")
@Owner("devops_team")
@Tag("integration")
public class AllureSeverityContainerTest {
    private static final Logger logger = LoggerFactory.getLogger(AllureSeverityContainerTest.class);

    @Container
    private static final GenericContainer<?> logstash = new GenericContainer<>("logstash:8.15.2")
            .withExposedPorts(5000);

    @Test
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