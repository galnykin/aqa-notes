# Практика и устойчивость UI-тестов: Selenium, ожидания, взаимодействие

Автоматизация UI-тестирования позволяет проверять поведение веб-приложений через программное взаимодействие с браузером. Selenium WebDriver — один из самых популярных инструментов для этой задачи. Он обеспечивает гибкость, кросс-браузерность и богатый набор возможностей для работы с веб-элементами, ожиданиями и пользовательскими действиями.

---

## 🧰 Selenium WebDriver: архитектура и запуск

Selenium WebDriver — это API для управления браузером программно. Он позволяет открывать страницы, находить элементы, выполнять действия и проверять результаты.

### Инициализация драйвера:
```java
WebDriver driver = new ChromeDriver();
driver.get("https://example.com");
```

Также можно использовать цепочку вызовов:
```java
driver.navigate().to("https://example.com");
```

### Подключение WebDriverManager (Bonigarcia)

Чтобы не скачивать драйверы вручную и не прописывать путь к ним, используется библиотека WebDriverManager:

```xml
<dependency>
    <groupId>io.github.bonigarcia</groupId>
    <artifactId>webdrivermanager</artifactId>
    <version>5.6.3</version>
</dependency>
```

Пример использования:
```java
WebDriverManager.chromedriver().setup();
WebDriver driver = new ChromeDriver();
```

Преимущества:
- автоматическая загрузка нужного драйвера;
- совместимость с версией браузера;
- отсутствие необходимости прописывать путь вручную;
- стабильная работа в CI/CD.

### Архитектура Selenium WebDriver
- **Клиент-серверная модель**: Тестовый код (клиент) отправляет команды драйверу (серверу), который взаимодействует с браузером. Это обеспечивает платформонезависимость.
- **JSON Wire Protocol**: В старых версиях использовался для передачи команд через RESTful API. В Selenium 4 заменён W3C Protocol для лучшей совместимости.
- **Драйверы**: ChromeDriver, GeckoDriver, EdgeDriver — реализуют протокол для конкретных браузеров.

### Запуск тестов
- Используйте JUnit 5 или TestNG для структурирования:
  ```java
  import org.junit.jupiter.api.Test;
  import org.openqa.selenium.WebDriver;
  import org.openqa.selenium.chrome.ChromeDriver;

  public class SimpleTest {
      @Test
      void testOpenPage() {
          WebDriverManager.chromedriver().setup();
          WebDriver driver = new ChromeDriver();
          driver.get("https://example.com");
          driver.quit();
      }
  }
  ```

### Кросс-браузерное тестирование
- Настройка для разных браузеров:
  ```java
  WebDriverManager.firefoxdriver().setup();
  WebDriver driver = new FirefoxDriver();
  ```

### CI/CD интеграция (GitHub Actions)
```yaml
name: Selenium UI Tests
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up JDK
        uses: actions/setup-java@v3
        with:
          java-version: '17'
      - name: Install dependencies
        run: mvn install
      - name: Run tests
        run: mvn test
```

---

## 🔍 Стратегии поиска веб-элементов

Поиск элементов осуществляется через класс `By`. Selenium поддерживает около 10 стратегий:

```java
By.id("login");
By.name("email");
By.className("btn");
By.tagName("input");
By.linkText("Подробнее");
By.partialLinkText("Подробнее о");
By.cssSelector(".form input[type='text']");
By.xpath("//input[@name='height']");
```

XPath — мощный язык запросов, позволяющий обращаться к узлам DOM. Примеры:
- Абсолютный путь: `/html/body/table/tbody/tr[2]/td[2]/input`
- Относительный путь: `//input[@name='height']`
- По индексу: `//bookstore/book[1]/title`
- По атрибуту: `//input[@type='submit']`

### Поиск нескольких элементов:
```java
List<WebElement> inputs = driver.findElements(By.tagName("input"));
```

Использование `findElements` возвращает список, даже если он пуст, что удобно для проверок.

### Отладка XPath в IntelliJ IDEA
- Запустите тест в режиме отладки (Shift+F9).
- Установите точку останова на строке с `findElement`.
- В окне Debug проверьте значение локатора и элемента.
- Используйте Chrome DevTools (Ctrl+F) для предварительной проверки XPath.

### Обработка NoSuchElementException
- Это исключение возникает, когда элемент не найден.
- Обработайте в try-catch:
  ```java
  try {
      driver.findElement(By.id("nonexistent")).click();
  } catch (NoSuchElementException e) {
      System.out.println("Элемент не найден: " + e.getMessage());
  }
  ```

---

## ⏳ Ожидания: устойчивость тестов

Ожидания позволяют дождаться появления или изменения состояния элемента. Это критично для стабильности тестов.

### Неявные ожидания:
```java
driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(2));
```
Применяются ко всем `findElement` вызовам. Увеличивают общее время выполнения.

### Явные ожидания:
```java
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
WebElement element = wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("login")));
```

### FluentWait:
```java
Wait<WebDriver> wait = new FluentWait<>(driver)
    .withTimeout(Duration.ofSeconds(15))
    .pollingEvery(Duration.ofMillis(500))
    .ignoring(NoSuchElementException.class);

WebElement element = wait.until(driver -> driver.findElement(By.id("submit")));
```

