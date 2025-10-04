# Стандарты валидации и ассертов

## Использование ассертов

Ассерты в автоматизированных тестах подтверждают корректность поведения системы. Используйте библиотеки JUnit 5, TestNG и AssertJ для написания читаемых и гибких проверок. Основные методы включают `assertEquals`, `assertTrue`, `assertThat` и `softAssert` для различных сценариев валидации.

### assertEquals, assertTrue, assertThat

- **`assertEquals`**: Сравнивает ожидаемое и фактическое значение. Используется для точного соответствия (например, текст или числовые значения).
- **`assertTrue`**: Проверяет истинность условия. Подходит для булевых проверок.
- **`assertThat` (AssertJ)**: Предоставляет гибкий синтаксис для сложных проверок с читаемыми сообщениями об ошибках.

Пример с JUnit 5:

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.assertj.core.api.Assertions.assertThat;

public class ValidationTest {
    @Test
    void testBasicAssertions() {
        // Arrange
        String actualText = "Welcome";
        int actualCount = 5;
        boolean isActive = true;

        // Act & Assert
        assertEquals("Welcome", actualText, "Text should match");
        assertTrue(isActive, "Element should be active");
        assertThat(actualCount).as("Count should be greater than 3").isGreaterThan(3);
    }
}
```

Пример с TestNG:

```java
import org.testng.annotations.Test;
import static org.testng.Assert.assertEquals;
import static org.testng.Assert.assertTrue;
import static org.assertj.core.api.Assertions.assertThat;

public class ValidationTestNG {
    @Test
    void testBasicAssertions() {
        // Given
        String actualText = "Welcome";
        int actualCount = 5;
        boolean isActive = true;

        // When & Then
        assertEquals(actualText, "Welcome", "Text should match");
        assertTrue(isActive, "Element should be active");
        assertThat(actualCount).as("Count should be greater than 3").isGreaterThan(3);
    }
}
```

### Soft Assertions

`SoftAssert` (TestNG) или `SoftAssertions` (AssertJ) позволяют выполнять все проверки в тесте, даже если некоторые из них не проходят, собирая ошибки для отчёта.

Пример с AssertJ SoftAssertions:

```java
import org.assertj.core.api.SoftAssertions;
import org.junit.jupiter.api.Test;

public class SoftValidationTest {
    @Test
    void testSoftAssertions() {
        // Arrange
        SoftAssertions softly = new SoftAssertions();
        String actualText = "Welcome";
        int actualCount = 5;

        // Act & Assert
        softly.assertThat(actualText).as("Text check").isEqualTo("Welcome");
        softly.assertThat(actualCount).as("Count check").isGreaterThan(10); // Ошибка
        softly.assertThat(actualCount).as("Count equality").isEqualTo(5);

        // Проверка всех ассертов
        softly.assertAll();
    }
}
```

Пример с TestNG SoftAssert:

```java
import org.testng.asserts.SoftAssert;
import org.testng.annotations.Test;

public class SoftValidationTestNG {
    @Test
    void testSoftAssertions() {
        // Given
        SoftAssert softly = new SoftAssert();
        String actualText = "Welcome";
        int actualCount = 5;

        // When & Then
        softly.assertEquals(actualText, "Welcome", "Text check");
        softly.assertTrue(actualCount > 10, "Count should be greater than 10"); // Ошибка
        softly.assertEquals(actualCount, 5, "Count equality");

        // Проверка всех ассертов
        softly.assertAll();
    }
}
```

## Проверка состояния элементов

В UI-тестах с Selenide проверяйте состояние элементов, такие как текст, классы и атрибуты, для подтверждения корректного отображения интерфейса. Используйте стабильные локаторы и явные ожидания для устойчивости тестов.

Пример проверки состояния с Selenide:

```java
import com.codeborne.selenide.SelenideElement;
import static com.codeborne.selenide.Condition.*;
import static com.codeborne.selenide.Selenide.$;

public class LoginPage {
    private final SelenideElement loginButton = $("#login-button");
    private final SelenideElement errorMessage = $(".error-message");

    public void validateLoginButtonState() {
        loginButton.shouldBe(visible).shouldHave(attribute("type", "submit"));
        loginButton.shouldHave(cssClass("btn-primary"));
    }

    public void validateErrorMessage(String expectedText) {
        errorMessage.shouldBe(visible).shouldHave(text(expectedText));
    }
}
```

Пример теста с JUnit 5:

```java
import org.junit.jupiter.api.Test;
import static com.codeborne.selenide.Selenide.open;

public class LoginPageTest {
    @Test
    void testLoginButtonAndErrorMessage() {
        // Arrange
        LoginPage loginPage = new LoginPage();
        open("/login");

        // Act
        loginPage.validateLoginButtonState();
        loginPage.validateErrorMessage("Invalid credentials");

        // Assert
        // Дополнительные проверки, если необходимо
    }
}
```

Пример теста с TestNG:

```java
import org.testng.annotations.Test;
import static com.codeborne.selenide.Selenide.open;

