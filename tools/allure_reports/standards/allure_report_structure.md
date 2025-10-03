# Структура отчёта Allure

Allure предоставляет структурированный подход к организации тестов через аннотации `@Epic`, `@Feature`, `@Story` и `@Severity`, что улучшает читаемость и понимание отчётов. В статье описывается использование этих аннотаций для категоризации тестов и указания их критичности. Используется технологический стек: Java 17, Maven, SLF4J с Logback, JUnit 5, TestNG, Selenium WebDriver, RestAssured, Allure и Jenkins.

## Структура отчёта

Allure-отчёты используют иерархическую структуру для классификации тестов, что помогает командам (QA, разработчики, бизнес) понимать назначение и приоритет тестов.

### @Epic — крупный бизнес-блок

Аннотация `@Epic` обозначает высокоуровневый бизнес-блок, например, модуль приложения, такой как «Авторизация» или «Платежи».

- **Назначение**: Группировка тестов по крупным функциональным областям.
- **Применение**: Используется для обозначения общей цели группы тестов.

**Пример**: Тесты авторизации относятся к эпику «Авторизация».

### @Feature — функциональный раздел

Аннотация `@Feature` указывает на конкретный функциональный раздел внутри эпика, например, «Модальное окно входа» или «Регистрация пользователя».

- **Назначение**: Детализация функциональности внутри эпика.
- **Применение**: Помогает выделить подмодули или функции.

**Пример**: В рамках эпика «Авторизация» фича может быть «Модальное окно входа».

### @Story — конкретный сценарий

Аннотация `@Story` описывает конкретный пользовательский сценарий или кейс, например, «Неверный формат email» или «Успешный вход».

- **Назначение**: Уточнение конкретного тестового сценария.
- **Применение**: Связывает тест с пользовательской историей.

**Пример**: В фиче «Модальное окно входа» стори может быть «Неверный формат email».

### @Severity — критичность теста

Аннотация `@Severity` определяет важность теста с уровнями: `BLOCKER`, `CRITICAL`, `NORMAL`, `MINOR`, `TRIVIAL`.

- **Назначение**: Указывает приоритет теста для анализа и исправления ошибок.
- **Применение**: Помогает бизнесу и разработчикам понять, какие тесты наиболее важны.

**Уровни критичности**:
- `BLOCKER`: Тест, сбой которого блокирует основную функциональность.
- `CRITICAL`: Тест, затрагивающий ключевые функции, но с обходным путём.
- `NORMAL`: Стандартный тест, сбой которого не критичен.
- `MINOR`: Незначительные проблемы, не влияющие на функциональность.
- `TRIVIAL`: Косметические дефекты.

## Пример реализации

### Тест с аннотациями (JUnit 5)

```java
import io.qameta.allure.Description;
import io.qameta.allure.Epic;
import io.qameta.allure.Feature;
import io.qameta.allure.Severity;
import io.qameta.allure.SeverityLevel;
import io.qameta.allure.Step;
import io.qameta.allure.Story;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import static org.assertj.core.api.Assertions.assertThat;

public class AllureStructureTest {
    private static final Logger logger = LoggerFactory.getLogger(AllureStructureTest.class);
    private WebDriver driver;

    @BeforeEach
    void setUp() {
        logger.info("Инициализация теста");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
    }

    @Test
    @Epic("Авторизация")
    @Feature("Модальное окно входа")
    @Story("Неверный формат email")
    @Severity(SeverityLevel.CRITICAL)
    @Description("Проверка обработки неверного формата email в модальном окне входа")
    void testInvalidEmailFormat() {
        openLoginPage();
        enterInvalidEmail();
        verifyErrorMessage();
    }

    @Step("Открытие страницы логина")
    private void openLoginPage() {
        logger.info("Открытие страницы: https://example.com/login");
        driver.get("https://example.com/login");
    }

    @Step("Ввод неверного email: {email}")
    private void enterInvalidEmail() {
        logger.info("Ввод email: invalid-email");
        // Предполагается, что есть элемент для ввода email
        // driver.findElement(By.id("email")).sendKeys("invalid-email");
    }

    @Step("Проверка сообщения об ошибке")
    private void verifyErrorMessage() {
        logger.info("Проверка наличия сообщения об ошибке");
        // Предполагается проверка текста ошибки
        assertThat(driver.getPageSource()).contains("Неверный формат email");
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

### Тест с аннотациями (TestNG)

```java
import io.qameta.allure.Description;
import io.qameta.allure.Epic;
import io.qameta.allure.Feature;
import io.qameta.allure.Severity;
import io.qameta.allure.SeverityLevel;
import io.qameta.allure.Step;
import io.qameta.allure.Story;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;

