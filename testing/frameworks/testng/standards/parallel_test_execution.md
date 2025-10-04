# Параллельный запуск тестов

Параллельный запуск тестов в TestNG с использованием атрибутов `parallel="methods"` или `parallel="classes"` позволяет значительно ускорить выполнение тестового набора. Однако это требует правильной настройки `thread-count` и обеспечения изоляции окружения, особенно при работе с WebDriver и API, чтобы гарантировать потокобезопасность. Эта статья дополняет предыдущие материалы, фокусируясь на настройке параллельного выполнения, контроле количества потоков и обеспечении потокобезопасности.

## Настройка параллельного запуска

TestNG поддерживает параллельное выполнение тестов через атрибут `parallel` в `testng.xml`. Доступные режимы:
- **`parallel="methods"`**: Каждый тестовый метод выполняется в отдельном потоке.
- **`parallel="classes"`**: Каждый тестовый класс выполняется в отдельном потоке.
- **`thread-count`**: Ограничивает количество параллельных потоков.

### Пример `testng.xml` с параллельным запуском

```xml
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="ParallelTestSuite" parallel="methods" thread-count="3">
    <test name="UITests">
        <parameter name="env" value="qa"/>
        <groups>
            <run>
                <include name="ui"/>
                <include name="smoke"/>
            </run>
        </groups>
        <classes>
            <class name="com.example.tests.ui.UserLoginTest"/>
            <class name="com.example.tests.ui.ProductCartTest"/>
        </classes>
    </test>
    <test name="APITests">
        <parameter name="env" value="prod"/>
        <groups>
            <run>
                <include name="api"/>
            </run>
        </groups>
        <classes>
            <class name="com.example.tests.api.UserApiTest"/>
        </classes>
    </test>
    <listeners>
        <listener class-name="com.example.tests.listeners.ParallelTestListener"/>
    </listeners>
</suite>
```

### Пояснения
- **`parallel="methods"`**: Каждый метод в классах выполняется в отдельном потоке.
- **`thread-count="3"`**: Ограничивает выполнение до трёх параллельных потоков.
- **Группы и классы**: Тесты разделены по слоям (`ui`, `api`) для логической организации.
- **Слушатели**: Используются для мониторинга параллельного выполнения.

### Лучшие практики
- **Выбор режима**: Используйте `parallel="methods"` для мелкозернистого параллелизма, если тесты независимы. Используйте `parallel="classes"` для тестов, требующих общей инициализации в классе.
- **Ограничение потоков**: Устанавливайте `thread-count` с учётом ресурсов CI/CD-сервера (например, 2–4 потока для стандартных серверов).
- **Мониторинг**: Настройте логирование для отслеживания выполнения потоков.

## Контроль thread-count и изоляции окружения

Для управления параллельным выполнением важно правильно настроить `thread-count` и обеспечить изоляцию окружения, чтобы избежать конфликтов между тестами.

### Контроль `thread-count`
- **Слишком большое значение**: Может привести к перегрузке ресурсов (CPU, память) или конфликтам (например, с общими ресурсами).
- **Слишком малое значение**: Замедлит выполнение тестов.
- **Рекомендация**: Начните с `thread-count=2` и увеличивайте, тестируя производительность на CI/CD.

### Изоляция окружения
- **UI-тесты**: Каждый тест должен использовать отдельный экземпляр WebDriver.
- **API-тесты**: RestAssured должен быть настроен без глобальных состояний.
- **Базы данных/сервисы**: Используйте Testcontainers для изолированных контейнеров.

### Пример кода (TestNG с изолированным WebDriver)

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import io.qameta.allure.Description;
import io.qameta.allure.Epic;
import io.qameta.allure.Feature;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import static org.testng.Assert.assertTrue;

@Epic("Авторизация пользователей")
@Feature("Веб-интерфейс авторизации")
public class UserLoginTest {
    private static final Logger logger = LoggerFactory.getLogger(UserLoginTest.class);
    private WebDriver driver;

    @BeforeMethod
    public void setup() {
        logger.info("Initializing WebDriver for thread {}", Thread.currentThread().getId());
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        driver.manage().window().maximize();
    }

    @Test(groups = {"ui", "smoke"})
    @Description("Проверка успешной авторизации")
    public void testSuccessfulUserLogin() {
        // Arrange
        LoginPage loginPage = new LoginPage(driver);
        driver.get("https://example.com/login");

        // Act
        loginPage.enterCredentials("validUser", "validPass");
        boolean isLoggedIn = loginPage.submitLogin();

        // Assert
        assertTrue(isLoggedIn, "User login was not successful");
    }

    @AfterMethod
    public void tearDown() {
        logger.info("Closing WebDriver for thread {}", Thread.currentThread().getId());
        if (driver != null) {
            driver.quit();
        }
    }
}
```

### Пояснения
- **Изоляция WebDriver**: Каждый тест создаёт и закрывает собственный экземпляр `WebDriver`, избегая конфликтов.
- **Логирование**: SLF4J фиксирует идентификатор потока для отслеживания.
- **Page Object**: `LoginPage` инкапсулирует взаимодействие с UI.

## Учёт потокобезопасности при работе с WebDriver и API

Параллельное выполнение требует особого внимания к потокобезопасности, чтобы избежать конфликтов при доступе к общим ресурсам.

### Потокобезопасность для WebDriver
- **Проблема**: Общий экземпляр `WebDriver` между потоками может вызывать ошибки (например, элементы не найдены).
- **Решение**: Создавайте новый `WebDriver` в `@BeforeMethod` и закрывайте в `@AfterMethod`.

### Потокобезопасность для API
- **Проблема**: Глобальные конфигурации RestAssured (например, `baseURI`) могут конфликтовать между тестами.
- **Решение**: Настраивайте RestAssured для каждого теста или используйте Testcontainers для изолированных сервисов.

### Пример кода (API-тест с Testcontainers)

```java
import io.qameta.allure.Description;
import io.qameta.allure.Epic;
import io.qameta.allure.Feature;
import org.testcontainers.containers.GenericContainer;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

