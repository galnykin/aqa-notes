# Использование Allure для создания отчётов в автоматизированном тестировании

Allure Framework — это мощный инструмент для создания читаемых и информативных отчётов о результатах автоматизированных тестов. Он интегрируется с JUnit 5, TestNG, Selenium, RestAssured и другими инструментами, предоставляя визуальное представление тестов, включая шаги, вложения и статистику. Эта статья описывает настройку Allure, использование аннотаций, интеграцию с Maven и CI/CD, а также лучшие практики для повышения качества отчётов.

## Настройка Allure с Maven

Для использования Allure необходимо добавить зависимости в `pom.xml` и настроить плагин для генерации отчётов. Allure поддерживает JUnit 5 и TestNG через отдельные модули.

### Зависимости в `pom.xml`

Добавьте следующие зависимости и плагин в `pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>allure-aqa</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <allure.version>2.25.0</allure.version>
        <aspectj.version>1.9.21</aspectj.version>
    </properties>

    <dependencies>
        <!-- Allure JUnit 5 -->
        <dependency>
            <groupId>io.qameta.allure</groupId>
            <artifactId>allure-junit5</artifactId>
            <version>${allure.version}</version>
            <scope>test</scope>
        </dependency>
        <!-- Allure TestNG -->
        <dependency>
            <groupId>io.qameta.allure</groupId>
            <artifactId>allure-testng</artifactId>
            <version>${allure.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.2.3</version>
                <configuration>
                    <argLine>
                        -javaagent:${settings.localRepository}/org/aspectj/aspectjweaver/${aspectj.version}/aspectjweaver-${aspectj.version}.jar
                    </argLine>
                </configuration>
                <dependencies>
                    <dependency>
                        <groupId>org.aspectj</groupId>
                        <artifactId>aspectjweaver</artifactId>
                        <version>${aspectj.version}</version>
                    </dependency>
                </dependencies>
            </plugin>
            <plugin>
                <groupId>io.qameta.allure</groupId>
                <artifactId>allure-maven</artifactId>
                <version>2.12.0</version>
                <configuration>
                    <reportVersion>${allure.version}</reportVersion>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

## Настройка Allure через allure.properties

Для кастомизации поведения Allure Framework можно использовать файл конфигурации `allure.properties`, размещённый в директории `src/test/resources`. Этот файл позволяет задавать параметры, такие как директория для результатов тестов, формат отчётов и другие настройки.

### Пример файла `allure.properties`

Создайте файл `src/test/resources/allure.properties` со следующими настройками:

```properties
# Указывает директорию для хранения результатов тестов
allure.results.directory=target/allure-results

# Включает скриншоты для UI-тестов (true/false)
allure.attachments.screenshots=true

# Уровень логирования (INFO, DEBUG, WARN, ERROR)
allure.logging.level=INFO

# Формат отчёта (json, xml)
allure.results.format=json

# Включение автоматического создания категорий для отчётов
allure.categories.enabled=true
```

### Основные настройки
- **`allure.results.directory`**: Указывает, где сохраняются результаты тестов (по умолчанию `target/allure-results`). Это важно для интеграции с CI/CD, чтобы отчёты были доступны для последующей обработки.
- **`allure.attachments.screenshots`**: Включает автоматическое прикрепление скриншотов для UI-тестов при использовании Allure с Selenium.
- **`allure.logging.level`**: Устанавливает уровень логирования для отладки. `DEBUG` полезен при анализе проблем.
- **`allure.results.format`**: Определяет формат результатов (`json` или `xml`). JSON является стандартом для большинства проектов.
- **`allure.categories.enabled`**: Позволяет группировать тесты по категориям (например, по типам ошибок) для более читаемых отчётов.

### Лучшие практики
- Размещайте `allure.properties` в `src/test/resources`, чтобы он автоматически подхватывался Maven.
- Используйте относительные пути для `allure.results.directory`, чтобы обеспечить переносимость проекта.
- Для CI/CD (например, GitHub Actions) убедитесь, что директория результатов (`target/allure-results`) архивируется для последующей генерации отчёта.

### Генерация отчётов

После запуска тестов (`mvn clean test`) сгенерируйте отчёт командой:

```bash
mvn allure:serve
```

Эта команда создаёт временный сервер с отчётом Allure, доступным в браузере. Для постоянного хранения используйте `mvn allure:report`.

## Использование аннотаций Allure

Allure предоставляет аннотации для улучшения читаемости отчётов, такие как `@Step`, `@Attachment`, `@Severity`, `@Description` и другие.

### Пример с JUnit 5

Рассмотрим тест UI с использованием Selenium и Allure.

```java
import io.qameta.allure.Description;
import io.qameta.allure.Severity;
import io.qameta.allure.SeverityLevel;
import io.qameta.allure.Step;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;
import static org.assertj.core.api.Assertions.assertThat;

