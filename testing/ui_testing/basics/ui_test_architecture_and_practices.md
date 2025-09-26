# Практика автоматизации UI: архитектура, взаимодействие, стабильность

Автоматизация пользовательского интерфейса (UI) охватывает широкий спектр задач — от открытия браузера до проверки поведения элементов на странице. Ниже структурированы ключевые аспекты, необходимые для построения надёжных UI-тестов: типы приложений, нестабильность элементов, действия пользователя, архитектура тестов, именование переменных и взаимодействие с фреймворком Selenium.

---

## 🧭 Типы пользовательских интерфейсов

UI может быть реализован в разных формах:

- **Web App** — веб-приложение, работающее в браузере.
- **Browser App** — расширения или приложения, встроенные в браузер.
- **Mobile App** — приложения для мобильных устройств.
- **Native App** — приложения, написанные под конкретную платформу (например, Android, iOS, Windows).

Автоматизация UI чаще всего применяется к Web App и Mobile App, используя инструменты вроде Selenium и Appium.

---

## ⚠️ Нестабильность веб-элементов

В реальных приложениях элементы могут вести себя непредсказуемо:

- **Нестабильные атрибуты** — `class`, `id`, особенно в React-приложениях, могут меняться динамически.
- **Элемент загружен, но не отображается** — присутствует в DOM, но скрыт стилями.
- **Элемент загружен, но не активен** — не принимает действия (например, `disabled`).

Решения:
- Использовать надёжные селекторы (`id`, `name`, `data-*`).
- Применять ожидания (`ExpectedConditions`) для проверки состояния.
- Проверять `isDisplayed()`, `isEnabled()` перед взаимодействием.

---

## 🖱️ Проверка действий пользователя

UI-тесты имитируют действия пользователя:

- **Клик (`click`)** — выбор, подтверждение.
- **Ввод (`input`)** — заполнение форм.
- **Просмотр текста** — проверка содержимого.
- **Наведение (`hover`)** — раскрытие меню, всплывающих окон.

Пример:
```java
WebElement input = driver.findElement(By.id("username"));
input.sendKeys("John");

WebElement button = driver.findElement(By.id("submit"));
button.click();
```

---

## 🧰 Библиотеки и фреймворки

Для автоматизации UI используются:

- **Selenium** — основной инструмент для браузерной автоматизации.  
  [Официальный сайт](https://www.selenium.dev/)  
  [Документация](https://www.selenium.dev/documentation/)

- **WebDriverManager** — автоматическая загрузка драйверов.
- **JUnit/TestNG** — фреймворки для написания тестов.
- **Allure** — генерация отчётов.
- **AssertJ** — расширенные утверждения.

---

## 🧪 API, классы и методы Selenium

### Открытие браузера:
```java
WebDriver driver = new ChromeDriver();
driver.get("https://example.com");
// или
driver.navigate().to("https://example.com");
```

### Поиск элемента:
```java
WebElement element = driver.findElement(By.xpath("//h1[@class='name']"));
element.click();
element.sendKeys("John");
```

### Стратегии поиска:
- `By.id("...")`
- `By.name("...")`
- `By.className("...")`
- `By.cssSelector("...")`
- `By.xpath("...")`
- `By.tagName("...")`
- `By.linkText("...")`
- `By.partialLinkText("...")`

---

## 🧱 Проблемы и решения

### Проблемы:
- **Нестабильные локаторы** — динамические атрибуты.
- **Время ожидания** — элементы появляются с задержкой.

### Решения:
- Использовать `Explicit Wait`:
```java
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
WebElement element = wait.until(ExpectedConditions.elementToBeClickable(By.id("submit")));
```

- Применять `FluentWait` для гибкости.

---

## 🧪 Шаблон AAA (Arrange-Act-Assert)

Структура теста:
1. **Arrange** — открыть браузер, перейти на страницу, заполнить поля.
2. **Act** — выполнить действие (клик, ввод).
3. **Assert** — проверить результат.

Пример:
```java
@Test
void testLogin() {
    // Arrange
    driver.get("https://example.com");
    driver.findElement(By.id("username")).sendKeys("admin");

    // Act
    driver.findElement(By.id("login")).click();

    // Assert
    assertTrue(driver.findElement(By.id("welcome")).isDisplayed());
}
```

---

## 🧾 Именование переменных

Хорошее имя переменной отражает её назначение и тип:

```java
String inputUserXpath;
By inputUserByXpath;
WebElement inputUserWebElement;

WebElement errorMessage;
WebElement errorUser;
WebElement errorPassword;
```

Для классов:
```java
public class TempoPizza {}
```

Для страниц:
```java
public class LoginPage {
    private final String INPUT_LOGIN = "//input[@id='login']";
}
```

---

## 🔄 Жизненный цикл теста

Использование аннотаций JUnit:

```java
@BeforeEach
void setUp() {
    driver = new ChromeDriver();
}

@AfterEach
void tearDown() {
    driver.quit();
}
```

Обеспечивает чистоту окружения и независимость тестов.

---

## 🔗 Источники:
- [Selenium WebDriver](https://www.selenium.dev/)
- [Selenium Documentation](https://www.selenium.dev/documentation/)
- [W3Schools HTML Reference](https://www.w3schools.com/html/)
- [XPath Syntax](https://www.w3schools.com/xml/xpath_syntax.asp)

---
[**← Назад к оглавлению**](../../../README.md)