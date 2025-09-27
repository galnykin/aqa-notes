# Архитектура тестового фреймворка: структура, шаблоны, принципы

Построение надёжного и масштабируемого тестового фреймворка — ключ к успешной автоматизации. Ниже представлена структурированная модель проекта, основанная на лучших практиках: разделение кода, шаблоны проектирования, принципы организации, а также подготовка к финальному проекту.

-----

## 🏗️ Название проекта и `pom.xml`

Имя проекта в Maven, например `test-automated-framework`, задаётся в `pom.xml`. Этот файл — сердце проекта, управляющее зависимостями и сборкой.

### `pom.xml` (Ключевые зависимости)

Файл `pom.xml` должен включать все необходимые библиотеки для вашего стека.

```xml
<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>5.10.0</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-params</artifactId>
        <version>5.10.0</version>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>org.seleniumhq.selenium</groupId>
        <artifactId>selenium-java</artifactId>
        <version>4.12.1</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>io.github.bonigarcia</groupId>
        <artifactId>webdrivermanager</artifactId>
        <version>5.5.3</version>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>io.rest-assured</groupId>
        <artifactId>rest-assured</artifactId>
        <version>5.3.1</version>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>org.assertj</groupId>
        <artifactId>assertj-core</artifactId>
        <version>3.24.2</version>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>io.qameta.allure</groupId>
        <artifactId>allure-junit5</artifactId>
        <version>2.24.0</version>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>2.0.7</version>
    </dependency>
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.4.11</version>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.30</version>
        <scope>provided</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.1.2</version>
            <configuration>
                </configuration>
        </plugin>
    </plugins>
</build>
```

-----

## 📁 Структура проекта (Расширенная версия)

Профессиональная структура разделяет не только код и тесты, но и конфигурацию, API-клиенты и тестовые данные.

```
test-automated-framework
│
├── pom.xml
└── src
    ├── main
    │   └── java
    │       └── com.companyname
    │           ├── api         // API клиенты и спецификации (Rest Assured)
    │           ├── config      // Классы для чтения конфигурации (dev, stage)
    │           ├── core        // Управление WebDriver (Factory, ThreadLocal)
    │           ├── dto         // Data Transfer Objects для API
    │           ├── pages       // Page Object классы
    │           └── utils       // Вспомогательные утилиты (DataGenerator)
    └── test
        ├── java
        │   └── com.companyname
        │       ├── tests       // Тестовые классы, разделенные по функционалу
        │       └── listeners   // Пользовательские слушатели (Retry, Allure)
        └── resources
            ├── logback-test.xml // Конфигурация логирования
            └── config.properties // Файл с настройками (URL, credentials)
```

-----

## 📦 Структуризация классов (Современный подход)

### Пакет `pages` (с Lombok и BasePage)

Page Object классы наследуются от `BasePage`, который инкапсулирует общую логику (ожидания, работа с WebDriver). **Lombok** сокращает boilerplate-код.

```java
// BasePage.java - содержит общие методы для всех страниц
public abstract class BasePage {
    protected WebDriver driver;
    protected WebDriverWait wait;

    public BasePage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    }
    
    protected void click(By locator) {
        wait.until(ExpectedConditions.elementToBeClickable(locator)).click();
    }
    
    protected void sendKeys(By locator, String text) {
        WebElement element = wait.until(ExpectedConditions.visibilityOfElementLocated(locator));
        element.clear();
        element.sendKeys(text);
    }
}

// LoginPage.java - конкретная страница
@Getter // Lombok генерирует геттеры
public class LoginPage extends BasePage {
    private final By usernameInput = By.id("username");
    private final By passwordInput = By.id("password");
    private final By submitButton = By.id("submit");

    public LoginPage(WebDriver driver) {
        super(driver);
    }

    @Step("Авторизация с пользователем '{user}'")
    public HomePage loginAs(String user, String pass) {
        sendKeys(usernameInput, user);
        sendKeys(passwordInput, pass);
        click(submitButton);
        return new HomePage(driver); // Возвращаем следующий Page Object
    }
}
```

### Пакет `core` (WebDriver Factory с ThreadLocal)

`Singleton` — плохая практика для параллельного запуска тестов. **Factory** в связке с **ThreadLocal** обеспечивает изоляцию драйверов между потоками.

```java
public class WebDriverFactory {
    private static final ThreadLocal<WebDriver> driver = new ThreadLocal<>();

    public static WebDriver getDriver() {
        if (driver.get() == null) {
            WebDriverManager.chromedriver().setup();
            driver.set(new ChromeDriver());
            driver.get().manage().window().maximize();
        }
        return driver.get();
    }

    public static void quitDriver() {
        if (driver.get() != null) {
            driver.get().quit();
            driver.remove();
        }
    }
}
```

