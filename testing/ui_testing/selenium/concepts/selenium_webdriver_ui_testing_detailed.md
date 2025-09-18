### Selenium WebDriver для UI-тестирования: WebDriverManager, пути к драйверам, действия с элементами, отладка XPath в IntelliJ IDEA, обработка NoSuchElementException, JUnit Jupiter зависимости, борьба с Flaky тестами, неявные и явные ожидания, Page Object Model

В этом руководстве подробно рассмотрены указанные аспекты работы с **Selenium WebDriver** в контексте автоматизации тестирования (AQA). Мы разберём использование WebDriverManager от Boni Garcia, пути для драйверов, действия с веб-элементами (`.click()`, `.sendKeys()`, `.getText()`), отладку XPath в IntelliJ IDEA, обработку исключений (`NoSuchElementException`), зависимости JUnit Jupiter, проблему **flaky тестов**, неявные и явные ожидания, а также Page Object Model (POM). Всё это связано с другими темами, такими как локаторы, JSON Wire Protocol, клиент-серверная модель и тестирование.

---

## 1. WebDriverManager (Boni Garcia)

### 1.1. Описание
- **WebDriverManager** — библиотека от Boni Garcia для автоматического управления драйверами браузеров (ChromeDriver, GeckoDriver и т.д.). Она устраняет необходимость вручную скачивать и указывать путь к драйверу.
- **Преимущества**:
    - Автоматически загружает подходящую версию драйвера для браузера.
    - Кроссплатформенность (Windows, macOS, Linux).
    - Упрощает настройку тестов.

### 1.2. Установка
- **Maven зависимость**:
  ```xml
  <dependency>
      <groupId>io.github.bonigarcia</groupId>
      <artifactId>webdrivermanager</artifactId>
      <version>5.9.2</version>
  </dependency>
  ```

### 1.3. Использование
- Вместо указания пути к драйверу (например, `C:\Users\username\.cache\selenium\chromedriver\win64\139.0.7258.66\chromedriver.exe`):
  ```java
  import io.github.bonigarcia.wdm.WebDriverManager;
  import org.openqa.selenium.WebDriver;
  import org.openqa.selenium.chrome.ChromeDriver;

  public class WebDriverSetup {
      public static void main(String[] args) {
          WebDriverManager.chromedriver().setup(); // Автоматическая загрузка драйвера
          WebDriver driver = new ChromeDriver();
          driver.get("https://example.com");
          driver.quit();
      }
  }
  ```

### 1.4. Применение в AQA
- Упрощает настройку тестов в CI/CD (Jenkins, GitHub Actions), так как не требует ручного управления драйверами.
- Пример в тесте:
  ```java
  @BeforeEach
  void setUp() {
      WebDriverManager.chromedriver().setup();
      driver = new ChromeDriver();
  }
  ```

---

## 2. Путь для WebDriver

- **Ручной путь**:
    - Если не использовать WebDriverManager, нужно указать путь к драйверу:
      ```java
      System.setProperty("webdriver.chrome.driver", "C:\\Users\\username\\.cache\\selenium\\chromedriver\\win64\\139.0.7258.66\\chromedriver.exe");
      WebDriver driver = new ChromeDriver();
      ```
- **Проблемы**:
    - Драйверы устаревают с обновлением браузеров.
    - Разные пути для разных ОС.
- **Рекомендация**: Используйте **WebDriverManager** для автоматического управления.

---

## 3. Действия с веб-элементами

### 3.1. `.click()`
- **Описание**: Выполняет клик по элементу.
- **Поддерживаемые теги**:
    - `<a>` (ссылки).
    - `<button>` (кнопки).
    - `<input type="button">`, `<input type="submit">` (кнопки ввода).
    - `<div>` (если настроено событие `onclick`).
- **Пример**:
  ```java
  driver.findElement(By.id("submit")).click();
  ```

### 3.2. `.sendKeys("...")`
- **Описание**: Вводит текст в поле ввода.
- **Поддерживаемые теги**: `<input>`, `<textarea>`.
- **Пример**:
  ```java
  driver.findElement(By.id("username")).sendKeys("John");
  ```

### 3.3. `.getText()`
- **Описание**: Получает текст из элемента.
- **Поддерживаемые теги**: `<div>`, `<p>`, `<span>`, `<h1>` и др.
- **Пример**:
  ```java
  String message = driver.findElement(By.id("message")).getText();
  assertEquals("Welcome, John", message);
  ```

