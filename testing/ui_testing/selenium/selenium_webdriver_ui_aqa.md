### Начало работы с Selenium WebDriver

В данном руководстве подробно рассмотрены базовые аспекты работы с **Selenium WebDriver** в контексте автоматизации тестирования (AQA). Мы разберём UI-тестирование (веб-приложения, браузерные приложения, мобильные приложения, нативные элементы), нестабильные атрибуты (class, id, React), проблемы с загрузкой веб-элементов, проверку действий пользователя, библиотеки и зависимости, а также структуру кода с учётом AAA (Arrange-Act-Assert), именование переменных, Page Object Model (POM) и стратегии поиска. Всё это связано с другими темами, такими как локаторы, JSON Wire Protocol, клиент-серверная модель и тестирование.

---

## 1. UI в контексте Selenium WebDriver

### 1.1. Веб-приложения (Web App)
- **Описание**: Веб-приложения — это приложения, доступные через браузер (например, Chrome, Firefox). Selenium WebDriver используется для автоматизации взаимодействия с веб-приложениями.
- **Применение**:
    - Тестирование форм, навигации, отображения контента.
    - Проверка UI на разных разрешениях и браузерах.
- **Пример**:
  ```java
  WebDriver driver = new ChromeDriver();
  driver.get("https://example.com"); // Открытие веб-приложения
  driver.findElement(By.id("login")).sendKeys("user");
  ```

### 1.2. Браузерные приложения (Browser App)
- **Описание**: Приложения, работающие исключительно в браузере (например, SPA на React, Angular). Selenium взаимодействует с DOM через WebDriver.
- **Особенности**:
    - Динамическая загрузка контента (асинхронные запросы).
    - Требуются ожидания для элементов.
- **Пример**:
  ```java
  WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
  wait.until(ExpectedConditions.elementToBeClickable(By.id("submit"))).click();
  ```

### 1.3. Мобильные приложения
- **Описание**: Selenium WebDriver может тестировать мобильные веб-приложения через браузеры на устройствах (например, Chrome на Android). Для нативных приложений используется **Appium**, который расширяет WebDriver API.
- **Применение**:
    - Тестирование мобильных версий сайтов.
    - Проверка адаптивности UI.
- **Пример**:
  ```java
  DesiredCapabilities caps = new DesiredCapabilities();
  caps.setCapability("browserName", "Chrome");
  caps.setCapability("platformName", "Android");
  WebDriver driver = new AndroidDriver(new URL("http://localhost:4723/wd/hub"), caps);
  driver.get("https://example.com");
  ```

### 1.4. Нативные элементы
- **Описание**: Элементы, специфичные для платформы (например, кнопки iOS/Android в нативных приложениях). Selenium не поддерживает их напрямую, но Appium использует WebDriver-протокол для работы с нативными элементами.
- **Применение**:
    - Тестирование UI нативных приложений через Appium.
- **Пример**:
  ```java
  driver.findElement(By.id("com.example:app_button")).click(); // Нативная кнопка
  ```

---

## 2. Нестабильные атрибуты (class, id, React)

### 2.1. Проблемы с нестабильными атрибутами
- **Class**:
    - Часто неуникальны или динамически генерируются (например, `class="btn-123abc"` в React).
    - Проблема: Изменение классов при обновлении стилей ломает тесты.
- **ID**:
    - Обычно уникальны, но в SPA (React) могут быть динамическими (например, `id="component-xyz123"`).
    - Проблема: Генерируемые ID меняются при каждом рендере.
- **React**:
    - React-приложения используют виртуальный DOM и динамические атрибуты.
    - Проблема: Отсутствие стабильных `id` или `class`, частое использование `data-*` атрибутов.

### 2.2. Решения
- **Используйте `data-*` атрибуты**:
    - Просите разработчиков добавлять `data-testid`:
      ```java
      driver.findElement(By.cssSelector("[data-testid='submit']")).click();
      ```
- **Стабильные локаторы**:
    - Предпочитайте `id`, если он статичен.
    - Используйте CSS или XPath для комбинированных условий:
      ```java
      By.xpath("//button[contains(@class, 'btn') and text()='Login']");
      ```
- **Избегайте индексов**:
    - Хрупкие локаторы, такие как `//div[2]`, ломаются при изменении DOM.

### 2.3. Пример в React-приложении
```java
// Проблемный локатор
driver.findElement(By.className("btn-123abc")).click(); // Динамический класс

// Стабильный локатор
driver.findElement(By.cssSelector("[data-testid='submit']")).click();
```

---

## 3. Проблемы с загрузкой веб-элементов

### 3.1. Элемент не загрузился
- **Описание**: Элемент отсутствует в DOM из-за асинхронной загрузки (например, в SPA).
- **Решение**: Используйте явные ожидания (`WebDriverWait`):
  ```java
  WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
  wait.until(ExpectedConditions.presenceOfElementLocated(By.id("submit")));
  ```

### 3.2. Элемент загружен, но не отображается
- **Описание**: Элемент есть в DOM, но невидим (`display: none` или вне области видимости).
- **Решение**: Проверяйте видимость:
  ```java
  wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("submit"))).click();
  ```

