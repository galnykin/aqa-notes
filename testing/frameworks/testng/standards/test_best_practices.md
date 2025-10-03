# Лучшие практики написания тестов

Эффективное написание автотестов требует соблюдения лучших практик, которые обеспечивают читаемость, надёжность и удобство поддержки кода. В этой статье рассматриваются ключевые рекомендации: использование одного `@Test` для одной логической проверки, избегание неинформативных `assertTrue`, правильное применение `assertEquals`, `assertAll` и `SoftAssert`, а также минимизация `Thread.sleep` с использованием утилит ожидания (`WaitUtils`). Статья дополняет предыдущие материалы новым контентом, ориентированным на улучшение качества тестов.

## Один `@Test` — одна логическая проверка

Каждый тест, помеченный аннотацией `@Test`, должен проверять одну логическую функцию или сценарий. Это упрощает отладку, делает тесты более читаемыми и позволяет точно определить причину сбоя.

### Пример кода (TestNG с одной логической проверкой)

```java
import io.qameta.allure.Description;
import io.qameta.allure.Epic;
import io.qameta.allure.Feature;
import org.openqa.selenium.support.PageFactory;
import org.testng.annotations.Test;
import static org.testng.Assert.assertTrue;

@Epic("Авторизация пользователей")
@Feature("Веб-интерфейс авторизации")
public class UserLoginTest extends BaseTest {

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
    @Description("Проверка отображения ошибки при неверных учетных данных")
    public void testInvalidUserLoginShowsError() {
        // Arrange
        LoginPage loginPage = PageFactory.initElements(driver, LoginPage.class);
        driver.get("https://example.com/login");

        // Act
        loginPage.enterCredentials("invalidUser", "invalidPass");
        loginPage.submitLogin();

        // Assert
        assertTrue(loginPage.isErrorMessageDisplayed(), "Error message was not displayed for invalid login");
    }
}
```

### Пояснения
- **Один `@Test` — одна проверка**:
  - `testSuccessfulUserLogin`: Проверяет успешную авторизацию.
  - `testInvalidUserLoginShowsError`: Проверяет отображение ошибки при неверных данных.
- **Преимущества**: Легко понять, что именно проверяет тест, и проще отлаживать сбои.
- **Allure**: Аннотация `@Description` уточняет назначение теста.

### Лучшие практики
- **Разделение сценариев**: Не комбинируйте в одном тесте проверку успешной и неуспешной авторизации.
- **Чёткие названия**: Используйте описательные имена, отражающие цель теста (например, `testSuccessfulUserLogin` вместо `testLogin`).
- **Группы**: Применяйте группы (`smoke`, `regression`) для классификации тестов.

## Избегать `assertTrue(condition)` без пояснения

Использование `assertTrue` без информативного сообщения об ошибке затрудняет анализ сбоев. Всегда указывайте сообщение, описывающее ожидаемый результат.

### Плохой пример

```java
@Test
public void testUserLogin() {
    LoginPage loginPage = new LoginPage(driver);
    driver.get("https://example.com/login");
    loginPage.enterCredentials("validUser", "validPass");
    boolean isLoggedIn = loginPage.submitLogin();
    assertTrue(isLoggedIn); // Нет пояснения
}
```

### Хороший пример

```java
@Test(groups = {"ui", "smoke"})
@Description("Проверка успешной авторизации пользователя")
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
```

### Пояснения
- **Сообщение об ошибке**: `"User login was not successful"` указывает, что именно пошло не так.
- **Логирование**: Добавьте логирование с SLF4J для дополнительной информации.

### Лучшие практики
- **Информативные сообщения**: Указывайте, что ожидалось и что не выполнено.
- **Контекст**: Включайте в сообщение детали, такие как входные данные или окружение.
- **Allure**: Используйте `@Description` для пояснений в отчётах.

## Использование `assertEquals`, `assertAll`, `SoftAssert` по назначению

TestNG и AssertJ предоставляют различные механизмы для утверждений, которые следует использовать в зависимости от сценария.

### `assertEquals`
- **Назначение**: Сравнивает ожидаемое и фактическое значение.
- **Когда использовать**: Для проверки точного соответствия (например, значения полей, статус-кодов).

### Пример кода (`assertEquals`)

```java
import io.qameta.allure.Description;
import org.testng.annotations.Test;
import static org.testng.Assert.assertEquals;

public class UserApiTest extends BaseTest {

    @Test(groups = {"api", "smoke"})
    @Description("Проверка создания пользователя через API")
    public void testCreateUserReturnsStatus() {
        // Arrange
        String requestBody = "{\"username\": \"newUser\", \"role\": \"admin\"}";

        // Act
        int statusCode = given()
                .contentType("application/json")
                .body(requestBody)
                .when()
                .post("/api/users")
                .then()
                .extract().statusCode();

        // Assert
        assertEquals(statusCode, 201, "Expected status code 201 for user creation");
    }
}
```

