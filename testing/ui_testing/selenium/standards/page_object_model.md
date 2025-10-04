# Структура Page Object модели

Page Object Model (POM) — это паттерн проектирования, который помогает организовать код автотестов, делая его более читаемым, поддерживаемым и масштабируемым. Он разделяет логику работы с веб-страницами от логики тестов, представляя каждую страницу или компонент как отдельный класс. Такой подход упрощает написание тестов, минимизирует дублирование кода и улучшает его читаемость.

## Основные принципы Page Object модели

- **Один класс — одна страница или компонент**: Каждый класс Page Object представляет конкретную страницу (например, главная страница, страница логина) или компонент (например, хедер, модальное окно).
- **Разделение логики**: Логика взаимодействия с элементами страницы (поиск, клики, проверки) находится в классах Page Object, а тесты содержат только высокоуровневые шаги.
- **Методы отражают действия пользователя**: Методы в Page Object должны описывать действия (например, `login`, `clickSubmitButton`) или проверки (`isErrorMessageDisplayed`), а не технические детали (например, поиск локаторов).
- **Повторное использование кода**: Общие функции, такие как ожидания или работа с драйвером, выносятся в базовые классы (`BasePage`, `BaseTest`).

## Реализация Page Object модели

### Базовый класс BasePage

Базовый класс `BasePage` содержит общие методы для работы с элементами страницы, такие как ожидания, клики и проверки. Он также инициализирует `WebDriver` и использует `PageFactory` для работы с аннотациями `@FindBy`.

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.PageFactory;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;

public class BasePage {
    protected WebDriver driver;
    protected WebDriverWait wait;

    public BasePage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        PageFactory.initElements(driver, this);
    }

    // Ожидание видимости элемента
    protected WebElement waitForElementVisible(WebElement element) {
        return wait.until(ExpectedConditions.visibilityOf(element));
    }

    // Клик по элементу с ожиданием
    protected void click(WebElement element) {
        waitForElementVisible(element).click();
    }

    // Проверка, отображается ли элемент
    protected boolean isElementDisplayed(WebElement element) {
        return waitForElementVisible(element).isDisplayed();
    }
}
```

### Пример класса Page Object

Класс `LoginPage` представляет страницу логина. Он содержит локаторы, методы для взаимодействия с элементами и проверки состояния страницы.

```java
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;

public class LoginPage extends BasePage {

    @FindBy(id = "username")
    private WebElement usernameField;

    @FindBy(id = "password")
    private WebElement passwordField;

    @FindBy(id = "loginButton")
    private WebElement loginButton;

    @FindBy(id = "errorMessage")
    private WebElement errorMessage;

    public LoginPage(WebDriver driver) {
        super(driver);
    }

    // Ввод логина
    public LoginPage enterUsername(String username) {
        usernameField.sendKeys(username);
        return this; // Поддержка цепочек вызовов
    }

    // Ввод пароля
    public LoginPage enterPassword(String password) {
        passwordField.sendKeys(password);
        return this;
    }

    // Нажатие кнопки логина
    public void clickLoginButton() {
        click(loginButton);
    }

    // Проверка отображения сообщения об ошибке
    public boolean isErrorMessageDisplayed() {
        return isElementDisplayed(errorMessage);
    }
}
```

### Пример теста с использованием Page Object

Тест использует класс `LoginPage` для выполнения сценария логина. Структура теста следует шаблону AAA (Arrange-Act-Assert).

#### JUnit 5
```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import static org.assertj.core.api.Assertions.assertThat;

public class LoginTest {
    private WebDriver driver;
    private LoginPage loginPage;

    @BeforeEach
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        loginPage = new LoginPage(driver);
        driver.get("https://example.com/login");
    }

    @Test
    void testInvalidLogin() {
        // Arrange
        String username = "invalidUser";
        String password = "invalidPass";

        // Act
        loginPage.enterUsername(username)
                .enterPassword(password)
                .clickLoginButton();

        // Assert
        assertThat(loginPage.isErrorMessageDisplayed()).isTrue();
    }

    @AfterEach
    void tearDown() {
        driver.quit();
    }
}
```

#### TestNG
```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import static org.assertj.core.api.Assertions.assertThat;

public class LoginTest {
    private WebDriver driver;
    private LoginPage loginPage;