---

## 4. Отладка XPath в IntelliJ IDEA

### 4.1. Проверка XPath
- **Проблема**: Неправильный XPath приводит к `NoSuchElementException`.
- **Решение в IntelliJ**:
    1. **Создайте тестовый код**:
       ```java
       WebElement element = driver.findElement(By.xpath("//input[@id='username']"));
       ```
    2. **Запустите в режиме отладки**:
        - Установите точку останова (breakpoint) на строке с `findElement`.
        - Нажмите **Debug** (Shift+F9).
        - Проверьте значение локатора в **Variables** или **Evaluate Expression** (Alt+F8).
    3. **Плагин для XPath**:
        - Установите плагин **XPathView + XSLT** в IntelliJ.
        - Используйте **Tools → XPathView → Show XPath Evaluator** для проверки XPath на HTML-странице.
    4. **Браузерные инструменты**:
        - В Chrome DevTools (F12) используйте `Ctrl+F` в вкладке Elements для проверки XPath.

### 4.2. Пример
```java
String xpathTemplate = "//h1[@class='name']";
WebElement element = driver.findElement(By.xpath(xpathTemplate));
// Отладка: проверить, возвращает ли XPath нужный элемент
```

---

## 5. Исключение `NoSuchElementException`

- **Описание**: Возникает, когда Selenium не может найти элемент по указанному локатору.
- **Причины**:
    - Неправильный локатор (например, неверный `id` или XPath).
    - Элемент ещё не загрузился (асинхронная загрузка).
    - Элемент не существует на странице.
- **Решение**:
    - Проверьте локатор в DevTools.
    - Добавьте ожидания (см. ниже).
    - Используйте `try-catch`:
      ```java
      try {
          WebElement element = driver.findElement(By.id("nonexistent"));
      } catch (NoSuchElementException e) {
          System.out.println("Элемент не найден: " + e.getMessage());
      }
      ```

---

## 6. Зависимости JUnit Jupiter

### 6.1. Описание
- **JUnit Jupiter** — модуль JUnit 5 для написания тестов.
- **Зависимости** (с сайта [mvnrepository.com](https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-engine)):
    - `junit-jupiter-engine`: Исполняемый движок для запуска тестов.
    - `junit-jupiter-api`: API для написания тестов.
    - `junit-jupiter-params`: Поддержка параметризованных тестов.

### 6.2. Maven зависимости
```xml
<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-engine</artifactId>
        <version>5.11.0</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>5.11.0</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-params</artifactId>
        <version>5.11.0</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### 6.3. Пример параметризованного теста
```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.ValueSource;

