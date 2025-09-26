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

Это особенно полезно при работе с формами, где кнопка отправки не имеет явного селектора или поведение зависит от клавиатуры.

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

Это удобно при проверке нескольких условий в одном тесте, особенно при анализе списков или таблиц.

---

## 🧱 Page Object как архитектура

Page Object — это основа UI-фреймворка. Он описывает:

- элементы страницы (`By`, `WebElement`);
- действия над ними (`click()`, `sendKeys()`);
- переходы между страницами (`return new LoginPage()`).

Пример:
```java
public class LoginPage {
    private final By inputLogin = By.id("username");
    private final By inputPassword = By.id("password");
    private final By buttonSubmit = By.id("submit");

    public HomePage login(String user, String pass) {
        driver.findElement(inputLogin).sendKeys(user);
        driver.findElement(inputPassword).sendKeys(pass);
        driver.findElement(buttonSubmit).click();
        return new HomePage();
    }
}
```

---

## 📦 Разделение локаторов: `HomePageLocators`

Иногда локаторы выносят в отдельный класс, чтобы отделить структуру страницы от логики действий:

```java
public class HomePageLocators {
    public static final By ACCEPT_COOKIES = By.id("acceptCookies");
    public static final By SEARCH_FIELD = By.name("search");
}
```

Это удобно при работе с несколькими языками, динамическими атрибутами или переиспользуемыми компонентами.

---

## 🧩 Шаги и действия: `Steps`-классы

Если на странице много полей, создают **комплексные методы** — шаги:

```java
public void fillLoginFormStep(String user, String pass) {
    enterLogin(user);
    enterPassword(pass);
    clickSubmit();
}
```

Для организации сценариев выносят действия в отдельный класс — например, `HomeSteps`:

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

Это позволяет:
- отделить логику теста от реализации;
- переиспользовать шаги;
- повысить читаемость тестов.

---

## 🔗 Источники:
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/)
- [AssertJ Documentation](https://assertj.github.io/doc/)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [Design Patterns for Test Automation](https://refactoring.guru/ru/design-patterns)

---
[**← Назад к оглавлению**](../../../README.md)