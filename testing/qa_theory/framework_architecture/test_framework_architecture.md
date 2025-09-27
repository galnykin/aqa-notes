# Архитектура тестового фреймворка: структура, шаблоны, принципы

Построение надёжного и масштабируемого тестового фреймворка — ключ к успешной автоматизации. Ниже представлена структурированная модель проекта, основанная на лучших практиках: разделение кода, шаблоны проектирования, принципы организации, а также подготовка к финальному проекту.

---

## 🏗️ Название проекта

**test-automated-framework** — пример имени Maven-проекта, отражающего его назначение.

---

## 📁 Структура проекта

Проект делится на две основные части: код приложения и тесты.

```
test-automated-framework
│
├── pom.xml
├── src
│   ├── main
│   │   └── java
│   │       └── com.companyname
│   │           ├── pages       // Page Object классы
│   │           ├── utils       // Утилиты, вспомогательные классы
│   │           └── core        // Базовые компоненты (например, WebDriverSingleton)
│   └── test
│       └── java
│           └── com.companyname
│               ├── tests       // Классы с @Test
│               └── data        // DataProvider, тестовые данные
```

### `pom.xml`
- Управление зависимостями: JUnit, Selenium, WebDriverManager, Allure.
- Настройка плагинов: Surefire, Compiler, Report.

---

## 📦 Структуризация классов

### Пакет `pages`
Содержит Page Object классы, описывающие элементы и действия на страницах:
```java
public class LoginPage {
    private final By inputLogin = By.id("username");
    private final By buttonSubmit = By.id("submit");

    public LoginPage login(String user, String pass) {
        sendKeys(inputLogin, user);
        click(buttonSubmit);
        return new HomePage();
    }
}
```

### Пакет `utils`
Содержит вспомогательные классы: генераторы данных, конвертеры, валидаторы.

### Пакет `core`
Содержит инфраструктурные компоненты, например, `WebDriverSingleton`:
```java
public class WebDriverSingleton {
    private static WebDriver driver;

    public static WebDriver getDriver() {
        if (driver == null) {
            driver = new ChromeDriver();
            driver.manage().window().maximize();
            // другие настройки
        }
        return driver;
    }

    public static void quit() {
        if (driver != null) {
            driver.quit();
            driver = null;
        }
    }
}
```

---

## 🧪 Структуризация тестов

### Пакет `tests`
Содержит классы с аннотацией `@Test`, сгруппированные по функциональности:
- `LoginTest`
- `CartTest`
- `SearchTest`

Каждый тест наследуется от `BaseTest`, где описаны общие действия:
```java
public class BaseTest {
    protected WebDriver driver;

    @BeforeEach
    void openBrowserAndCloseCookie() {
        driver = WebDriverSingleton.getDriver();
        driver.get("https://example.com");
        closeCookieBanner();
    }

    @AfterEach
    void tearDown() {
        WebDriverSingleton.quit();
    }
}
```

---

## 🧱 Шаблоны проектирования

### Page Object
- Описание элементов через `By`
- Методы взаимодействия
- Цепочка вызовов:
  ```java
  public HomePage clickAcceptAllCookiesButton() {
      clickElement(ACCEPT_ALL_COOKIES_BUTTON);
      return this;
  }

  public LoginPage goToLoginPage() {
      clickElement(LOGIN_BUTTON);
      return new LoginPage();
  }
  ```

### Singleton
- Управление WebDriver через единый доступ
- Изоляция от тестов

### Fail-Fast
- Быстрое завершение теста при первой ошибке
- Упрощает отладку и ускоряет диагностику

---

## 🧬 Принципы организации

- **Изоляция**: тесты не должны знать о реализации страниц.
- **Прокладка**: WebDriverSingleton — прослойка между тестами и Selenium.
- **Чистота**: тесты содержат только действия и ассерты.

---

## 🧪 Финальный проект

Автоматизация трёх ключевых сценариев:
1. **Форма логина** — проверка авторизации.
2. **Корзина** — добавление и удаление товара.
3. **Форма поиска** — поиск конкретного товара.

### Работа с группой элементов:
```java
List<WebElement> results = driver.findElements(By.className("product"));
for (WebElement product : results) {
    Assertions.assertTrue(product.getText().contains("Pizza"));
}
```

---

## ⚙️ Дополнительно

### JavaScript Executor
Для нестандартных действий:
```java
JavascriptExecutor js = (JavascriptExecutor) driver;
js.executeScript("arguments[0].scrollIntoView(true);", element);
```

### DataProvider (в TestNG или JUnit 5)
Для параметризации тестов:
```java
@ParameterizedTest
@CsvSource({
    "admin, 1234",
    "user, pass"
})
void testLogin(String username, String password) {
    loginPage.login(username, password);
    assertTrue(homePage.isLoggedIn());
}
```

---

## 🔗 Источники:
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [WebDriverManager by Bonigarcia](https://github.com/bonigarcia/webdrivermanager)
- [Refactoring Guru — Шаблоны проектирования](https://refactoring.guru/ru/design-patterns)
- [XPath Syntax Reference](https://www.w3schools.com/xml/xpath_syntax.asp)

---
[**← Назад к оглавлению**](../README.md)