public class LoginTest {
    @ParameterizedTest
    @ValueSource(strings = {"John", "Alice"})
    void testLogin(String username) {
        WebDriver driver = new ChromeDriver();
        driver.get("https://example.com");
        driver.findElement(By.id("username")).sendKeys(username);
        driver.findElement(By.id("submit")).click();
        assertEquals("Welcome, " + username, driver.findElement(By.id("message")).getText());
        driver.quit();
    }
}
```

---

## 7. Flaky тесты

### 7.1. Описание
- **Flaky тесты** — тесты, которые иногда проходят, а иногда падают из-за непредсказуемого поведения (например, из-за асинхронной загрузки).
- **Причины**:
    - Динамические элементы (React, SPA).
    - Сетевые задержки.
    - Неправильные или отсутствующие ожидания.

### 7.2. Решения
- **Явные ожидания** (см. ниже).
- **Стабильные локаторы**: Используйте `data-testid`.
- **Повторы тестов**:
    - В JUnit 5 используйте `@RepeatedTest` или плагины (например, `maven-surefire-plugin` с `rerunFailingTestsCount`).
- **Логирование**: Добавьте логи для отладки:
  ```java
  System.out.println("Элемент: " + driver.findElement(By.id("submit")).isDisplayed());
  ```

---

## 8. Ожидания в Selenium

### 8.1. Неявные ожидания
- **Описание**: Задают время ожидания для всех операций поиска элементов. Применяются при создании драйвера.
- **Пример**:
  ```java
  driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(2));
  driver.findElement(By.id("submit")).click(); // Ждёт до 2 секунд
  ```
- **Проблемы**:
    - Увеличивает время выполнения тестов, если много элементов.
    - Негибко: одинаковое время для всех операций.
- **Рекомендация**: Используйте неявные ожидания только для начальной настройки, предпочтите явные.

### 8.2. Явные ожидания
- **Описание**: Ожидание конкретного условия для элемента (например, видимость, кликабельность).
- **Класс**: `WebDriverWait` с `ExpectedConditions`.
- **Пример**:
  ```java
  WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
  wait.until(ExpectedConditions.elementToBeClickable(By.id("submit"))).click();
  ```
- **Популярные условия**:
    - `presenceOfElementLocated`: Элемент в DOM.
    - `visibilityOfElementLocated`: Элемент видим.
    - `elementToBeClickable`: Элемент кликабелен.
- **Преимущества**:
    - Гибкость: Ожидание только для нужных элементов.
    - Уменьшает вероятность flaky тестов.

### 8.3. Ссылка
- [Selenium Waits Documentation](https://www.selenium.dev/documentation/webdriver/waits/)

---

## 9. Page Object Model (POM)

### 9.1. Описание
- **POM** — паттерн проектирования для структурирования тестов, где каждая страница представлена классом с локаторами и методами.
- **Преимущества**:
    - Упрощает поддержку тестов.
    - Повторное использование кода.
    - Читаемость.

### 9.2. Пример
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

### 9.3. Тест с POM
```java
public class TempoPizzaTest {
    private WebDriver driver;
    private LoginPage loginPage;

    @BeforeEach
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        loginPage = new LoginPage(driver);
    }

    @Test
    public void testLogin() {
        // Arrange
        driver.get("https://temppizza.com");
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

## 10. Связь с другими темами

- **Локаторы и селекторы**:
    - Стабильные локаторы (`data-testid`) решают проблемы с React-приложениями:
      ```java
      driver.findElement(By.cssSelector("[data-testid='submit']"));
      ```

- **JSON Wire Protocol и RESTful Web Service**:
    - WebDriver отправляет команды через RESTful API (например, `POST /element` для `.findElement()`).
    - Ожидания (`WebDriverWait`) синхронизируют API-запросы.

- **Клиент-серверная модель**:
    - WebDriver (клиент) взаимодействует с драйвером (сервером), что важно для обработки `NoSuchElementException`.

- **Тестирование (JUnit-подобные подходы)**:
    - AAA структура в JUnit 5:
      ```java
      @Test
      void testLogin() {
          // Arrange
          LoginPage page = new LoginPage(driver);
          // Act
          page.clickLogin();
          // Assert
          assertEquals("Error", page.getErrorMessage());
      }
      ```

- **ООП**:
    - POM использует ООП для инкапсуляции локаторов и действий.

---

## 11. Рекомендации

- **Используйте WebDriverManager**:
    - Избегайте ручного указания пути к драйверу.
- **Стабильные локаторы**:
    - Предпочитайте `data-testid` или комбинированные CSS/XPath.
- **Ожидания**:
    - Используйте явные ожидания (`WebDriverWait`) вместо неявных.
- **Отладка**:
    - Проверяйте XPath в IntelliJ (Debug или плагин) и DevTools.
- **Обработка исключений**:
    - Ловите `NoSuchElementException`:
      ```java
      try {
          driver.findElement(By.id("submit")).click();
      } catch (NoSuchElementException e) {
          fail("Элемент не найден: " + e.getMessage());
      }
      ```
- **POM**:
    - Структурируйте тесты с Page Object Model.
- **Flaky тесты**:
    - Добавляйте ожидания и логирование для диагностики.
- **CI/CD**:
    - Интегрируйте тесты с Jenkins/GitHub Actions:
      ```bash
      mvn test
      ```

---

## 12. Источники
- [Selenium Documentation](https://www.selenium.dev/documentation/)
- [WebDriverManager](https://bonigarcia.dev/webdrivermanager/)
- [Baeldung: Selenium WebDriver](https://www.baeldung.com/java-selenium)
- [mvnrepository: JUnit Jupiter](https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-engine)
- [Selenium Waits](https://www.selenium.dev/documentation/webdriver/waits/)

[**&#x2190; Назад к оглавлению**](../README.md)