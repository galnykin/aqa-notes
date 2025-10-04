# Конфигурационные методы в тестах

Конфигурационные методы в TestNG, такие как `@BeforeSuite`, `@BeforeTest`, `@BeforeClass`, `@BeforeMethod` и соответствующие `@After*`, обеспечивают структурированную инициализацию и очистку окружения для тестов. Правильное использование этих аннотаций позволяет разделить настройку окружения и логику тестов, а также обеспечить корректную очистку и логирование. Эта статья дополняет предыдущие материалы, фокусируясь на чётком назначении конфигурационных методов и лучших практиках их применения.

## Назначение конфигурационных методов

Каждый конфигурационный метод имеет специфическую область применения, определяемую его местом в иерархии выполнения TestNG.

### `@BeforeSuite` и `@AfterSuite`
- **Назначение**: Выполняются один раз перед началом и после завершения всей тестовой свиты (suite), определённой в `testng.xml`.
- **Использование**:
  - Инициализация глобальных ресурсов, таких как запуск серверов, настройка Testcontainers или загрузка конфигураций.
  - Очистка глобальных ресурсов после всех тестов.
- **Пример сценария**: Запуск Docker-контейнера с базой данных перед тестами и его остановка после завершения.

### `@BeforeTest` и `@AfterTest`
- **Назначение**: Выполняются перед и после каждого тестового набора (`<test>` в `testng.xml`).
- **Использование**:
  - Настройка окружения для группы тестов, например, инициализация специфичных данных.
  - Очистка данных, созданных для набора тестов.
- **Пример сценария**: Подготовка тестовых данных для API-тестов в рамках одного набора.

### `@BeforeClass` и `@AfterClass`
- **Назначение**: Выполняются один раз перед первым и после последнего тестового метода в классе.
- **Использование**:
  - Инициализация ресурсов, общих для всех тестов в классе, например, настройка WebDriver или RestAssured.
  - Очистка ресурсов, специфичных для класса.
- **Пример сценария**: Инициализация WebDriver для UI-тестов в классе.

### `@BeforeMethod` и `@AfterMethod`
- **Назначение**: Выполняются перед и после каждого тестового метода (`@Test`) в классе.
- **Использование**:
  - Подготовка окружения для конкретного теста, например, открытие браузера на определённой странице.
  - Очистка после теста, например, сброс состояния страницы или удаление тестовых данных.
- **Пример сценария**: Очистка кэша браузера после каждого UI-теста.

### Лучшие практики
- **Чёткое разделение**: Не включайте логику тестов в конфигурационные методы, используйте их только для настройки и очистки.
- **Минимизация дублирования**: Переносите общую инициализацию в `BaseTest` или более высокоуровневые аннотации (`@BeforeSuite`, `@BeforeClass`).
- **Логирование**: Используйте SLF4J с Logback для записи этапов инициализации и очистки.

## Пример кода (TestNG с конфигурационными методами)

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import io.qameta.allure.Description;
import io.qameta.allure.Step;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testng.annotations.AfterClass;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeClass;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.BeforeSuite;
import org.testng.annotations.BeforeTest;
import org.testng.annotations.Test;
import static org.testng.Assert.assertTrue;

public class ConfigMethodsTest {
    private static final Logger logger = LoggerFactory.getLogger(ConfigMethodsTest.class);
    protected WebDriver driver;

    @BeforeSuite
    @Step("Инициализация глобальных ресурсов")
    public void setupSuite() {
        logger.info("Setting up suite resources");
        // Например, запуск Testcontainers или загрузка конфигурации
    }

    @BeforeTest
    @Step("Подготовка тестового набора")
    public void setupTest() {
        logger.info("Setting up test resources");
        // Например, инициализация тестовых данных
    }

    @BeforeClass
    @Step("Инициализация WebDriver для класса")
    public void setupClass() {
        logger.info("Setting up WebDriver");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        driver.manage().window().maximize();
    }

    @BeforeMethod
    @Step("Подготовка окружения для теста")
    public void setupMethod() {
        logger.info("Setting up test method");
        driver.get("https://example.com");
    }

    @Test(groups = {"ui", "smoke"})
    @Description("Проверка загрузки главной страницы")
    public void testHomePageLoad() {
        // Arrange
        HomePage homePage = new HomePage(driver);
        // Act
        boolean isPageLoaded = homePage.isPageDisplayed();
        // Assert
        assertTrue(isPageLoaded, "Home page failed to load");
    }

