# Структура тестов

Эффективная структура тестов повышает читаемость, масштабируемость и удобство поддержки автотестов. Разделение тестов по слоям (UI, API, интеграционные, smoke, regression) и использование единого шаблона с аннотациями TestNG (`@Test`, `@BeforeMethod`, `@AfterMethod`) в сочетании с классом `BaseTest` для инициализации окружения позволяют создавать надёжные тестовые наборы. В этой статье рассматриваются лучшие практики организации тестов, дополняющие ранее описанные подходы, с акцентом на новый контент.

## Разделение тестов по слоям

Тесты разделяются по слоям в зависимости от их цели и уровня взаимодействия с системой. Это упрощает управление, отладку и запуск тестов.

### UI-тесты
UI-тесты проверяют пользовательский интерфейс через браузер с использованием Selenium WebDriver или Selenide. Они наиболее ресурсоёмкие и медленные, поэтому их количество должно быть минимальным.

- **Пример сценария**: Проверка процесса логина через веб-форму.
- **Особенности**:
  - Используйте Page Object Model (POM) с аннотациями `@FindBy` и `PageFactory`.
  - Применяйте `WebDriverWait` для устойчивости.
  - Проверяйте стабильные локаторы (`id`, `data-*`).

### API-тесты
API-тесты проверяют взаимодействие с сервером через HTTP-запросы, используя RestAssured. Они быстрее UI-тестов и проверяют логику приложения на уровне API.

- **Пример сценария**: Проверка создания пользователя через POST-запрос.
- **Особенности**:
  - Используйте RestAssured для формирования запросов и валидации ответов.
  - Проверяйте статус-коды, тело ответа и заголовки.
  - Используйте AssertJ для мягких утверждений.

### Интеграционные тесты
Интеграционные тесты проверяют взаимодействие между компонентами системы, например, между приложением и базой данных или внешними сервисами.

- **Пример сценария**: Проверка, что данные, отправленные через API, корректно сохраняются в базе.
- **Особенности**:
  - Используйте Testcontainers для запуска сервисов (например, базы данных) в Docker.
  - Проверяйте согласованность данных между слоями.

### Smoke-тесты
Smoke-тесты проверяют основные функции приложения, чтобы убедиться, что система работает в целом.

- **Пример сценария**: Проверка доступности главной страницы и базового функционала.
- **Особенности**:
  - Минимизируйте количество smoke-тестов.
  - Используйте группу `smoke` в TestNG для быстрого запуска.

### Regression-тесты
Regression-тесты охватывают весь функционал приложения, чтобы выявить регрессии после изменений.

- **Пример сценария**: Проверка всех сценариев логина, включая позитивные и негативные случаи.
- **Особенности**:
  - Включайте тесты из всех слоёв (UI, API, интеграционные).
  - Используйте группу `regression` для запуска в CI/CD.

### Лучшие практики разделения
- **Иерархия тестов**: Структурируйте тесты по пакетам, например, `tests.ui`, `tests.api`, `tests.integration`, `tests.smoke`, `tests.regression`.
- **Минимизация дублирования**: Используйте общие утилиты и базовые классы для повторяющихся операций.
- **Фильтрация по группам**: Настройте TestNG для выборочного запуска групп (`smoke`, `regression`) через `testng.xml`.

## Единый шаблон тестов

Использование аннотаций `@Test`, `@BeforeMethod`, `@AfterMethod` в TestNG обеспечивает единообразие тестов. Шаблон AAA (Arrange-Act-Assert) или Given-When-Then применяется для структурирования кода.

### Пример кода (TestNG с единым шаблоном)

```java
import io.qameta.allure.Description;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.support.PageFactory;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import static org.testng.Assert.assertTrue;

public class LoginTest extends BaseTest {

    @BeforeMethod
    public void setUp() {
        // Инициализация окружения выполняется в BaseTest
        // Дополнительная настройка, специфичная для теста
    }

    @Test(groups = {"ui", "smoke"})
    @Description("Проверка успешного логина через UI")
    public void testSuccessfulLogin() {
        // Arrange
        LoginPage loginPage = PageFactory.initElements(driver, LoginPage.class);
        String username = "validUser";
        String password = "validPass";

        // Act
        loginPage.enterCredentials(username, password);
        boolean isLoggedIn = loginPage.submitLogin();

        // Assert
        assertTrue(isLoggedIn, "Login was not successful");
    }

    @AfterMethod
    public void tearDown() {
        // Очистка выполняется в BaseTest
        // Дополнительная очистка, если требуется
    }
}
```

### Пояснения
- **`@BeforeMethod`**: Используется для инициализации перед каждым тестом (например, открытие браузера, настройка данных).
- **`@Test`**: Содержит основную логику теста с применением шаблона AAA.
- **`@AfterMethod`**: Используется для очистки после каждого теста (например, закрытие браузера, удаление тестовых данных).
- **Группы**: Атрибут `groups` позволяет классифицировать тесты для выборочного запуска.

## Использование `BaseTest` для инициализации окружения

Класс `BaseTest` централизует общую логику инициализации и очистки окружения, что уменьшает дублирование кода и упрощает поддержку.

### Пример кода (`BaseTest`)

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import io.qameta.allure.Step;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;

public abstract class BaseTest {
    protected WebDriver driver;
    private static final Logger logger = LoggerFactory.getLogger(BaseTest.class);

    @BeforeMethod
    @Step("Инициализация WebDriver")
    public void setUp() {
        logger.info("Starting WebDriver initialization");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        driver.manage().window().maximize();
        logger.info("WebDriver initialized");
    }

