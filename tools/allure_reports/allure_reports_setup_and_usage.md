# Allure Reports в автоматизации: зачем нужны, как подключить к IntelliJ IDEA и использовать с тестами

Allure Framework — это инструмент для создания визуальных и информативных отчётов по результатам автотестов, который помогает тестировщикам, разработчикам и менеджерам анализировать результаты. Эта статья объясняет, зачем нужен Allure, как подключить его к IntelliJ IDEA и использовать в тестах с JUnit 5, TestNG, Selenium и RestAssured.

## Что такое Allure Report

Allure — это фреймворк для генерации HTML-отчётов, который преобразует результаты тестов в наглядные дашборды.

### Основные возможности:
- Визуализация шагов теста с поддержкой вложенных действий.
- Отображение статусов: `passed`, `failed`, `skipped`.
- Прикрепление скриншотов, логов и других вложений.
- Группировка тестов по категориям (`Epic`, `Feature`, `Story`) и уровням критичности (`Severity`).
- Интеграция с CI/CD (GitHub Actions, Jenkins).
- Поддержка JUnit 5, TestNG, Selenium, RestAssured и других библиотек.

## Зачем использовать Allure

- **Прозрачность**: Отчёты понятны не только техническим специалистам, но и менеджерам благодаря визуальной подаче.
- **Анализ ошибок**: Скриншоты, логи и шаги теста в одном месте ускоряют диагностику проблем.
- **Демонстрация результатов**: Удобно для презентации прогресса на демо.
- **История запусков**: Позволяет отслеживать стабильность тестов и выявлять flaky тесты.

## Подключение Allure к IntelliJ IDEA

### 1. Настройка Maven

Добавьте зависимости для Allure в `pom.xml`. Для поддержки JUnit 5 и TestNG включите соответствующие модули. Зависимость AspectJ необходима для перехвата событий тестов.

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
        <!-- JUnit 5 -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.10.2</version>
            <scope>test</scope>
        </dependency>
        <!-- TestNG -->
        <dependency>
            <groupId>org.testng</groupId>
            <artifactId>testng</artifactId>
            <version>7.8.0</version>
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

### 2. Установка Allure CLI

- Скачайте Allure с [GitHub Releases](https://github.com/allure-framework/allure2/releases).
- Распакуйте архив и добавьте папку `bin` в переменную окружения `PATH`.
- Проверьте установку командой:
  ```bash
  allure --version
  ```

### 3. Настройка IntelliJ IDEA

- Установите плагин **Allure Report** через `File → Settings → Plugins → Marketplace`.
- После запуска тестов (`mvn clean test`) результаты сохраняются в `target/allure-results`.
- Откройте вкладку **Allure Report** в IntelliJ IDEA для просмотра отчёта или сгенерируйте его командой:
  ```bash
  mvn allure:serve
  ```
- Плагин позволяет просматривать отчёты непосредственно в IDE, включая шаги, вложения и статистику.

### 4. Настройка allure.properties

Создайте файл `src/test/resources/allure.properties` для кастомизации:

```properties
allure.results.directory=target/allure-results
allure.attachments.screenshots=true
allure.logging.level=INFO
allure.results.format=json
allure.categories.enabled=true
```

- `allure.results.directory`: Задаёт директорию для результатов тестов.
- `allure.attachments.screenshots`: Включает прикрепление скриншотов для UI-тестов.
- `allure.logging.level`: Уровень логирования (`INFO`, `DEBUG`, `WARN`, `ERROR`).
- `allure.results.format`: Формат результатов (`json` или `xml`).
- `allure.categories.enabled`: Включает группировку тестов по категориям.

## Использование Allure в тестах

### Аннотации для структурирования

Allure предоставляет аннотации для улучшения отчётов. Пример UI-теста с Selenium и JUnit 5:

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

public class AllureCartTest {

    private WebDriver driver;
    private CartPage cartPage;

    @BeforeEach
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
    void addItemToCartTest() {
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
public class CartPage {
    private final WebDriver driver;

    @FindBy(id = "add-item")
    private WebElement addItemButton;

    @FindBy(id = "cart-items")
    private WebElement cartItems;

    public CartPage(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this);
    }

    public void addItem(String item) {
        addItemButton.sendKeys(item);
        addItemButton.submit();
    }

    public boolean isItemInCart(String item) {
        return cartItems.getText().contains(item);
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

### Лучшие практики
- Используйте `@Step` для описания ключевых действий в тесте.
- Прикрепляйте скриншоты и ответы API с помощью `@Attachment`.
- Применяйте `WebDriverWait` для UI-тестов, чтобы избежать flaky тестов, и проверяйте элементы (`isDisplayed`, `isEnabled`).
- Используйте стабильные локаторы (`id`, `data-*`) для Selenium.

## Интеграция с CI/CD

Allure легко интегрируется с CI/CD системами. Пример для GitHub Actions:

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
      - name: Run tests
        run: mvn clean test
      - name: Generate Allure report
        run: mvn allure:report
      - name: Upload Allure report
        uses: actions/upload-artifact@v3
        with:
          name: allure-report
          path: target/site/allure-maven-plugin
```

Для Jenkins используйте плагин Allure Jenkins Plugin и команды `mvn allure:report` или `mvn allure:serve` для генерации и просмотра отчётов.

## Отладка тестов с Allure

- В IntelliJ IDEA установите точки останова в тестовом коде или Page Object для анализа.
- Используйте SLF4J+Logback для логирования шагов и ошибок.
- Анализируйте отчёты Allure для выявления причин сбоев (скриншоты, логи, ответы API).

## Источники
- [Allure Framework Documentation](https://docs.qameta.io/allure/)
- [Allure GitHub Repository](https://github.com/allure-framework/allure2)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [TestNG Documentation](https://testng.org/doc/documentation-main.html)
- [Allure Maven Integration](https://docs.qameta.io/allure/#_maven)
- [Allure Plugin for IntelliJ IDEA](https://plugins.jetbrains.com/plugin/10407-allure)

---
[**← Назад к оглавлению**](README.md)