public class AllureLoginTest {

    private WebDriver driver;
    private LoginPage loginPage;

    @BeforeEach
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        driver.get("https://example.com/login");
        loginPage = new LoginPage(driver);
    }

    @Test
    @Description("Тест успешного логина с использованием Allure")
    @Severity(SeverityLevel.CRITICAL)
    void testSuccessfulLogin() {
        // Arrange
        String username = "testuser";
        String password = "password123";
        
        // Act
        loginStep(username, password);
        
        // Assert
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        assertThat(wait.until(d -> d.getCurrentUrl())).contains("/dashboard");
        attachScreenshot();
    }

    @Step("Выполнение логина с пользователем {username}")
    private void loginStep(String username, String password) {
        loginPage.login(username, password);
    }

    @Attachment(value = "Скриншот", type = "image/png")
    private byte[] attachScreenshot() {
        return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
    }

    @AfterEach
    void tearDown() {
        driver.quit();
    }
}
```

### Page Object для теста

```java
import lombok.Getter;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;

@Getter
public class LoginPage {
    private final WebDriver driver;

    @FindBy(id = "username")
    private WebElement usernameField;

    @FindBy(id = "password")
    private WebElement passwordField;

    @FindBy(id = "login-button")
    private WebElement loginButton;

    public LoginPage(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this);
    }

    public void login(String username, String password) {
        usernameField.sendKeys(username);
        passwordField.sendKeys(password);
        loginButton.click();
    }
}
```

### Пример с TestNG

Аналогичный тест с TestNG и Allure:

```java
import io.qameta.allure.Description;
import io.qameta.allure.Severity;
import io.qameta.allure.SeverityLevel;
import io.qameta.allure.Step;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import java.time.Duration;
import static org.assertj.core.api.Assertions.assertThat;

public class AllureLoginTestNG {

    private WebDriver driver;
    private LoginPage loginPage;

    @BeforeMethod
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        driver.get("https://example.com/login");
        loginPage = new LoginPage(driver);
    }

    @Test
    @Description("Тест успешного логина с использованием Allure")
    @Severity(SeverityLevel.CRITICAL)
    public void testSuccessfulLogin() {
        // Arrange
        String username = "testuser";
        String password = "password123";
        
        // Act
        loginStep(username, password);
        
        // Assert
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        assertThat(wait.until(d -> d.getCurrentUrl())).contains("/dashboard");
        attachScreenshot();
    }

    @Step("Выполнение логина с пользователем {username}")
    private void loginStep(String username, String password) {
        loginPage.login(username, password);
    }

    @Attachment(value = "Скриншот", type = "image/png")
    private byte[] attachScreenshot() {
        return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
    }

    @AfterMethod
    void tearDown() {
        driver.quit();
    }
}
```

### Аннотации Allure
- `@Step`: Описывает шаги теста, отображаемые в отчёте.
- `@Attachment`: Добавляет вложения, такие как скриншоты или JSON-ответы.
- `@Severity`: Указывает критичность теста (`BLOCKER`, `CRITICAL`, `NORMAL`, `MINOR`, `TRIVIAL`).
- `@Description`: Добавляет описание теста.
- `@Epic`, `@Feature`, `@Story`: Организуют тесты в иерархию для крупных проектов.

## API-тестирование с Allure и RestAssured

Allure улучшает отчёты для API-тестов, добавляя информацию о запросах и ответах. Пример теста с RestAssured:

```java
import io.qameta.allure.Attachment;
import io.qameta.allure.Step;
import io.restassured.RestAssured;
import io.restassured.response.Response;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.assertj.core.api.Assertions.assertThat;