### `assertAll` (AssertJ)
- **Назначение**: Выполняет все проверки, даже если некоторые проваливаются, собирая все ошибки.
- **Когда использовать**: Для проверки нескольких условий в одном тесте.

### Пример кода (`assertAll`)

```java
import io.qameta.allure.Description;
import org.testng.annotations.Test;
import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.SoftAssertions.assertSoftly;

public class UserProfileTest extends BaseTest {

    @Test(groups = {"ui", "regression"})
    @Description("Проверка отображения профиля пользователя")
    public void testUserProfileDetails() {
        // Arrange
        UserProfilePage profilePage = new UserProfilePage(driver);
        driver.get("https://example.com/profile");

        // Act
        String username = profilePage.getUsername();
        String email = profilePage.getEmail();

        // Assert
        assertSoftly(softly -> {
            softly.assertThat(username).as("Username should match").isEqualTo("validUser");
            softly.assertThat(email).as("Email should match").isEqualTo("user@example.com");
        });
    }
}
```

### `SoftAssert` (TestNG)
- **Назначение**: Собирает все ошибки в тесте, не прерывая выполнение.
- **Когда использовать**: Когда нужно проверить несколько условий, но продолжить выполнение теста.

### Пример кода (`SoftAssert`)

```java
import io.qameta.allure.Description;
import org.testng.annotations.Test;
import org.testng.asserts.SoftAssert;

public class ProductCartTest extends BaseTest {

    @Test(groups = {"ui", "regression"})
    @Description("Проверка добавления продукта в корзину")
    public void testAddProductToCart() {
        // Arrange
        CartPage cartPage = new CartPage(driver);
        driver.get("https://example.com/cart");
        SoftAssert softAssert = new SoftAssert();

        // Act
        cartPage.addProduct("product1");
        String productName = cartPage.getProductName();
        int productCount = cartPage.getProductCount();

        // Assert
        softAssert.assertEquals(productName, "product1", "Product name in cart is incorrect");
        softAssert.assertEquals(productCount, 1, "Product count in cart is incorrect");
        softAssert.assertAll(); // Проверяет все утверждения
    }
}
```

### Пояснения
- **`assertEquals`**: Используется для точного сравнения (например, статус-кода API).
- **`assertAll` (AssertJ)**: Собирает все ошибки, отображая их в отчёте.
- **`SoftAssert` (TestNG)**: Аналогична `assertAll`, но встроена в TestNG.
- **Зависимости**:

```xml
<dependency>
    <groupId>org.assertj</groupId>
    <artifactId>assertj-core</artifactId>
    <version>3.26.3</version>
    <scope>test</scope>
</dependency>
```

### Лучшие практики
- **Используйте `assertEquals` для точных проверок**: Например, сравнение строк, чисел или статус-кодов.
- **Применяйте `assertAll` или `SoftAssert` для множественных проверок**: Это позволяет увидеть все ошибки, а не только первую.
- **Добавляйте сообщения**: Указывайте описания для всех утверждений, чтобы упростить отладку.

## Минимизация `Thread.sleep`, использование `WaitUtils`

Использование `Thread.sleep` делает тесты нестабильными и замедляет их выполнение. Вместо этого применяйте утилиты ожидания, такие как `WebDriverWait` для UI-тестов или кастомные `WaitUtils` для API.

### Пример кода (Плохой подход с `Thread.sleep`)

```java
@Test
public void testLoginWithSleep() {
    LoginPage loginPage = new LoginPage(driver);
    driver.get("https://example.com/login");
    loginPage.enterCredentials("validUser", "validPass");
    loginPage.submitLogin();
    Thread.sleep(5000); // Жёсткое ожидание
    assertTrue(loginPage.isLoggedIn(), "Login failed");
}
```

### Пример кода (Хороший подход с `WaitUtils`)

```java
import io.qameta.allure.Description;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.testng.annotations.Test;
import java.time.Duration;
import static org.testng.Assert.assertTrue;

public class UserLoginWaitTest extends BaseTest {

    @Test(groups = {"ui", "smoke"})
    @Description("Проверка авторизации с ожиданием")
    public void testUserLoginWithWait() {
        // Arrange
        LoginPage loginPage = new LoginPage(driver);
        driver.get("https://example.com/login");

        // Act
        loginPage.enterCredentials("validUser", "validPass");
        loginPage.submitLogin();
        WaitUtils.waitForLoginSuccess(driver, Duration.ofSeconds(10));

        // Assert
        assertTrue(loginPage.isLoggedIn(), "Login failed");
    }
}
```

### Пример утилиты `WaitUtils`

