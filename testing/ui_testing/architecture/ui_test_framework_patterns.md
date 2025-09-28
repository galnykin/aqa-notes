
# Расширенные практики архитектуры UI-тестов на Java

Современные UI-фреймворки на Java строятся не только на Page Object, но и на дополнительных архитектурных подходах, которые делают тесты более читаемыми, масштабируемыми и гибкими. Ниже представлены ключевые практики: работа с клавишами, мягкие проверки, разделение локаторов, шаги и действия, а также использование вспомогательных классов.

---

## ⌨️ Работа с клавишами: `Keys.RETURN`

Иногда вместо клика по кнопке требуется имитировать нажатие клавиши — например, `Enter` для отправки формы:

```java
WebElement input = driver.findElement(By.id("search"));
input.sendKeys("Pizza");
input.sendKeys(Keys.RETURN); // имитация Enter
```

### Пример теста с TestNG

```java
public class SearchTest {

    private WebDriver driver;

    @BeforeMethod
    public void setUp() {
        driver = new ChromeDriver();
        driver.get("https://example.com");
    }

    @Test
    public void searchWithEnterKey() {
        WebElement input = driver.findElement(By.id("search"));
        input.sendKeys("Pizza");
        input.sendKeys(Keys.RETURN);

        Assert.assertTrue(driver.getTitle().contains("Pizza"));
    }

    @AfterMethod
    public void tearDown() {
        driver.quit();
    }
}
```

---

## ✅ Мягкие проверки: AssertJ и Soft Assertions

Вместо стандартных `Assertions.assertTrue()` можно использовать **AssertJ** — библиотеку для выразительных и цепочных проверок:

```java
import static org.assertj.core.api.Assertions.assertThat;

assertThat(element.getText()).contains("Welcome");
```

Для **мягких проверок** (soft assertions), которые не прерывают тест при первой ошибке:

```java
SoftAssertions softly = new SoftAssertions();
softly.assertThat(title).contains("Pizza");
softly.assertThat(price).isGreaterThan(0);
softly.assertAll(); // финальная проверка
```

### Пример теста с TestNG и SoftAssertions

```java
public class ProductPageTest {

    private WebDriver driver;

    @BeforeMethod
    public void setUp() {
        driver = new ChromeDriver();
        driver.get("https://example.com/product");
    }

    @Test
    public void checkProductDetails() {
        String title = driver.findElement(By.id("title")).getText();
        int price = Integer.parseInt(driver.findElement(By.id("price")).getText());

        SoftAssertions softly = new SoftAssertions();
        softly.assertThat(title).contains("Pizza");
        softly.assertThat(price).isGreaterThan(0);
        softly.assertAll();
    }

    @AfterMethod
    public void tearDown() {
        driver.quit();
    }
}
```

---

## 🧱 Page Object как архитектура

Page Object описывает элементы страницы и действия над ними:

```java
public class LoginPage {
    private final WebDriver driver;

    private final By inputLogin = By.id("username");
    private final By inputPassword = By.id("password");
    private final By buttonSubmit = By.id("submit");

    public LoginPage(WebDriver driver) {
        this.driver = driver;
    }

    public HomePage login(String user, String pass) {
        driver.findElement(inputLogin).sendKeys(user);
        driver.findElement(inputPassword).sendKeys(pass);
        driver.findElement(buttonSubmit).click();
        return new HomePage(driver);
    }
}
```

### Пример теста с TestNG

```java
public class LoginTest {

    private WebDriver driver;
    private LoginPage loginPage;

    @BeforeMethod
    public void setUp() {
        driver = new ChromeDriver();
        driver.get("https://example.com/login");
        loginPage = new LoginPage(driver);
    }

    @Test(groups = {"smoke"})
    public void validLoginTest() {
        HomePage homePage = loginPage.login("admin", "1234");
        assertThat(homePage.getWelcomeMessage()).contains("Welcome");
    }

    @AfterMethod
    public void tearDown() {
        driver.quit();
    }
}
```

---

## 📦 Разделение локаторов: `HomePageLocators`

Иногда локаторы выносят в отдельный класс:

```java
public class HomePageLocators {
    public static final By ACCEPT_COOKIES = By.id("acceptCookies");
    public static final By SEARCH_FIELD = By.name("search");
}
```

### Пример теста с TestNG и локаторами

```java
public class HomeTest {

    private WebDriver driver;

    @BeforeMethod
    public void setUp() {
        driver = new ChromeDriver();
        driver.get("https://example.com");
    }

    @Test
    public void searchFromHomePage() {
        driver.findElement(HomePageLocators.SEARCH_FIELD).sendKeys("Pizza");
        driver.findElement(HomePageLocators.SEARCH_FIELD).sendKeys(Keys.RETURN);

        Assert.assertTrue(driver.getTitle().contains("Pizza"));
    }

    @AfterMethod
    public void tearDown() {
        driver.quit();
    }
}
```

---

## 🧩 Шаги и действия: `Steps`-классы

Для организации сценариев выносят действия в отдельный класс:

```java
public class HomeSteps {
    private final WebDriver driver;

    public HomeSteps(WebDriver driver) {
        this.driver = driver;
    }

    public void searchFor(String query) {
        driver.findElement(HomePageLocators.SEARCH_FIELD).sendKeys(query);
        driver.findElement(HomePageLocators.SEARCH_FIELD).sendKeys(Keys.RETURN);
    }
}
```

### Пример теста с TestNG и шагами

```java
public class SearchStepsTest {

    private WebDriver driver;
    private HomeSteps homeSteps;

    @BeforeMethod
    public void setUp() {
        driver = new ChromeDriver();
        driver.get("https://example.com");
        homeSteps = new HomeSteps(driver);
    }

    @Test(dataProvider = "queries")
    public void searchWithSteps(String query) {
        homeSteps.searchFor(query);
        Assert.assertTrue(driver.getTitle().contains(query));
    }

    @DataProvider(name = "queries")
    public Object[][] queries() {
        return new Object[][] {
            {"Pizza"}, {"Burger"}
        };
    }

    @AfterMethod
    public void tearDown() {
        driver.quit();
    }
}
```

---

## 🔗 Источники:

* [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/)
* [AssertJ Documentation](https://assertj.github.io/doc/)
* [TestNG Documentation](https://testng.org/doc/)
* [Design Patterns for Test Automation](https://refactoring.guru/ru/design-patterns)

---

[**← Назад к оглавлению**](../../../README.md)

---
