# Именование тестов и методов

Правильное именование тестов и методов в автоматизированном тестировании повышает читаемость, упрощает поддержку кода и делает тесты понятными не только разработчикам, но и бизнес-пользователям. Названия должны отражать суть теста, избегать технического жаргона и следовать единому стилю, такому как `camelCase` без сокращений. Эта статья дополняет предыдущие материалы, фокусируясь на лучших практиках именования и их применении в контексте TestNG, Selenium и RestAssured.

## Принципы именования тестов

Названия тестов должны быть:
- **Описательными**: Чётко указывать, что проверяется (например, `testInvalidLogin` вместо `testLogin`).
- **Понятными бизнесу**: Избегать технических терминов, таких как "assert", "endpoint", вместо них использовать термины, понятные заказчикам (например, "проверка", "авторизация").
- **Единообразными**: Следовать стилю `camelCase` без сокращений (например, `testUserRegistration` вместо `testUsrReg`).
- **Краткими, но информативными**: Содержать достаточно деталей, но не быть избыточно длинными.

### Пример кода (TestNG с правильным именованием)

```java
import io.qameta.allure.Description;
import org.openqa.selenium.support.PageFactory;
import org.testng.annotations.Test;
import static org.testng.Assert.assertFalse;
import static org.testng.Assert.assertTrue;

public class UserAuthTest extends BaseTest {

    @Test(groups = {"ui", "smoke"})
    @Description("Проверка успешной авторизации пользователя")
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
    @Description("Проверка авторизации с неверными данными")
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
- **Названия тестов**:
  - `testSuccessfulUserLogin`: Указывает на проверку успешной авторизации.
  - `testInvalidUserLogin`: Указывает на проверку авторизации с неверными данными.
- **Описание в Allure**: Аннотация `@Description` дополняет название, делая его понятным для бизнеса.
- **Группы**: Используются для классификации тестов (`smoke`, `regression`).

## Именование методов в Page Object

Методы в классах Page Object должны быть понятными, описывать действия на странице и использовать глаголы, отражающие поведение.

### Пример кода (Page Object с правильным именованием)

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;

public class LoginPage {
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
        // Проверка успешности логина, например, по наличию элемента профиля
        return driver.findElements(By.id("profile")).size() > 0;
    }
}
```

### Пояснения
- **Названия методов**:
  - `enterCredentials`: Описывает действие ввода учетных данных.
  - `submitLogin`: Описывает отправку формы логина.
- **Избежание жаргона**: Вместо `setUsername` или `clickLoginBtn` используются понятные бизнесу термины.
- **camelCase**: Полные слова без сокращений (например, `enterCredentials` вместо `enterCreds`).

## Именование API-тестов

Для API-тестов названия должны отражать проверяемую функциональность и быть понятными бизнесу.

### Пример кода (TestNG с RestAssured)

```java
import io.qameta.allure.Description;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class UserApiTest extends BaseTest {

    @Test(groups = {"api", "smoke"})
    @Description("Проверка создания пользователя через API")
    public void testCreateUser() {
        // Arrange
        String requestBody = "{\"username\": \"newUser\", \"role\": \"admin\"}";

        // Act
        given()
                .contentType("application/json")
                .body(requestBody)
                .when()
                .post("/api/users")
                .then()
                .statusCode(201)
                .body("status", equalTo("created"));
    }

    @Test(groups = {"api", "regression"})
    @Description("Проверка ошибки при создании пользователя с некорректными данными")
    public void testCreateUserWithInvalidData() {
        // Arrange
        String requestBody = "{\"username\": \"\", \"role\": \"admin\"}";

        // Act
        given()
                .contentType("application/json")
                .body(requestBody)
                .when()
                .post("/api/users")
                .then()
                .statusCode(400)
                .body("error", equalTo("Invalid username"));
    }
}
```

### Пояснения
- **Названия тестов**:
  - `testCreateUser`: Указывает на проверку создания пользователя.
  - `testCreateUserWithInvalidData`: Уточняет сценарий с некорректными данными.
- **Бизнес-термины**: Используются слова "создание", "ошибка", вместо технических "postRequest" или "fail".