Документация:  
[Ожидания в Selenium](https://www.selenium.dev/documentation/webdriver/waits/)

### Нестабильность элементов и flaky-тесты
Flaky-тесты — тесты, которые иногда падают из-за асинхронности или динамики.
- Причины: Элемент загружен, но не отображается; загружен, но не активен; нестабильные атрибуты (class, id в React).
- Решения: Используйте явные ожидания, стабильные локаторы (data-testid), проверки isDisplayed()/isEnabled().
- Пример проверки:
  ```java
  WebElement button = driver.findElement(By.id("submit"));
  if (button.isDisplayed() && button.isEnabled()) {
      button.click();
  }
  ```

---

## ⚠️ Нестабильность элементов и flaky-тесты

**Flaky test** — тест, который иногда проходит, иногда падает без изменения кода. Причины:

- элемент загружен, но не отображается;
- элемент загружен, но не активен;
- нестабильные атрибуты (`class`, `id`, особенно в React);
- асинхронная загрузка данных.

Решения:
- использовать надёжные селекторы (`id`, `name`, `data-*`);
- применять `ExpectedConditions` для проверки состояния;
- проверять `isDisplayed()`, `isEnabled()` перед действием;
- использовать `WebDriverWait` вместо `Thread.sleep()`.

---

## 🖱️ Взаимодействие с элементами

Selenium позволяет имитировать действия пользователя:

- `click()` — клик по элементу;
- `sendKeys("...")` — ввод текста;
- `getText()` — получение текста;
- `submit()` — отправка формы;
- `clear()` — очистка поля.

### Часто используемые теги:
- `<a>` — ссылки
- `<button>` — кнопки
- `<input type="button">`, `<input type="submit">` — кнопки формы
- `<input>` — поля ввода
- `<div>`, `<p>`, `<span>` — текстовые блоки

### Всплывающие окна (Alerts)
```java
Alert alert = driver.switchTo().alert();
alert.accept(); // подтвердить
alert.dismiss(); // отменить
alert.getText(); // получить сообщение
```

### Cookies
```java
driver.manage().getCookies(); // получить все cookies
driver.manage().addCookie(new Cookie("token", "abc123"));
driver.manage().deleteAllCookies();
```

Cookies полезны для авторизации и управления сессиями.

### Frames (фреймы)
```java
driver.switchTo().frame("frameName");
driver.switchTo().defaultContent(); // вернуться в основной контекст
```

Фреймы требуют явного переключения контекста перед взаимодействием.

### Actions (сложные действия)
```java
Actions actions = new Actions(driver);
actions.moveToElement(element).click().build().perform();
```

Позволяет выполнять drag-and-drop, двойной клик, наведение и другие пользовательские действия.

### JavascriptExecutor
Позволяет выполнять JavaScript напрямую:

```java
JavascriptExecutor js = (JavascriptExecutor) driver;
js.executeScript("arguments[0].scrollIntoView(true);", element);
```

Полезно для скроллинга, получения скрытых данных, обхода нестандартных UI-ограничений.

### Проверка действий пользователя
- **Клики**: `.click()` для `<a>`, `<button>`, `<input type="button/submit">`, `<div>` с onclick.
- **Ввод**: `.sendKeys("...")` для `<input>`.
- **Просмотр текста**: `.getText()` для проверки отображаемого текста.
- **Наведение (hover)**: Используйте `Actions` для имитации hover.

---

## 🧱 Шаблон проектирования Page Object

Page Object — архитектурный шаблон, при котором каждый экран или компонент описывается отдельным классом.

```java
public class LoginPage {
    private final By inputLogin = By.id("username");
    private final By inputPassword = By.id("password");
    private final By buttonSubmit = By.id("submit");

    public void login(String user, String pass) {
        driver.findElement(inputLogin).sendKeys(user);
        driver.findElement(inputPassword).sendKeys(pass);
        driver.findElement(buttonSubmit).click();
    }
}
```

Преимущества:
- повторное использование;
- упрощение поддержки;
- отделение логики теста от структуры страницы.

### Именование переменных в POM
- Локаторы: `inputUserXpath`, `inputUserByXpath`, `inputUserWebElement`.
- Для текста: `errorMessage`, `errorUser`, `errorPassword`.
- Классы: `TempoPizza`.

### @BeforeEach / @AfterEach в тестах
```java
@BeforeEach
void setUp() {
    driver = new ChromeDriver();
    driver.get("https://example.com");
}

@AfterEach
void tearDown() {
    driver.quit();
}
```

---

## 🧪 Шаблон AAA (Arrange-Act-Assert)

Структура теста:
1. **Arrange** — подготовка: открыть браузер, перейти на страницу, заполнить поля.
2. **Act** — действие: клик, ввод, отправка.
3. **Assert** — проверка: результат, состояние, сообщение.

```java
@Test
public void testLogin() {
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

## 🐞 Отладка и распространённые ошибки

### IntelliJ Debug для проверки XPath
- Запустите тест в режиме отладки (Shift+F9).
- Установите точку останова на строке с `findElement`.
- В окне Debug проверьте значение локатора и элемента.
- Используйте Chrome DevTools (Ctrl+F) для предварительной проверки XPath.

### Частые ошибки
- `NoSuchElementException` — элемент не найден.
- Решение: Добавьте ожидания или проверьте локатор.

---

## 📦 Зависимости JUnit Jupiter

Для JUnit 5 используются три зависимости:

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.9.3</version>
</dependency>
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.9.3</version>
</dependency>
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-params</artifactId>
    <version>5.9.3</version>
</dependency>
```

---

## 🔗 Источники:
- [Selenium WebDriver](https://www.selenium.dev/)
- [Selenium Waits](https://www.selenium.dev/documentation/webdriver/waits/)
- [JUnit Jupiter Engine](https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-engine)
- [XPath Syntax](https://www.w3schools.com/xml/xpath_syntax.asp)

---
[**← Назад к оглавлению**](../README.md)