    @AfterMethod
    @Step("Очистка после теста")
    public void tearDownMethod() {
        logger.info("Cleaning up after test method");
        // Например, очистка кэша браузера
        driver.manage().deleteAllCookies();
    }

    @AfterClass
    @Step("Очистка WebDriver")
    public void tearDownClass() {
        logger.info("Cleaning up WebDriver");
        if (driver != null) {
            driver.quit();
        }
    }
}
```

### Пример кода (`HomePage` с Page Object)

```java
import org.openqa.selenium.WebDriver;
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

### Пояснения
- **`@BeforeSuite`**: Используется для глобальной настройки, например, запуска Testcontainers.
- **`@BeforeTest`**: Подготавливает данные для тестового набора.
- **`@BeforeClass`**: Инициализирует WebDriver для всех тестов в классе.
- **`@BeforeMethod`**: Открывает начальную страницу перед каждым тестом.
- **`@AfterMethod`**: Очищает состояние браузера после теста.
- **`@AfterClass`**: Закрывает WebDriver после всех тестов.
- **Логирование**: SLF4J с Logback фиксирует этапы выполнения.

## Интеграция с CI/CD

Для запуска тестов с конфигурационными методами настройте Maven и TestNG в CI/CD.

### Пример настройки GitHub Actions

```yaml
name: Run Tests with Config Methods
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
      - name: Run tests
        run: mvn test -Dtestng.dtd.http=true
      - name: Generate Allure report
        run: mvn allure:report
      - name: Publish Allure report
        uses: actions/upload-artifact@v4
        with:
          name: allure-report
          path: target/site/allure-maven-plugin
```

### Пример `testng.xml`

```xml
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="ConfigTestSuite">
    <test name="UITests">
        <classes>
            <class name="ConfigMethodsTest"/>
        </classes>
    </test>
</suite>
```

## Дополнительно

### Интеграция с Testcontainers

Testcontainers можно использовать в `@BeforeSuite` для запуска сервисов, необходимых для всех тестов.

```java
import org.testcontainers.containers.GenericContainer;
import org.testng.annotations.BeforeSuite;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class ContainerConfigTest {
    private static GenericContainer<?> apiContainer;

    @BeforeSuite
    @Step("Запуск API-контейнера")
    public void setupSuite() {
        apiContainer = new GenericContainer<>("my-api-image:latest")
                .withExposedPorts(8080);
        apiContainer.start();
        RestAssured.baseURI = "http://" + apiContainer.getHost() + ":" + apiContainer.getMappedPort(8080);
    }

    @Test(groups = {"api", "smoke"})
    @Description("Проверка эндпоинта здоровья")
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

### Использование Allure для логирования

Allure улучшает отчётность, отображая этапы конфигурационных методов.

```java
import io.qameta.allure.Step;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;

public class AllureConfigTest extends BaseTest {

    @BeforeMethod
    @Step("Подготовка теста: открытие страницы логина")
    public void setupMethod() {
        driver.get("https://example.com/login");
    }

    @Test(groups = {"ui", "regression"})
    @Description("Проверка формы логина")
    public void testLogin() {
        // Arrange
        LoginPage loginPage = new LoginPage(driver);
        // Act
        loginPage.enterCredentials("validUser", "validPass");
        boolean isLoggedIn = loginPage.submitLogin();
        // Assert
        assertTrue(isLoggedIn, "Login failed");
    }
}
```

### Проверка качества кода

Используйте Checkstyle для проверки конфигурационных методов. Пример конфигурации в `pom.xml`:

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

## Распространённые ошибки

- **Смешивание логики**: Не включайте проверки или действия теста в `@Before*` методы.
- **Неправильный уровень инициализации**: Используйте подходящий уровень аннотаций (например, `@BeforeSuite` для глобальных ресурсов, а не `@BeforeMethod`).
- **Отсутствие очистки**: Всегда закрывайте ресурсы в `@After*` методах, чтобы избежать утечек.

## Источники
- [TestNG Configuration Documentation](https://testng.org/doc/documentation-main.html#annotations)
- [Allure Framework](https://allurereport.org/docs/)
- [Testcontainers Documentation](https://testcontainers.org/)
- [Checkstyle Documentation](https://checkstyle.sourceforge.io/)

---
[**← Назад к оглавлению**](../README.md)