-----

## 🧪 Структуризация тестов

### `BaseTest`

`BaseTest` управляет жизненным циклом драйвера и может содержать экземпляры страниц для удобства.

#### JUnit 5

```java
public abstract class BaseTest {
    protected WebDriver driver;
    protected LoginPage loginPage;

    @BeforeEach
    void setUp() {
        driver = WebDriverFactory.getDriver();
        driver.get("https://your-app-url.com");
        loginPage = new LoginPage(driver);
        // Можно добавить другие общие действия, например, закрытие cookie
    }

    @AfterEach
    void tearDown() {
        WebDriverFactory.quitDriver();
    }
}
```

#### TestNG

```java
public abstract class BaseTest {
    protected WebDriver driver;
    // ...
    @BeforeMethod
    void setUp() { /* ... */ }

    @AfterMethod
    void tearDown() { /* ... */ }
}
```

### Тестовый класс с AssertJ

Тесты становятся чистыми, читаемыми и содержат только логику проверки. **AssertJ** предоставляет гибкие и описательные ассерты.

```java
@Epic("Authentication")
public class LoginTest extends BaseTest {

    @Test
    @Feature("Positive Login")
    @DisplayName("User should be able to login with valid credentials")
    void validLoginTest() {
        HomePage homePage = loginPage.loginAs("standard_user", "secret_sauce");
        
        // AssertJ для мощных и читаемых проверок
        assertThat(homePage.isShoppingCartVisible())
            .withFailMessage("Shopping cart should be visible after login")
            .isTrue();
    }
}
```

-----

## 🧱 Шаблоны проектирования

* **Page Object**: Изолирует логику страниц от тестов.

* **Factory + ThreadLocal**: Заменяет Singleton для безопасного параллельного выполнения тестов.

* **Builder**: Идеален для создания сложных тестовых данных (DTO). **Lombok** предоставляет аннотацию `@Builder`.

  ```java
  @Builder // Lombok
  @Getter
  public class User {
      private String username;
      private String password;
      private String email;
  }

  // Использование в тесте
  User testUser = User.builder()
                      .username("test_user")
                      .password("ValidPass123")
                      .email("test@example.com")
                      .build();
  ```

* **Strategy**: Позволяет динамически выбирать реализацию, например, для запуска тестов локально, в Selenoid или с помощью **Testcontainers**.

-----

## 🧬 Принципы организации

* **DRY (Don't Repeat Yourself)**: `BaseTest`, `BasePage` и утилиты помогают избежать дублирования кода.
* **Изоляция**: Тесты не знают о локаторах (`By`), они оперируют бизнес-методами страниц (`loginAs(...)`).
* **Чистота**: Тестовые методы содержат только три части: подготовка данных (Arrange), выполнение действия (Act) и проверка результата (Assert).

-----


Проверка, что каждый результат поиска содержит нужное слово.

```java
@Test
@DisplayName("Search results should contain the search query")
void searchResultsTest() {
    List<String> productTitles = searchPage.searchFor("Pizza")
                                           .getProductTitles();

    // AssertJ для проверки коллекций
    assertThat(productTitles)
        .isNotEmpty()
        .allMatch(title -> title.toLowerCase().contains("pizza"), "Each title should contain 'pizza'");
}
```

**На странице `SearchPage`:**

```java
public List<String> getProductTitles() {
    List<WebElement> productElements = driver.findElements(By.className("product-title"));
    return productElements.stream()
                          .map(WebElement::getText)
                          .collect(Collectors.toList());
}
```

-----

## ⚙️ Дополнительно: API + UI

Для ускорения и стабилизации тестов используйте API для предварительных шагов.
**Сценарий:** Добавление товара в корзину.

1.  **Arrange (API)**: Через **Rest Assured** отправьте POST-запрос, который добавляет товар в корзину для тестового пользователя. Это быстрее и надёжнее, чем кликать по UI.
2.  **Act (UI)**: Обновите страницу корзины в браузере.
3.  **Assert (UI)**: С помощью **Selenium** убедитесь, что товар отображается в корзине.

---

## 🔗 Источники:
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [WebDriverManager by Bonigarcia](https://github.com/bonigarcia/webdrivermanager)
- [Refactoring Guru — Шаблоны проектирования](https://refactoring.guru/ru/design-patterns)
- [XPath Syntax Reference](https://www.w3schools.com/xml/xpath_syntax.asp)

---
[**← Назад к оглавлению**](../README.md)