import static org.testng.Assert.assertTrue;

public class AllureStructureTestNG {
    private static final Logger logger = LoggerFactory.getLogger(AllureStructureTestNG.class);
    private WebDriver driver;

    @BeforeMethod
    void setUp() {
        logger.info("Инициализация теста");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
    }

    @Test
    @Epic("Авторизация")
    @Feature("Модальное окно входа")
    @Story("Неверный формат email")
    @Severity(SeverityLevel.CRITICAL)
    @Description("Проверка обработки неверного формата email в модальном окне входа")
    void testInvalidEmailFormat() {
        openLoginPage();
        enterInvalidEmail();
        verifyErrorMessage();
    }

    @Step("Открытие страницы логина")
    private void openLoginPage() {
        logger.info("Открытие страницы: https://example.com/login");
        driver.get("https://example.com/login");
    }

    @Step("Ввод неверного email: {email}")
    private void enterInvalidEmail() {
        logger.info("Ввод email: invalid-email");
        // driver.findElement(By.id("email")).sendKeys("invalid-email");
    }

    @Step("Проверка сообщения об ошибке")
    private void verifyErrorMessage() {
        logger.info("Проверка наличия сообщения об ошибке");
        assertTrue(driver.getPageSource().contains("Неверный формат email"));
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

## Интеграция с Allure

Allure-отчёты отображают иерархию (`Epic > Feature > Story`) и критичность (`Severity`) в удобном формате.

## Интеграция с CI/CD

Allure интегрируется с Jenkins для автоматической генерации отчётов.

Allure-отчёт показывает иерархию тестов (`Epic`, `Feature`, `Story`) и их критичность, упрощая анализ.

## Лучшие практики и отладка

### Использование аннотаций

- **@Epic**: Применяйте для крупных модулей, таких как «Авторизация» или «Платежи».
- **@Feature**: Указывайте конкретные функции, например, «Модальное окно входа».
- **@Story**: Описывайте пользовательские сценарии, такие как «Неверный формат email».
- **@Severity**: Устанавливайте критичность в зависимости от влияния теста на функциональность.

### Отладка

- Проверяйте Allure-отчёты локально с помощью `mvn allure:serve`.
- Используйте IntelliJ IDEA для анализа логов (`logs/tests.log`) и корреляции с Allure.

### Распространённые ошибки

- **Неправильная иерархия**: Смешивание `@Epic` и `@Feature` усложняет навигацию.
- **Отсутствие `@Severity`**: Без указания критичности трудно оценить приоритет.
- **Недостаточная детализация**: Пропуск `@Step` снижает прозрачность теста.

## Дополнительно

### Пример API-теста с Allure

```java
import io.qameta.allure.Epic;
import io.qameta.allure.Feature;
import io.qameta.allure.Severity;
import io.qameta.allure.SeverityLevel;
import io.qameta.allure.Step;
import io.qameta.allure.Story;
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
    @Epic("API-тестирование")
    @Feature("Получение данных пользователя")
    @Story("Проверка ответа API")
    @Severity(SeverityLevel.NORMAL)
    void testApiStructure() {
        sendGetRequest();
    }

    @Step("Отправка GET-запроса")
    private void sendGetRequest() {
        logger.info("Отправка GET-запроса: /users/1");
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
import io.qameta.allure.Epic;
import io.qameta.allure.Feature;
import io.qameta.allure.Severity;
import io.qameta.allure.SeverityLevel;
import io.qameta.allure.Step;
import io.qameta.allure.Story;
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
    @Epic("Интеграционное тестирование")
    @Feature("Логирование в Logstash")
    @Story("Проверка отправки логов")
    @Severity(SeverityLevel.MINOR)
    void testLogstashIntegration() {
        logContainerStep();
    }

    @Step("Отправка лога в Logstash")
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