@Epic("API-тестирование")
@Feature("Проверка сервисов")
public class UserApiTest {
    private static final Logger logger = LoggerFactory.getLogger(UserApiTest.class);
    private GenericContainer<?> apiContainer;

    @BeforeMethod
    public void setup() {
        logger.info("Starting API container for thread {}", Thread.currentThread().getId());
        apiContainer = new GenericContainer<>("my-api-image:latest")
                .withExposedPorts(8080);
        apiContainer.start();
        RestAssured.baseURI = "http://" + apiContainer.getHost() + ":" + apiContainer.getMappedPort(8080);
    }

    @Test(groups = {"api", "smoke"})
    @Description("Проверка эндпоинта здоровья API")
    public void testApiHealthCheck() {
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
    public void tearDown() {
        logger.info("Stopping API container for thread {}", Thread.currentThread().getId());
        if (apiContainer != null) {
            apiContainer.stop();
        }
    }
}
```

### Пояснения
- **Testcontainers**: Каждый тест запускает отдельный контейнер, обеспечивая изоляцию.
- **RestAssured**: `baseURI` настраивается для каждого теста, избегая конфликтов.
- **Логирование**: Фиксирует действия для каждого потока.

## Интеграция с CI/CD

Для параллельного запуска тестов настройте Maven и TestNG в CI/CD.

### Пример настройки GitHub Actions

```yaml
name: Run Parallel Tests
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
      - name: Run parallel tests
        run: mvn test -Dtestng.dtd.http=true -DsuiteXmlFile=testng.xml
      - name: Generate Allure report
        run: mvn allure:report
      - name: Publish Allure report
        uses: actions/upload-artifact@v4
        with:
          name: allure-report
          path: target/site/allure-maven-plugin
```

## Дополнительно

### Интеграция с Allure

Allure поддерживает параллельное выполнение, отображая результаты в отчётах.

```java
import io.qameta.allure.Attachment;
import io.qameta.allure.Description;
import io.qameta.allure.Epic;
import io.qameta.allure.Feature;
import io.qameta.allure.Step;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.testng.annotations.Test;

@Epic("Авторизация пользователей")
@Feature("Веб-интерфейс авторизации")
public class AllureParallelTest extends BaseTest {
    private static final Logger logger = LoggerFactory.getLogger(AllureParallelTest.class);

    @Test(groups = {"ui", "regression"})
    @Description("Проверка формы логина")
    public void testLoginForm() {
        // Arrange
        LoginPage loginPage = new LoginPage(driver);
        driver.get("https://example.com/login");

        // Act
        enterCredentials(loginPage, "validUser", "validPass");
        boolean isLoggedIn = submitLogin(loginPage);
        attachScreenshot();

        // Assert
        assertTrue(isLoggedIn, "Login failed");
    }

    @Step("Ввод учетных данных: {username}")
    private void enterCredentials(LoginPage loginPage, String username, String password) {
        logger.info("Entering credentials for thread {}", Thread.currentThread().getId());
        loginPage.enterCredentials(username, password);
    }

    @Step("Отправка формы логина")
    private boolean submitLogin(LoginPage loginPage) {
        logger.info("Submitting login form for thread {}", Thread.currentThread().getId());
        return loginPage.submitLogin();
    }

    @Attachment(value = "Screenshot", type = "image/png")
    private byte[] attachScreenshot() {
        logger.info("Taking screenshot for thread {}", Thread.currentThread().getId());
        return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
    }
}
```

### Проверка качества кода

Используйте SpotBugs для проверки потокобезопасности. Пример конфигурации в `pom.xml`:

```xml
<plugin>
    <groupId>com.github.spotbugs</groupId>
    <artifactId>spotbugs-maven-plugin</artifactId>
    <version>4.8.6.4</version>
    <executions>
        <execution>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

Запуск:
```bash
mvn spotbugs:check
```

### Отладка

Для отладки в IntelliJ IDEA:
1. Убедитесь, что `testng.xml` настроен с `parallel` и `thread-count`.
2. Установите точки останова в `@BeforeMethod`, `@Test` или `@AfterMethod`.
3. Проверяйте логи SLF4J с идентификаторами потоков.

## Распространённые ошибки

- **Конфликты WebDriver**: Используйте изолированные экземпляры `WebDriver` для каждого теста.
- **Глобальные конфигурации RestAssured**: Настраивайте `baseURI` для каждого теста.
- **Перегрузка ресурсов**: Ограничивайте `thread-count` в зависимости от возможностей CI/CD.

## Источники
- [TestNG Parallel Execution Documentation](https://testng.org/doc/documentation-main.html#parallel-tests)
- [Allure Framework](https://allurereport.org/docs/)
- [Testcontainers Documentation](https://testcontainers.org/)
- [SpotBugs Documentation](https://spotbugs.github.io/)

---
[**← Назад к оглавлению**](../README.md)