### 3.3. Элемент загружен, но не активен
- **Описание**: Элемент видим, но неактивен (`disabled`).
- **Решение**: Проверяйте атрибут `enabled`:
  ```java
  WebElement button = driver.findElement(By.id("submit"));
  wait.until(ExpectedConditions.elementToBeClickable(button)).click();
  ```

---

## 4. Проверка действий пользователя

### 4.1. Клики (`click`, `select`)
- **Описание**: Проверка кликов по кнопкам, ссылкам или выпадающим спискам.
- **Пример**:
  ```java
  driver.findElement(By.id("submit")).click();
  // Для select
  Select dropdown = new Select(driver.findElement(By.id("dropdown")));
  dropdown.selectByVisibleText("Option 1");
  ```

### 4.2. Ввод (`input`)
- **Описание**: Ввод текста в поля ввода.
- **Пример**:
  ```java
  driver.findElement(By.id("username")).sendKeys("John");
  ```

### 4.3. Просмотр текста
- **Описание**: Проверка текста в элементах (`div`, `p`, `span`).
- **Пример**:
  ```java
  String text = driver.findElement(By.id("message")).getText();
  assertEquals("Welcome, John", text);
  ```

### 4.4. Наведение (`hover`)
- **Описание**: Имитация наведения мыши.
- **Пример**:
  ```java
  Actions actions = new Actions(driver);
  WebElement element = driver.findElement(By.id("menu"));
  actions.moveToElement(element).perform();
  ```

---

## 5. Библиотеки, зависимости, фреймворки

### 5.1. Selenium WebDriver
- **Описание**: Основной инструмент для автоматизации браузеров. Использует WebDriver API для взаимодействия с браузерами через драйверы (ChromeDriver, GeckoDriver).
- **Зависимости**:
    - Maven:
      ```xml
      <dependency>
          <groupId>org.seleniumhq.selenium</groupId>
          <artifactId>selenium-java</artifactId>
          <version>4.25.0</version>
      </dependency>
      ```
- **Фреймворки**:
    - **JUnit 5**: Для структурированных тестов.
    - **TestNG**: Альтернатива JUnit с дополнительными функциями.
    - **Serenity**: Для BDD-тестирования с отчётами.

### 5.2. Драйверы
- **ChromeDriver**, **SafariDriver**, **GeckoDriver**:
    - Драйверы для конкретных браузеров, реализующие WebDriver-протокол.
    - Скачиваются отдельно или через WebDriverManager:
      ```xml
      <dependency>
          <groupId>io.github.bonigarcia</groupId>
          <artifactId>webdrivermanager</artifactId>
          <version>5.9.2</version>
      </dependency>
      ```
      ```java
      WebDriverManager.chromedriver().setup();
      WebDriver driver = new ChromeDriver();
      ```

---

## 6. API, классы, методы Selenium WebDriver

### 6.1. Открытие браузера
- **Методы**:
    - `get("url")`: Открывает URL.
    - `navigate().to("url")`: То же, с возможностью цепочки вызовов.
- **Пример**:
  ```java
  WebDriver driver = new ChromeDriver();
  driver.get("https://example.com");
  // или
  driver.navigate().to("https://example.com").refresh();
  ```

### 6.2. Поиск элементов
- **Метод**: `findElement(By locator)` возвращает `WebElement`.
- **Пример**:
  ```java
  WebElement element = driver.findElement(By.id("username"));
  element.sendKeys("John");
  element.click();
  ```

### 6.3. Основные классы
- **WebDriver**: Интерфейс для управления браузером.
- **WebElement**: Представляет элемент на странице (кнопка, поле ввода).
- **By**: Класс для стратегий поиска.
- **WebDriverWait**: Для явных ожиданий.

---

## 7. Стратегии поиска (около 10)
- **Основные**:
    - `By.id("id")`: По уникальному ID.
    - `By.name("name")`: По атрибуту `name`.
    - `By.className("class")`: По классу.
    - `By.tagName("tag")`: По тегу.
    - `By.linkText("text")`: По тексту ссылки.
    - `By.partialLinkText("text")`: По части текста ссылки.
    - `By.cssSelector("selector")`: CSS-селектор.
    - `By.xpath("expression")`: XPath.
- **Примеры**:
  ```java
  String xpathTemplate = "//h1[@class='name']";
  driver.findElement(By.xpath(xpathTemplate));
  
  String cssSelector = "input#username";
  driver.findElement(By.cssSelector(cssSelector));
  ```

---

## 8. Проблемы и решения

### 8.1. Стабильные локаторы
- **Проблема**: Динамические `id`/`class` в React-приложениях.
- **Решение**:
    - Используйте `data-testid`:
      ```java
      driver.findElement(By.cssSelector("[data-testid='submit']"));
      ```
    - Комбинируйте атрибуты:
      ```java
      By.xpath("//input[@type='text' and @name='username']");
      ```

### 8.2. Время ожидания
- **Проблема**: Элементы не загружаются вовремя.
- **Решение**: Явные ожидания (`WebDriverWait`):
  ```java
  WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
  wait.until(ExpectedConditions.elementToBeClickable(By.id("submit"))).click();
  ```