public class AllureApiTest {

    @Test
    void testApiWithAllure() {
        // Arrange
        RestAssured.baseURI = "https://api.example.com";
        
        // Act
        Response response = getUsers();
        
        // Assert
        assertThat(response.getStatusCode()).isEqualTo(200);
        attachResponse(response.getBody().asString());
    }

    @Step("Получение списка пользователей")
    private Response getUsers() {
        return given()
            .when()
            .get("/users")
            .then()
            .extract()
            .response();
    }

    @Attachment(value = "Ответ API", type = "text/plain")
    private String attachResponse(String responseBody) {
        return responseBody;
    }
}
```

### Лучшие практики
- Используйте `@Step` для каждого запроса API, чтобы отслеживать шаги.
- Прикрепляйте тело запроса и ответа с помощью `@Attachment`.
- Логируйте действия с SLF4J+Logback для отладки.

## Дополнительно

### Testcontainers для интеграционного тестирования

Testcontainers позволяет запускать сервисы (например, API) в Docker-контейнерах для тестирования. Пример интеграции с Allure:

```java
import io.qameta.allure.Step;
import org.junit.jupiter.api.Test;
import org.testcontainers.containers.GenericContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;
import static org.assertj.core.api.Assertions.assertThat;

@Testcontainers
public class AllureContainerTest {

    @Container
    private GenericContainer<?> apiContainer = new GenericContainer<>("httpd:2.4")
        .withExposedPorts(80);

    @Test
    void testApiContainer() {
        // Arrange
        String address = "http://" + apiContainer.getHost() + ":" + apiContainer.getFirstMappedPort();
        
        // Act
        boolean isRunning = checkContainerStatus();
        
        // Assert
        assertThat(isRunning).isTrue();
    }

    @Step("Проверка статуса контейнера")
    private boolean checkContainerStatus() {
        return apiContainer.isRunning();
    }
}
```

### Retry-механизмы для flaky тестов

Flaky тесты можно стабилизировать с помощью механизма повторных попыток. Пример для TestNG:

```java
import io.qameta.allure.Description;
import org.testng.annotations.Test;
import org.testng.annotations.RetryAnalyzer;
import org.testng.annotations.Retry;

public class FlakyAllureTest {

    @Test(retryAnalyzer = Retry.class)
    @Description("Тест с повторными попытками")
    public void testFlakyWithAllure() {
        // Логика теста
    }
}
```

### CI/CD с GitHub Actions

Настройка GitHub Actions для запуска тестов и генерации отчёта Allure:

```yaml
name: CI with Allure
on:
  push:
    branches:
      - main
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
      - name: Build and test
        run: mvn clean test
      - name: Generate Allure report
        run: mvn allure:report
      - name: Upload Allure report
        uses: actions/upload-artifact@v3
        with:
          name: allure-report
          path: target/site/allure-maven-plugin
```

### Отладка тестов с Allure

- Используйте IntelliJ IDEA для установки точек останова в тестовом коде.
- Проверяйте логи SLF4J+Logback для анализа ошибок.
- Анализируйте отчёты Allure для выявления причин сбоев (скриншоты, логи, ответы API).

### Устойчивость тестов
- Используйте стабильные локаторы (`id`, `data-*`) для UI-тестов.
- Применяйте `WebDriverWait` или `FluentWait` для ожидания элементов.
- Проверяйте состояние элементов (`isDisplayed`, `isEnabled`) перед взаимодействием.

## Источники
- [Allure Framework Documentation](https://docs.qameta.io/allure/)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [TestNG Documentation](https://testng.org/doc/documentation-main.html)
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/en/)
- [RestAssured Documentation](https://rest-assured.io/)
- [Testcontainers Documentation](https://www.testcontainers.org/)

---
[**← Назад к оглавлению**](../README.md)