    @BeforeMethod
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        loginPage = new LoginPage(driver);
        driver.get("https://example.com/login");
    }

    @Test
    void testInvalidLogin() {
        // Given
        String username = "invalidUser";
        String password = "invalidPass";

        // When
        loginPage.enterUsername(username)
                .enterPassword(password)
                .clickLoginButton();

        // Then
        assertThat(loginPage.isErrorMessageDisplayed()).isTrue();
    }

    @AfterMethod
    void tearDown() {
        driver.quit();
    }
}
```

## Maven зависимости

Для работы с Selenium, JUnit 5, TestNG, AssertJ и WebDriverManager добавьте следующие зависимости в `pom.xml`:

```xml
<dependencies>
    <!-- Selenium -->
    <dependency>
        <groupId>org.seleniumhq.selenium</groupId>
        <artifactId>selenium-java</artifactId>
        <version>4.25.0</version>
    </dependency>
    <!-- WebDriverManager -->
    <dependency>
        <groupId>io.github.bonigarcia</groupId>
        <artifactId>webdrivermanager</artifactId>
        <version>5.9.2</version>
    </dependency>
    <!-- JUnit 5 -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>5.11.3</version>
        <scope>test</scope>
    </dependency>
    <!-- AssertJ -->
    <dependency>
        <groupId>org.assertj</groupId>
        <artifactId>assertj-core</artifactId>
        <version>3.26.3</version>
        <scope>test</scope>
    </dependency>
    <!-- TestNG (если используется) -->
    <dependency>
        <groupId>org.testng</groupId>
        <artifactId>testng</artifactId>
        <version>7.10.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

## Устойчивость тестов

Для повышения стабильности тестов в Page Object модели:

- **Стабильные локаторы**: Используйте `id` или атрибуты `data-*` для поиска элементов. Избегайте сложных CSS или XPath, так как они могут быть нестабильными при изменении структуры страницы.
- **Ожидания**: Применяйте `WebDriverWait` для ожидания видимости, кликабельности или других условий. Например, метод `waitForElementVisible` в `BasePage` гарантирует, что элемент доступен перед взаимодействием.
- **Проверки состояния**: Используйте методы `isDisplayed()` и `isEnabled()` для проверки состояния элементов перед действиями.
- **Fluent API**: Поддерживайте цепочки вызовов (методы возвращают `this`), чтобы упростить написание тестов.

## Интеграция с CI/CD

Для запуска тестов в Jenkins или GitHub Actions настройте выполнение Maven-команд. Пример конфигурации для GitHub Actions:

```yaml
name: CI Pipeline
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
      - name: Run tests
        run: mvn clean test
```

## Отладка

Для отладки тестов в IntelliJ IDEA:
- Установите точки останова в методах Page Object или тестов.
- Используйте режим Debug для проверки значений локаторов и состояния элементов.
- Логируйте действия (например, с помощью SLF4J и Logback) для отслеживания шагов теста.

## Распространённые ошибки

- **Дублирование кода**: Не выносите общую логику в `BasePage`, что приводит к повторению кода в классах страниц.
- **Сложные локаторы**: Использование хрупких локаторов (например, длинных XPath) делает тесты нестабильными.
- **Отсутствие ожиданий**: Игнорирование `WebDriverWait` приводит к flaky тестам из-за асинхронной загрузки страниц.
- **Смешивание логики**: Размещение тестовых шагов в классах Page Object вместо тестов усложняет поддержку.

## Дополнительно

- **Allure для отчётов**: Интегрируйте Allure для генерации отчётов. Добавьте зависимость в `pom.xml` и аннотации `@Step` в методы Page Object для детализированных отчётов.
- **Selenide**: Для упрощения работы с Selenium рассмотрите использование Selenide, который предоставляет встроенные ожидания и более лаконичный синтаксис.
- **Testcontainers**: Для тестирования в изолированных окружениях используйте Testcontainers для запуска браузеров в Docker-контейнерах.

## Источники
- [Selenium Documentation](https://www.selenium.dev/documentation/)
- [WebDriverManager](https://bonigarcia.dev/webdrivermanager/)
- [Allure Framework](https://allurereport.org/)

---
[**← Назад к оглавлению**](../README.md)