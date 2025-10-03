
# Allure Reports в автоматизации: настройка, использование и интеграция

Allure Framework — мощный инструмент для создания наглядных и информативных отчётов по результатам автотестов, который упрощает анализ, отладку и демонстрацию результатов команде. Он интегрируется с JUnit 5, TestNG, Selenium, RestAssured и CI/CD-системами, предоставляя визуальные дашборды с шагами, статусами, скриншотами и статистикой. В этой статье мы разберём, зачем нужен Allure, как настроить его в IntelliJ IDEA, использовать с тестами и интегрировать с GitHub Actions для автоматизации отчётов.

## Зачем использовать Allure

Allure превращает сухие логи тестов в читаемые HTML-отчёты, которые помогают:
- **Обеспечить прозрачность**: Отчёты понятны тестировщикам, разработчикам и менеджерам благодаря визуальной подаче.
- **Ускорить анализ ошибок**: Шаги, скриншоты и логи собраны в одном месте.
- **Упростить демонстрацию**: Наглядные результаты удобно показывать на демо.
- **Отслеживать историю**: Можно анализировать стабильность тестов и выявлять flaky тесты.

Основные возможности включают визуализацию шагов, поддержку вложений (скриншоты, JSON-ответы), группировку по категориям (`Epic`, `Feature`, `Story`) и интеграцию с CI/CD.

## Настройка Allure в Maven

Для работы Allure необходимо добавить зависимости и плагин в `pom.xml`. Поддержка JUnit 5 и TestNG включается через отдельные модули, но используйте только нужный фреймворк, чтобы избежать избыточности.

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
        <!-- AssertJ for assertions -->
        <dependency>
            <groupId>org.assertj</groupId>
            <artifactId>assertj-core</artifactId>
            <version>3.24.2</version>
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

## Подключение Allure к IntelliJ IDEA

### Установка Allure CLI

- Скачайте Allure с [GitHub Releases](https://github.com/allure-framework/allure2/releases).
- Распакуйте архив и добавьте папку `bin` в переменную окружения `PATH`.
- Проверьте установку:
  ```bash
  allure --version
  ```

### Установка плагина Allure

- В IntelliJ IDEA откройте `File → Settings → Plugins → Marketplace`.
- Найдите и установите плагин **Allure Report**.
- Перезапустите IDE.

### Настройка allure.properties

Создайте файл `src/test/resources/allure.properties` для кастомизации:

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

### Генерация отчёта

- Запустите тесты:
  ```bash
  mvn clean test
  ```
- Сгенерируйте отчёт через терминал:
  ```bash
  allure generate target/allure-results --clean -o target/allure-report
  allure open target/allure-report
  ```
- Или используйте вкладку **Allure** в IntelliJ IDEA: после запуска тестов нажмите **Generate Report** для просмотра результатов в браузере.

### Содержимое `target/allure-results`

После выполнения тестов в `target/allure-results` появляются:
- JSON-файлы с результатами тестов.
- Скриншоты и вложения, если используются аннотации `@Attachment`.
- Логи, если настроено логирование (например, через SLF4J+Logback).

## Использование Allure в тестах

### Аннотации Allure
- `@Step`: Описывает шаги теста, отображаемые в отчёте.
- `@Attachment`: Добавляет вложения, такие как скриншоты или JSON-ответы.
- `@Severity`: Указывает критичность теста (`BLOCKER`, `CRITICAL`, `NORMAL`, `MINOR`, `TRIVIAL`).
- `@Description`: Добавляет описание теста.
- `@Epic`, `@Feature`, `@Story`: Организуют тесты в иерархию для крупных проектов.

```java
import io.qameta.allure.Description;
import io.qameta.allure.Epic;
import io.qameta.allure.Feature;
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
    @Epic("Авторизация")
    @Feature("Логин")
    @Description("Проверка успешного логина")
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

    @io.qameta.allure.Attachment(value = "Скриншот", type = "image/png")
    private byte[] attachScreenshot() {
        return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
    }

    @AfterEach
    void tearDown() {
        driver.quit();
    }
}
```

### Page Object

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
### TestNG: Аналогичный пример

```java
import io.qameta.allure.Description;
import io.qameta.allure.Epic;
import io.qameta.allure.Feature;
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

public class AllureCartTestNG {

    private WebDriver driver;
    private CartPage cartPage;

    @BeforeMethod
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        driver.get("https://example.com/cart");
        cartPage = new CartPage(driver);
    }

    @Test
    @Epic("Корзина")
    @Feature("Добавление товара")
    @Description("Проверка добавления товара в корзину")
    @Severity(SeverityLevel.CRITICAL)
    public void addItemToCartTest() {
        // Arrange
        String item = "Laptop";
        
        // Act
        addItemToCart(item);
        
        // Assert
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        assertThat(cartPage.isItemInCart(item)).isTrue();
        attachScreenshot();
    }

    @Step("Добавление товара {item} в корзину")
    private void addItemToCart(String item) {
        cartPage.addItem(item);
    }

    @io.qameta.allure.Attachment(value = "Скриншот", type = "image/png")
    private byte[] attachScreenshot() {
        return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
    }

    @AfterMethod
    void tearDown() {
        driver.quit();
    }
}
```

### API-тестирование с RestAssured

Для API-тестов Allure помогает документировать запросы и ответы:

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
    @Description("Проверка получения списка товаров")
    @Severity(SeverityLevel.NORMAL)
    void testGetItems() {
        // Arrange
        RestAssured.baseURI = "https://api.example.com";
        
        // Act
        Response response = getItems();
        
        // Assert
        assertThat(response.getStatusCode()).isEqualTo(200);
        attachResponse(response.getBody().asString());
    }

    @Step("Запрос списка товаров")
    private Response getItems() {
        return given()
            .when()
            .get("/items")
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

## Лучшие практики и отладка

- **Аннотации**: Используйте `@Step` для шагов, `@Attachment` для скриншотов и ответов API, `@Severity` для указания критичности (подробно описано ранее).
- **Устойчивость тестов**: Применяйте `WebDriverWait`, стабильные локаторы (`id`, `data-*`), и проверки (`isDisplayed`, `isEnabled`) для UI-тестов.
- **Retry-механизмы**: Для flaky тестов используйте `@Retry` в TestNG или кастомную логику в JUnit 5.
- **Отладка**: Устанавливайте точки останова в IntelliJ IDEA и анализируйте содержимое `target/allure-results` для диагностики.
- **Логирование**: Настройте SLF4J+Logback для записи логов, упрощающих анализ ошибок.

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

## Источники
- [Allure Framework Documentation](https://docs.qameta.io/allure/)
- [Allure Maven Integration](https://docs.qameta.io/allure/#_maven)
- [Allure IntelliJ Plugin](https://plugins.jetbrains.com/plugin/10407-allure)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [TestNG Documentation](https://testng.org/doc/documentation-main.html)
- [Allure Command Line Documentation](https://docs.qameta.io/allure/#_commandline)
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/en/)
- [RestAssured Documentation](https://rest-assured.io/)
- [Testcontainers Documentation](https://www.testcontainers.org/)

---
[**← Назад к оглавлению**](README.md)