```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;

public class WaitUtils {
    public static void waitForLoginSuccess(WebDriver driver, Duration timeout) {
        new WebDriverWait(driver, timeout)
                .until(ExpectedConditions.visibilityOfElementLocated(By.id("profile")));
    }

    public static void waitForElementVisible(WebDriver driver, By locator, Duration timeout) {
        new WebDriverWait(driver, timeout)
                .until(ExpectedConditions.visibilityOfElementLocated(locator));
    }
}
```

### Пояснения
- **`WebDriverWait`**: Ожидает определённое условие (например, видимость элемента) вместо фиксированного времени.
- **`WaitUtils`**: Инкапсулирует логику ожиданий, повышая читаемость кода.
- **Преимущества**: Уменьшает нестабильность тестов и ускоряет их выполнение.

### Лучшие практики
- **Используйте явные ожидания**: Применяйте `WebDriverWait` для UI-тестов и кастомные ожидания для API.
- **Инкапсуляция**: Храните логику ожиданий в утилитах, таких как `WaitUtils`.
- **Таймауты**: Устанавливайте разумные таймауты (например, 10 секунд) с учётом окружения.

## Интеграция с CI/CD

Для запуска тестов с соблюдением лучших практик настройте Maven и TestNG в CI/CD.

### Пример настройки GitHub Actions

```yaml
name: Run Tests with Best Practices
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
<suite name="BestPracticesSuite" parallel="methods" thread-count="3">
    <test name="UITests">
        <groups>
            <run>
                <include name="ui"/>
            </run>
        </groups>
        <classes>
            <class name="com.example.tests.ui.UserLoginTest"/>
            <class name="com.example.tests.ui.UserLoginWaitTest"/>
        </classes>
    </test>
    <test name="APITests">
        <groups>
            <run>
                <include name="api"/>
            </run>
        </groups>
        <classes>
            <class name="com.example.tests.api.UserApiTest"/>
        </classes>
    </test>
</suite>
```

## Дополнительно

### Интеграция с Allure

Allure улучшает отчётность, отображая шаги, утверждения и ошибки.

```java
import io.qameta.allure.Description;
import io.qameta.allure.Epic;
import io.qameta.allure.Feature;
import io.qameta.allure.Step;
import org.testng.annotations.Test;
import org.testng.asserts.SoftAssert;

@Epic("Покупки")
@Feature("Корзина")
public class CartTest extends BaseTest {

    @Test(groups = {"ui", "regression"})
    @Description("Проверка добавления нескольких продуктов в корзину")
    public void testAddMultipleProductsToCart() {
        // Arrange
        CartPage cartPage = new CartPage(driver);
        driver.get("https://example.com/cart");
        SoftAssert softAssert = new SoftAssert();

        // Act
        addProductToCart(cartPage, "product1");
        addProductToCart(cartPage, "product2");

        // Assert
        softAssert.assertEquals(cartPage.getProductName(1), "product1", "First product name is incorrect");
        softAssert.assertEquals(cartPage.getProductName(2), "product2", "Second product name is incorrect");
        softAssert.assertEquals(cartPage.getProductCount(), 2, "Product count is incorrect");
        softAssert.assertAll();
    }

    @Step("Добавление продукта {productId} в корзину")
    private void addProductToCart(CartPage cartPage, String productId) {
        cartPage.addProduct(productId);
    }
}
```

### Проверка качества кода

Используйте PMD для проверки соблюдения лучших практик. Пример конфигурации в `pom.xml`:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-pmd-plugin</artifactId>
    <version>3.23.0</version>
    <configuration>
        <rulesets>
            <ruleset>rulesets/java/bestpractices.xml</ruleset>
        </rulesets>
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

Запуск:
```bash
mvn pmd:check
```

### Отладка

Для отладки в IntelliJ IDEA:
1. Установите точки останова в методах `@Test` или утилитах `WaitUtils`.
2. Используйте TestNG Run/Debug Configuration для запуска тестов.
3. Проверяйте логи SLF4J и отчёты Allure для анализа ошибок.

## Распространённые ошибки

- **Множественные проверки в одном тесте**: Разделяйте тесты, чтобы каждый `@Test` проверял одну функцию.
- **Неинформативные `assertTrue`**: Всегда добавляйте сообщения об ошибках.
- **Злоупотребление `Thread.sleep`**: Используйте `WebDriverWait` или кастомные ожидания.
- **Игнорирование `assertAll`/`SoftAssert`**: Применяйте их для множественных проверок, чтобы увидеть все ошибки.

## Источники
- [TestNG Documentation](https://testng.org/doc/documentation-main.html)
- [AssertJ Documentation](https://assertj.github.io/doc/)
- [Allure Framework](https://allurereport.org/docs/)
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/webdriver/)

---
[**← Назад к оглавлению**](../README.md)