    @AfterMethod
    @Step("Очистка WebDriver")
    public void tearDown() {
        if (driver != null) {
            logger.info("Closing WebDriver");
            driver.quit();
        }
    }
}
```

### Пример кода (UI-тест с использованием `BaseTest`)

```java
import io.qameta.allure.Description;
import org.openqa.selenium.support.PageFactory;
import org.testng.annotations.Test;
import static org.testng.Assert.assertTrue;

public class HomePageTest extends BaseTest {

    @Test(groups = {"ui", "smoke"})
    @Description("Проверка доступности главной страницы")
    public void testHomePageIsAccessible() {
        // Arrange
        HomePage homePage = PageFactory.initElements(driver, HomePage.class);
        driver.get("https://example.com");

        // Act
        boolean isPageLoaded = homePage.isPageDisplayed();

        // Assert
        assertTrue(isPageLoaded, "Home page is not displayed");
    }
}
```

### Пример кода (`HomePage` с Page Object)

```java
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;

import java.time.Duration;

public class HomePage {
    private final WebDriver driver;
    private final WebDriverWait wait;

    @FindBy(id = "home-page-header")
    private WebElement header;

    public HomePage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        PageFactory.initElements(driver, this);
    }

    public boolean isPageDisplayed() {
        return wait.until(ExpectedConditions.visibilityOf(header)).isDisplayed();
    }
}
```

### Лучшие практики для `BaseTest`
- **Централизация**: Храните общую инициализацию (WebDriver, RestAssured, Testcontainers) в `BaseTest`.
- **Логирование**: Используйте SLF4J с Logback для логирования этапов инициализации и очистки.
- **Гибкость**: Позвольте дочерним классам переопределять настройки при необходимости.
- **Очистка**: Гарантируйте закрытие ресурсов (браузера, контейнеров) в `@AfterMethod`.

## Интеграция с CI/CD

Для запуска тестов разных слоёв настройте CI/CD с использованием Maven и TestNG. Пример конфигурации `testng.xml` для выборочного запуска:

```xml
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="TestSuite">
    <test name="SmokeTests">
        <groups>
            <run>
                <include name="smoke"/>
            </run>
        </groups>
        <packages>
            <package name="tests.ui"/>
            <package name="tests.api"/>
        </packages>
    </test>
    <test name="RegressionTests">
        <groups>
            <run>
                <include name="regression"/>
            </run>
        </groups>
        <packages>
            <package name="tests.*"/>
        </packages>
    </test>
</suite>
```

### Пример настройки GitHub Actions

```yaml
name: Run Layered Tests
on:
  push:
    branches:
      - main
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
      - name: Run smoke tests
        run: mvn test -Dtestng.dtd.http=true -DsuiteXmlFile=testng-smoke.xml
      - name: Run regression tests
        run: mvn test -Dtestng.dtd.http=true -DsuiteXmlFile=testng-regression.xml
      - name: Generate Allure report
        run: mvn allure:report
      - name: Publish Allure report
        uses: actions/upload-artifact@v4
        with:
          name: allure-report
          path: target/site/allure-maven-plugin
```

## Дополнительно

### Использование Testcontainers для интеграционных тестов

Testcontainers упрощает настройку окружения для интеграционных тестов, запуская сервисы в Docker-контейнерах.

```java
import io.restassured.RestAssured;
import org.testcontainers.containers.GenericContainer;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class ApiIntegrationTest extends BaseTest {
    private GenericContainer<?> apiContainer;

    @BeforeMethod
    @Override
    public void setUp() {
        // Инициализация WebDriver из BaseTest, если нужно
        apiContainer = new GenericContainer<>("my-api-image:latest")
                .withExposedPorts(8080);
        apiContainer.start();
        RestAssured.baseURI = "http://" + apiContainer.getHost() + ":" + apiContainer.getMappedPort(8080);
    }

    @Test(groups = {"api", "integration"})
    public void testApiEndpoint() {
        // Arrange
        // Act
        given()
                .when()
                .get("/api/health")
                .then()
                .statusCode(200)
                .body("status", equalTo("UP"));
    }

    @AfterMethod
    @Override
    public void tearDown() {
        if (apiContainer != null) {
            apiContainer.stop();
        }
        // Вызов tearDown из BaseTest
        super.tearDown();
    }
}
```

Добавьте зависимости в `pom.xml`:

```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers</artifactId>
    <version>1.20.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>rest-assured</artifactId>
    <version>5.5.0</version>
    <scope>test</scope>
</dependency>
```

### Проверка качества кода

Используйте Checkstyle, PMD и SpotBugs для обеспечения качества тестового кода. Пример конфигурации в `pom.xml`:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-checkstyle-plugin</artifactId>
    <version>3.4.0</version>
    <configuration>
        <configLocation>checkstyle.xml</configLocation>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

### Отладка

Для отладки тестов в IntelliJ IDEA:
1. Установите точки останова в `BaseTest` или тестовых методах.
2. Используйте TestNG Run/Debug Configuration с указанием `testng.xml`.
3. Проверяйте состояние WebDriver, API-ответов или контейнеров через Variables.

## Распространённые ошибки

- **Смешивание слоёв**: Не включайте API-проверки в UI-тесты, чтобы избежать усложнения.
- **Отсутствие очистки**: Убедитесь, что `BaseTest` корректно закрывает ресурсы в `@AfterMethod`.
- **Flaky тесты**: Используйте `WebDriverWait` для UI и таймауты для API, чтобы минимизировать нестабильность.

## Источники
- [TestNG Documentation](https://testng.org/doc/documentation-main.html)
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/webdriver/)
- [RestAssured Documentation](http://rest-assured.io/)
- [Testcontainers Documentation](https://testcontainers.org/)
- [Allure Framework](https://allurereport.org/docs/)

---
[**← Назад к оглавлению**](../README.md)