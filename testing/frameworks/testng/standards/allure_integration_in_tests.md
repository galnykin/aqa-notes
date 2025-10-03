# Интеграция с Allure

Allure — это мощный инструмент для создания информативных и читаемых отчётов по результатам тестов. Интеграция с TestNG позволяет использовать аннотации `@Epic`, `@Feature`, `@Story`, `@Severity` для структурирования тестов, а также `@Step` и вложения для детализации шагов и результатов. Эта статья дополняет предыдущие материалы, фокусируясь на настройке Allure, использовании аннотаций, совместимости с `@Step` и вложениями, а также генерации отчётов после прогона тестов.

## Настройка Allure в проекте

Для интеграции Allure с TestNG добавьте необходимые зависимости в `pom.xml` и настройте Maven для генерации отчётов.

### Зависимости в `pom.xml`

```xml
<dependency>
    <groupId>io.qameta.allure</groupId>
    <artifactId>allure-testng</artifactId>
    <version>2.27.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>2.0.16</version>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.5.12</version>
</dependency>
<plugin>
    <groupId>io.qameta.allure</groupId>
    <artifactId>allure-maven</artifactId>
    <version>2.12.0</version>
    <configuration>
        <reportVersion>2.27.0</reportVersion>
        <resultsDirectory>${project.basedir}/target/allure-results</resultsDirectory>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

### Пояснения
- **`allure-testng`**: Интеграция Allure с TestNG.
- **`allure-maven`**: Плагин для генерации отчётов.
- **SLF4J и Logback**: Для логирования шагов и ошибок.

## Использование аннотаций Allure

Allure предоставляет аннотации для структурирования тестов и улучшения читаемости отчётов:
- **`@Epic`**: Описывает высокоуровневую функциональность или модуль.
- **`@Feature`**: Указывает конкретную функциональность внутри эпика.
- **`@Story`**: Описывает пользовательскую историю или сценарий.
- **`@Severity`**: Указывает важность теста (например, `BLOCKER`, `CRITICAL`, `NORMAL`).

### Пример кода (TestNG с аннотациями Allure)

```java
import io.qameta.allure.Epic;
import io.qameta.allure.Feature;
import io.qameta.allure.Severity;
import io.qameta.allure.SeverityLevel;
import io.qameta.allure.Story;
import io.qameta.allure.Description;
import org.openqa.selenium.support.PageFactory;
import org.testng.annotations.Test;
import static org.testng.Assert.assertTrue;

@Epic("Авторизация пользователей")
@Feature("Веб-интерфейс авторизации")
public class UserLoginTest extends BaseTest {

    @Test(groups = {"ui", "smoke"})
    @Story("Успешная авторизация")
    @Severity(SeverityLevel.CRITICAL)
    @Description("Проверка успешной авторизации пользователя через веб-форму")
    public void testSuccessfulUserLogin() {
        // Arrange
        LoginPage loginPage = PageFactory.initElements(driver, LoginPage.class);
        driver.get("https://example.com/login");

        // Act
        loginPage.enterCredentials("validUser", "validPass");
        boolean isLoggedIn = loginPage.submitLogin();

        // Assert
        assertTrue(isLoggedIn, "User login was not successful");
    }

    @Test(groups = {"ui", "regression"})
    @Story("Обработка ошибок авторизации")
    @Severity(SeverityLevel.NORMAL)
    @Description("Проверка ошибки при вводе некорректных учетных данных")
    public void testInvalidUserLogin() {
        // Arrange
        LoginPage loginPage = PageFactory.initElements(driver, LoginPage.class);
        driver.get("https://example.com/login");

        // Act
        loginPage.enterCredentials("invalidUser", "invalidPass");
        boolean isLoggedIn = loginPage.submitLogin();

        // Assert
        assertFalse(isLoggedIn, "User login succeeded with invalid credentials");
    }
}
```

### Пояснения
- **`@Epic` и `@Feature`**: Структурируют тесты по модулям и функциональности.
- **`@Story`**: Описывает конкретный сценарий, например, успешная авторизация.
- **`@Severity`**: Указывает важность теста (`CRITICAL` для ключевых функций, `NORMAL` для регрессии).
- **`@Description`**: Добавляет подробное описание теста.

## Совместимость с `@Step` и вложениями

Аннотация `@Step` позволяет разбивать тест на шаги, а вложения (`@Attachment`) добавляют дополнительную информацию, такую как скриншоты или логи, в отчёты Allure.

### Пример кода (TestNG с `@Step` и вложениями)

```java
import io.qameta.allure.Attachment;
import io.qameta.allure.Step;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testng.annotations.Test;
import java.time.Duration;

public class LoginPageTest extends BaseTest {
    private static final Logger logger = LoggerFactory.getLogger(LoginPageTest.class);

    @Test(groups = {"ui", "smoke"})
    @Epic("Авторизация пользователей")
    @Feature("Веб-интерфейс авторизации")
    @Story("Успешная авторизация")
    @Severity(SeverityLevel.CRITICAL)
    @Description("Проверка успешной авторизации с логированием шагов")
    public void testUserLoginWithSteps() {
        // Arrange
        LoginPage loginPage = new LoginPage(driver);
        driver.get("https://example.com/login");

        // Act
        enterCredentials(loginPage, "validUser", "validPass");
        boolean isLoggedIn = submitLogin(loginPage);
        attachScreenshot();

        // Assert
        assertTrue(isLoggedIn, "User login was not successful");
    }