## Лучшие практики именования

- **Отражение сути**: Название должно отвечать на вопрос "что проверяет тест?" (например, `testInvalidUserLogin` вместо `testLoginFail`).
- **Бизнес-ориентированность**: Используйте термины, понятные заказчикам, например, "авторизация", "регистрация", вместо "check", "validate".
- **Единый стиль**: Применяйте `camelCase` для всех методов и тестов, избегайте сокращений (например, `testUserRegistration` вместо `testUsrReg`).
- **Контекст в группах**: Используйте группы TestNG (`ui`, `api`, `smoke`) для дополнительной классификации, чтобы названия оставались краткими.
- **Интеграция с Allure**: Добавляйте аннотации `@Description` для пояснений, видимых в отчётах.

## Интеграция с CI/CD

Для запуска тестов с правильным именованием настройте Maven и TestNG, чтобы тесты отображались в отчётах Allure с понятными названиями.

### Пример настройки GitHub Actions

```yaml
name: Run Tests with Named Tests
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
<suite name="NamedTestSuite">
    <test name="UITests">
        <groups>
            <run>
                <include name="ui"/>
            </run>
        </groups>
        <classes>
            <class name="UserAuthTest"/>
        </classes>
    </test>
    <test name="APITests">
        <groups>
            <run>
                <include name="api"/>
            </run>
        </groups>
        <classes>
            <class name="UserApiTest"/>
        </classes>
    </test>
</suite>
```

## Дополнительно

### Интеграция с Allure

Allure отображает названия тестов и методы в отчётах, делая их понятными для бизнеса.

```java
import io.qameta.allure.Description;
import io.qameta.allure.Step;
import org.testng.annotations.Test;

public class AllureNamedTest extends BaseTest {

    @Test(groups = {"ui", "regression"})
    @Description("Проверка добавления продукта в корзину")
    public void testAddProductToCart() {
        // Arrange
        CartPage cartPage = new CartPage(driver);
        driver.get("https://example.com/cart");

        // Act
        addProductToCart(cartPage, "product1");
        boolean isProductAdded = cartPage.isProductInCart("product1");

        // Assert
        assertTrue(isProductAdded, "Product was not added to cart");
    }

    @Step("Добавление продукта {productId} в корзину")
    private void addProductToCart(CartPage cartPage, String productId) {
        cartPage.addProduct(productId);
    }
}
```

### Проверка качества кода

Используйте Checkstyle для проверки соблюдения стиля именования. Пример конфигурации в `checkstyle.xml`:

```xml
<module name="MethodName">
    <property name="format" value="^[a-z][a-zA-Z0-9]*$"/>
    <message key="name.invalidPattern" value="Method name ''{0}'' must match pattern ''{1}''"/>
</module>
<module name="MemberName">
    <property name="format" value="^[a-z][a-zA-Z0-9]*$"/>
    <message key="name.invalidPattern" value="Member name ''{0}'' must match pattern ''{1}''"/>
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
1. Убедитесь, что названия тестов и методов понятны в окне TestNG Results.
2. Используйте точки останова в методах Page Object или тестах.
3. Проверяйте логи SLF4J для анализа выполнения.

## Распространённые ошибки

- **Неописательные названия**: Названия вроде `testLogin` не дают достаточно информации; используйте `testSuccessfulUserLogin`.
- **Технический жаргон**: Избегайте терминов, непонятных бизнесу, таких как `testEndpoint` или `checkBtn`.
- **Несоответствие стиля**: Убедитесь, что все методы следуют `camelCase` без сокращений.
- **Длинные названия**: Избегайте избыточной длины, например, `testUserLoginWithInvalidCredentialsAndErrorMessage` можно сократить до `testInvalidUserLogin`.

## Источники
- [TestNG Documentation](https://testng.org/doc/documentation-main.html)
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/webdriver/)
- [RestAssured Documentation](http://rest-assured.io/)
- [Allure Framework](https://allurereport.org/docs/)
- [Checkstyle Documentation](https://checkstyle.sourceforge.io/)

---
[**← Назад к оглавлению**](../README.md)