public class LoginPageTestNG {
    @Test
    void testLoginButtonAndErrorMessage() {
        // Given
        LoginPage loginPage = new LoginPage();
        open("/login");

        // When
        loginPage.validateLoginButtonState();
        loginPage.validateErrorMessage("Invalid credentials");

        // Then
        // Дополнительные проверки, если необходимо
    }
}
```

## Валидация бизнес-логики и UI-состояний

Валидация бизнес-логики требует проверки соответствия функциональности требованиям. Например, после отправки формы должны отображаться определённые элементы или изменяться состояние UI. Для этого используйте проверки на основе клиент-серверной модели, где UI отражает результат серверных операций.

Пример валидации бизнес-логики:

```java
import com.codeborne.selenide.SelenideElement;
import static com.codeborne.selenide.Condition.visible;
import static com.codeborne.selenide.Selenide.$;

public class OrderPage {
    private final SelenideElement orderConfirmation = $("#order-confirmation");
    private final SelenideElement orderId = $("#order-id");

    public void submitOrder(String product) {
        $("#product-select").selectOption(product);
        $("#submit-order").click();
        orderConfirmation.shouldBe(visible, Duration.ofSeconds(5));
    }

    public String getOrderId() {
        return orderId.getText();
    }
}
```

Пример теста с JUnit 5:

```java
import org.junit.jupiter.api.Test;
import static org.assertj.core.api.Assertions.assertThat;
import static com.codeborne.selenide.Selenide.open;

public class OrderPageTest {
    @Test
    void testOrderSubmission() {
        // Arrange
        OrderPage orderPage = new OrderPage();
        open("/order");

        // Act
        orderPage.submitOrder("Laptop");

        // Assert
        assertThat(orderPage.getOrderId()).startsWith("ORD-");
    }
}
```

Пример теста с TestNG:

```java
import org.testng.annotations.Test;
import static org.assertj.core.api.Assertions.assertThat;
import static com.codeborne.selenide.Selenide.open;

public class OrderPageTestNG {
    @Test
    void testOrderSubmission() {
        // Given
        OrderPage orderPage = new OrderPage();
        open("/order");

        // When
        orderPage.submitOrder("Laptop");

        // Then
        assertThat(orderPage.getOrderId()).startsWith("ORD-");
    }
}
```

## Группировка ассертов и отчётность

Группировка ассертов улучшает читаемость тестов и упрощает анализ ошибок. Используйте `SoftAssertions` для выполнения всех проверок перед завершением теста. Для отчётности интегрируйте Allure, чтобы фиксировать результаты и прикреплять скриншоты при падении.

Пример с группировкой ассертов и Allure:

```java
import io.qameta.allure.Step;
import org.assertj.core.api.SoftAssertions;
import org.junit.jupiter.api.Test;
import static com.codeborne.selenide.Selenide.*;

public class CheckoutPageTest {
    @Test
    @Step("Валидация страницы оформления заказа")
    void testCheckoutPage() {
        // Arrange
        SoftAssertions softly = new SoftAssertions();
        open("/checkout");

        // Act
        SelenideElement totalPrice = $("#total-price");
        SelenideElement submitButton = $("#submit-button");

        // Assert
        softly.assertThat(totalPrice.getText()).as("Total price check").isEqualTo("$100.00");
        softly.assertThat(submitButton.isEnabled()).as("Submit button enabled").isTrue();
        softly.assertThat(submitButton.has(cssClass("btn-success"))).as("Button class check").isTrue();

        // Проверка всех ассертов
        softly.assertAll();
        screenshot("checkout_page");
    }
}
```

Для TestNG:

```java
import io.qameta.allure.Step;
import org.testng.asserts.SoftAssert;
import org.testng.annotations.Test;
import static com.codeborne.selenide.Selenide.*;

public class CheckoutPageTestNG {
    @Test
    @Step("Валидация страницы оформления заказа")
    void testCheckoutPage() {
        // Given
        SoftAssert softly = new SoftAssert();
        open("/checkout");

        // When
        SelenideElement totalPrice = $("#total-price");
        SelenideElement submitButton = $("#submit-button");

        // Then
        softly.assertEquals(totalPrice.getText(), "$100.00", "Total price check");
        softly.assertTrue(submitButton.isEnabled(), "Submit button enabled");
        softly.assertTrue(submitButton.has(cssClass("btn-success")), "Button class check");

        // Проверка всех ассертов
        softly.assertAll();
        screenshot("checkout_page");
    }
}
```

## Отладка

Для отладки в IntelliJ IDEA:
- Устанавливайте точки останова в местах выполнения ассертов.
- Проверяйте значения элементов (текст, атрибуты) в окне Debug.
- Используйте DevTools браузера для анализа локаторов и классов.
- Настройте Allure для просмотра подробных отчётов с указанием причин падения.

## Источники
- [AssertJ Documentation](https://assertj.github.io/doc/)
- [Selenide Documentation](https://selenide.org/documentation.html)
- [Allure Framework](https://docs.qameta.io/allure/)

---
[**← Назад к оглавлению**](../README.md)