    @Step("Ввод учетных данных: {username}")
    private void enterCredentials(LoginPage loginPage, String username, String password) {
        logger.info("Entering credentials for user: {}", username);
        loginPage.enterCredentials(username, password);
    }

    @Step("Отправка формы логина")
    private boolean submitLogin(LoginPage loginPage) {
        logger.info("Submitting login form");
        return loginPage.submitLogin();
    }

    @Attachment(value = "Screenshot", type = "image/png")
    private byte[] attachScreenshot() {
        logger.info("Taking screenshot");
        return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
    }
}

class LoginPage {
    private final WebDriver driver;
    private final WebDriverWait wait;

    @FindBy(id = "username")
    private WebElement usernameField;

    @FindBy(id = "password")
    private WebElement passwordField;

    @FindBy(id = "login-button")
    private WebElement loginButton;

    public LoginPage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        PageFactory.initElements(driver, this);
    }

    public void enterCredentials(String username, String password) {
        wait.until(ExpectedConditions.visibilityOf(usernameField)).sendKeys(username);
        wait.until(ExpectedConditions.visibilityOf(passwordField)).sendKeys(password);
    }

    public boolean submitLogin() {
        wait.until(ExpectedConditions.elementToBeClickable(loginButton)).click();
        return driver.findElements(By.id("profile")).size() > 0;
    }
}
```

### Пояснения
- **`@Step`**: Разбивает тест на шаги, отображаемые в отчёте Allure.
- **`@Attachment`**: Добавляет скриншот в отчёт при выполнении теста.
- **Логирование**: SLF4J фиксирует выполнение шагов для отладки.

## Генерация отчётов после прогона

Allure генерирует отчёты на основе результатов тестов, сохранённых в папке `target/allure-results`.

### Команды для генерации и просмотра отчёта

1. **Запуск тестов**:
   ```bash
   mvn clean test
   ```

2. **Генерация отчёта**:
   ```bash
   mvn allure:report
   ```

3. **Просмотр отчёта**:
   ```bash
   mvn allure:serve
   ```

### Интеграция с CI/CD

Настройте CI/CD для автоматической генерации и публикации отчётов Allure.

#### Пример настройки GitHub Actions

```yaml
name: Run Tests with Allure
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
        run: mvn clean test -Dtestng.dtd.http=true
      - name: Generate Allure report
        run: mvn allure:report
      - name: Publish Allure report
        uses: actions/upload-artifact@v4
        with:
          name: allure-report
          path: target/site/allure-maven-plugin
```

#### Пример настройки Jenkins

1. Установите плагин Allure Jenkins.
2. Настройте проект с шагами:
   ```bash
   mvn clean test
   mvn allure:report
   ```
3. Настройте Allure для публикации отчёта в интерфейсе Jenkins.

## Дополнительно

### Интеграция с Testcontainers

Testcontainers можно использовать для запуска сервисов, а Allure — для документирования шагов.

```java
import io.qameta.allure.Epic;
import io.qameta.allure.Feature;
import io.qameta.allure.Severity;
import io.qameta.allure.SeverityLevel;
import io.qameta.allure.Step;
import org.testcontainers.containers.GenericContainer;
import org.testng.annotations.BeforeClass;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

@Epic("API-тестирование")
@Feature("Проверка сервисов")
public class ApiTestWithAllure extends BaseTest {
    private static GenericContainer<?> apiContainer;

    @BeforeClass
    @Step("Запуск API-контейнера")
    public void setupContainer() {
        apiContainer = new GenericContainer<>("my-api-image:latest")
                .withExposedPorts(8080);
        apiContainer.start();
        RestAssured.baseURI = "http://" + apiContainer.getHost() + ":" + apiContainer.getMappedPort(8080);
    }

    @Test(groups = {"api", "smoke"})
    @Severity(SeverityLevel.BLOCKER)
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
}
```

### Проверка качества кода

Используйте Checkstyle для проверки аннотаций Allure. Пример конфигурации в `checkstyle.xml`:

```xml
<module name="AnnotationUseStyle">
    <property name="elementType" value="method"/>
    <property name="pattern" value="^(Epic|Feature|Story|Severity|Description|Step|Attachment)$"/>
</module>
```

Добавьте в `pom.xml`:

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

Для отладки в IntelliJ IDEA:
1. Убедитесь, что Allure генерирует результаты в `target/allure-results`.
2. Установите точки останова в методах с `@Step` или тестах.
3. Проверяйте логи SLF4J и отчёты Allure для анализа.

## Распространённые ошибки

- **Отсутствие результатов**: Убедитесь, что папка `target/allure-results` создаётся после тестов.
- **Неправильные аннотации**: Проверяйте корректность использования `@Epic`, `@Feature`, `@Story`, `@Severity`.
- **Проблемы с вложениями**: Убедитесь, что метод `@Attachment` возвращает правильный тип данных (например, `byte[]` для скриншотов).

## Источники
- [Allure Framework Documentation](https://allurereport.org/docs/)
- [TestNG Documentation](https://testng.org/doc/documentation-main.html)
- [Testcontainers Documentation](https://testcontainers.org/)
- [Checkstyle Documentation](https://checkstyle.sourceforge.io/)

---
[**← Назад к оглавлению**](../README.md)