---

## 9. AAA (Arrange-Act-Assert)
- **Arrange**: Подготовка (открытие браузера, переход на страницу, заполнение полей).
- **Act**: Выполнение действия (клик, ввод).
- **Assert**: Проверка результата.
- **Пример**:
  ```java
  @Test
  public void testLogin() {
      // Arrange
      WebDriver driver = new ChromeDriver();
      driver.get("https://example.com");
      driver.findElement(By.id("username")).sendKeys("John");
      // Act
      driver.findElement(By.id("submit")).click();
      // Assert
      assertEquals("Welcome", driver.findElement(By.id("message")).getText());
      driver.quit();
  }
  ```

---

## 10. Именование переменных
- **Общие правила**:
    - Используйте описательные имена, отражающие назначение элемента.
    - Следуйте бизнес-логике: `inputUser`, `buttonLogin`, `errorMessage`.
- **Примеры**:
    - Локаторы:
      ```java
      String inputUserXpath = "//input[@id='username']";
      By inputUserByXpath = By.xpath(inputUserXpath);
      WebElement inputUserWebElement = driver.findElement(inputUserByXpath);
      ```
    - Возвращаемый текст (`div`, `p`, `span`):
      ```java
      WebElement errorUser = driver.findElement(By.id("error-user"));
      WebElement message = driver.findElement(By.cssSelector("div.message"));
      ```

---

## 11. Page Object Model (POM)
- **Описание**: Инкапсуляция локаторов и действий в классах для упрощения поддержки тестов.
- **Пример**:
  ```java
  public class LoginPage {
      private final WebDriver driver;
      private final By inputUser = By.id("username");
      private final By buttonLogin = By.cssSelector("[data-testid='submit']");
      private final By errorMessage = By.id("error");

      public LoginPage(WebDriver driver) {
          this.driver = driver;
      }

      public void enterUsername(String username) {
          driver.findElement(inputUser).sendKeys(username);
      }

      public void clickLogin() {
          driver.findElement(buttonLogin).click();
      }

      public String getErrorMessage() {
          return driver.findElement(errorMessage).getText();
      }
  }
  ```

### Тест с POM
```java
public class TempoPizzaTest {
    private WebDriver driver;

    @BeforeEach
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
    }

    @Test
    public void testLogin() {
        // Arrange
        driver.get("https://temppizza.com");
        LoginPage loginPage = new LoginPage(driver);
        loginPage.enterUsername("John");
        // Act
        loginPage.clickLogin();
        // Assert
        assertEquals("Invalid credentials", loginPage.getErrorMessage());
    }

    @AfterEach
    void tearDown() {
        driver.quit();
    }
}
```

---

## 12. Связь с другими темами

- **Локаторы и селекторы**:
    - Используйте стабильные локаторы (`data-testid`, `id`) для React-приложений:
      ```java
      driver.findElement(By.cssSelector("[data-testid='submit']"));
      ```

- **JSON Wire Protocol и RESTful Web Service**:
    - Selenium WebDriver использует RESTful API для отправки команд (например, `POST /session/{sessionId}/element` для поиска).
    - Проблемы с загрузкой элементов решаются ожиданием через API.

- **Клиент-серверная модель**:
    - WebDriver (клиент) отправляет команды драйверу (серверу), который управляет браузером. Это важно для обработки асинхронных загрузок.

- **Тестирование (JUnit-подобные подходы)**:
    - Структура AAA интегрируется с JUnit 5:
      ```java
      @Test
      void testLogin() {
          // Arrange
          LoginPage page = new LoginPage(driver);
          page.enterUsername("John");
          // Act
          page.clickLogin();
          // Assert
          assertEquals("Error", page.getErrorMessage());
      }
      ```

- **ООП**:
    - POM реализует ООП, инкапсулируя локаторы и действия:
      ```java
      public class LoginPage {
          private final By inputUser = By.id("username");
      }
      ```

---

## 13. Рекомендации
- **Стабильные локаторы**:
    - Используйте `data-testid` или комбинированные CSS/XPath.
- **Ожидания**:
    - Всегда добавляйте `WebDriverWait` для динамических элементов.
- **POM**:
    - Структурируйте тесты с Page Object Model для поддержки.
- **Именование**:
    - Следуйте бизнес-логике: `inputUser`, `buttonLogin`.
- **Тестирование**:
    - Запускайте тесты после изменений:
      ```bash
      mvn test
      ```
- **CI/CD**:
    - Интегрируйте тесты с Jenkins/GitHub Actions.

---

## 14. Источники
- [Selenium Documentation](https://www.selenium.dev/documentation/)
- [Baeldung: Selenium WebDriver](https://www.baeldung.com/java-selenium)
- [BrowserStack: Selenium Locators](https://www.browserstack.com/guide/locators-in-selenium)
- [LambdaTest: Selenium Waits](https://www.lambdatest.com/learning-hub/selenium-waits)

---

[**&#x2190; Назад к оглавлению**](README.md)