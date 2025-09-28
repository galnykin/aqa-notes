# Автоматизация тестирования пользовательского интерфейса (UI) с Selenium

Автоматизация тестирования пользовательского интерфейса (UI) с использованием Selenium WebDriver — это мощный подход для проверки поведения веб-приложений через программное взаимодействие с браузером. Selenium обеспечивает кросс-браузерную совместимость, гибкость в работе с веб-элементами, а также поддержку сложных сценариев, таких как ожидания, действия пользователя и выполнение JavaScript. Это руководство подробно описывает ключевые аспекты работы с Selenium, включая ожидания, стратегии поиска элементов, взаимодействие с веб-элементами, а также лучшие практики для автоматизации тестирования. Примеры кода приведены на Java (с приоритетом) и JavaScript (с использованием Cypress для UI-тестирования), без Python, как указано в запросе. Руководство также связывает темы с клиент-серверной моделью, локаторами и API-тестированием, чтобы показать их интеграцию в AQA.

---

## Ожидания в Selenium

Ожидания — это механизмы, которые синхронизируют тесты с асинхронной загрузкой веб-страниц, предотвращая нестабильность тестов из-за отсутствия или изменения элементов. Selenium поддерживает несколько типов ожиданий, каждый из которых подходит для определённых сценариев.

### Неявное ожидание (Implicit Wait)
- **Описание**: Устанавливает глобальное время ожидания для всех операций поиска элементов (`findElement`/`findElements`). Если элемент не найден сразу, WebDriver ждёт до истечения времени.
- **Применение в AQA**: Используется для простых тестов, где элементы появляются с предсказуемой задержкой.
- **Недостатки**: Может замедлять тесты, если элемент отсутствует, так как ожидание применяется ко всем операциям поиска.
- **Пример на Java**:
  ```java
  import org.openqa.selenium.WebDriver;
  import org.openqa.selenium.chrome.ChromeDriver;
  import java.time.Duration;

  public class ImplicitWaitExample {
      public static void main(String[] args) {
          WebDriver driver = new ChromeDriver();
          driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
          driver.get("https://example.com");
          driver.findElement(By.id("login")).click();
          driver.quit();
      }
  }
- **Связь с другими темами**: Неявное ожидание полезно в клиент-серверной модели, где UI зависит от ответа API.

### Явное ожидание (Explicit Wait)
- **Описание**: Ожидает выполнения конкретного условия для определённого элемента (например, видимость, кликабельность). Используется `WebDriverWait` с классом `ExpectedConditions`.
- **Применение в AQA**: Тестирование динамических страниц, где элементы появляются асинхронно.
- **Популярные условия**:
    - `visibilityOfElementLocated`: Элемент видим.
    - `elementToBeClickable`: Элемент кликабелен.
    - `presenceOfElementLocated`: Элемент присутствует в DOM.
    - `textToBePresentInElement`: Текст присутствует в элементе.
- **Пример на Java**:
  ```java
  import org.openqa.selenium.By;
  import org.openqa.selenium.WebDriver;
  import org.openqa.selenium.WebElement;
  import org.openqa.selenium.chrome.ChromeDriver;
  import org.openqa.selenium.support.ui.WebDriverWait;
  import org.openqa.selenium.support.ui.ExpectedConditions;
  import java.time.Duration;

  public class ExplicitWaitExample {
      public static void main(String[] args) {
          WebDriver driver = new ChromeDriver();
          driver.get("https://example.com");
          WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
          WebElement element = wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("login")));
          element.click();
          driver.quit();
      }
  }
  
- **Связь с другими темами**: Явные ожидания часто используются с локаторами для точной работы с UI.

### FluentWait
- **Описание**: Настраиваемое ожидание с гибкими параметрами, такими как интервал опроса и игнорируемые исключения. Подходит для сложных сценариев с нестабильными элементами.
- **Применение в AQA**: Тестирование страниц с нерегулярной загрузкой (например, SPA).
- **Пример на Java**:
  ```java
  import org.openqa.selenium.By;
  import org.openqa.selenium.WebDriver;
  import org.openqa.selenium.WebElement;
  import org.openqa.selenium.chrome.ChromeDriver;
  import org.openqa.selenium.support.ui.FluentWait;
  import org.openqa.selenium.support.ui.Wait;
  import org.openqa.selenium.NoSuchElementException;
  import java.time.Duration;

  public class FluentWaitExample {
      public static void main(String[] args) {
          WebDriver driver = new ChromeDriver();
          driver.get("https://example.com");
          Wait<WebDriver> wait = new FluentWait<>(driver)
                  .withTimeout(Duration.ofSeconds(15))
                  .pollingEvery(Duration.ofMillis(500))
                  .ignoring(NoSuchElementException.class);
          WebElement element = wait.until(driver -> driver.findElement(By.id("submit")));
          element.click();
          driver.quit();
      }
  }
 
- **Связь с другими темами**: FluentWait помогает в тестировании UI, зависящем от асинхронных API-ответов.

---

## Стратегии поиска веб-элементов

Selenium предоставляет класс `By` для поиска элементов на странице. Выбор правильной стратегии поиска (локаторов) критичен для стабильности и читаемости тестов.

### Основные локаторы
- **id**: Поиск по уникальному идентификатору элемента.
- **className**: Поиск по имени класса.
- **cssSelector**: Гибкий поиск с использованием CSS-селекторов.
- **linkText**: Поиск ссылки по точному тексту.
- **partialLinkText**: Поиск ссылки по части текста.
- **tagName**: Поиск по имени тега.
- **name**: Поиск по атрибуту name.
- **xpath**: Поиск с использованием XPath (менее предпочтителен из-за сложности и скорости).
- **Пример на Java**:
  ```java
  import org.openqa.selenium.By;
  import org.openqa.selenium.WebDriver;
  import org.openqa.selenium.WebElement;
  import org.openqa.selenium.chrome.ChromeDriver;

  public class LocatorExample {
      public static void main(String[] args) {
          WebDriver driver = new ChromeDriver();
          driver.get("https://example.com");
          driver.findElement(By.id("login")).click(); // Поиск по ID
          driver.findElement(By.className("btn-primary")).click(); // Поиск по классу
          driver.findElement(By.cssSelector(".form input[type='text']")).sendKeys("test"); // CSS-селектор
          driver.findElement(By.linkText("Подробнее")).click(); // Поиск по тексту ссылки
          driver.findElement(By.partialLinkText("Подробнее о")).click(); // Поиск по части текста
          driver.quit();
      }
  }
 

### Поиск нескольких элементов
- **Описание**: Метод `findElements` возвращает список всех элементов, соответствующих локатору. Используется для проверки списков, таблиц или множественных элементов.
- **Применение в AQA**: Проверка наличия или количества элементов (например, строк в таблице).
- **Пример на Java**:
  ```java
  import org.openqa.selenium.By;
  import org.openqa.selenium.WebDriver;
  import org.openqa.selenium.WebElement;
  import org.openqa.selenium.chrome.ChromeDriver;
  import java.util.List;

  public class MultipleElementsExample {
      public static void main(String[] args) {
          WebDriver driver = new ChromeDriver();
          driver.get("https://example.com");
          List<WebElement> inputs = driver.findElements(By.tagName("input"));
          System.out.println("Найдено " + inputs.size() + " полей ввода");
          driver.quit();
      }
  }
  

**Связь с другими темами**: Локаторы — ключевой элемент UI-тестирования, связанный с клиент-серверной моделью, где UI зависит от данных API.

---

## Взаимодействие с веб-элементами

Selenium позволяет автоматизировать действия пользователя, такие как ввод текста, клики, работа с всплывающими окнами, cookies, фреймами и выполнение JavaScript.

### Всплывающие окна (Alerts)
- **Описание**: Управление браузерными алертами (подтверждение, отмена, получение текста).
- **Применение в AQA**: Тестирование диалогов подтверждения (например, при удалении записи).
- **Пример на Java**:
  ```java
  import org.openqa.selenium.Alert;
  import org.openqa.selenium.By;
  import org.openqa.selenium.WebDriver;
  import org.openqa.selenium.chrome.ChromeDriver;

  public class AlertExample {
      public static void main(String[] args) {
          WebDriver driver = new ChromeDriver();
          driver.get("https://example.com");
          driver.findElement(By.id("alertButton")).click();
          Alert alert = driver.switchTo().alert();
          System.out.println("Текст алерта: " + alert.getText());
          alert.accept(); // Подтвердить
          driver.quit();
      }
  }
 
### Cookies
- **Описание**: Управление cookies для авторизации, сессий или персонализации.
- **Применение в AQA**: Тестирование авторизации через cookies или очистка сессий.
- **Пример на Java**:
  ```java
  import org.openqa.selenium.Cookie;
  import org.openqa.selenium.WebDriver;
  import org.openqa.selenium.chrome.ChromeDriver;

  public class CookieExample {
      public static void main(String[] args) {
          WebDriver driver = new ChromeDriver();
          driver.get("https://example.com");
          driver.manage().addCookie(new Cookie("token", "abc123"));
          System.out.println("Cookies: " + driver.manage().getCookies());
          driver.manage().deleteAllCookies();
          driver.quit();
      }
  }
 
### Фреймы (Frames)
- **Описание**: Переключение контекста на iframe для взаимодействия с элементами внутри.
- **Применение в AQA**: Тестирование страниц с встроенными фреймами (например, редакторы или виджеты).
- **Пример на Java**:
  ```java
  import org.openqa.selenium.By;
  import org.openqa.selenium.WebDriver;
  import org.openqa.selenium.chrome.ChromeDriver;

  public class FrameExample {
      public static void main(String[] args) {
          WebDriver driver = new ChromeDriver();
          driver.get("https://example.com");
          driver.switchTo().frame("frameName");
          driver.findElement(By.id("frameButton")).click();
          driver.switchTo().defaultContent();
          driver.quit();
      }
  }
  
### Сложные действия (Actions)
- **Описание**: Выполнение сложных пользовательских действий, таких как drag-and-drop, наведение, двойной клик.
- **Применение в AQA**: Тестирование интерактивных элементов (например, перетаскивание в редакторах).
- **Пример на Java**:
  ```java
  import org.openqa.selenium.By;
  import org.openqa.selenium.WebDriver;
  import org.openqa.selenium.WebElement;
  import org.openqa.selenium.chrome.ChromeDriver;
  import org.openqa.selenium.interactions.Actions;

  public class ActionsExample {
      public static void main(String[] args) {
          WebDriver driver = new ChromeDriver();
          driver.get("https://example.com");
          WebElement element = driver.findElement(By.id("draggable"));
          Actions actions = new Actions(driver);
          actions.dragAndDropBy(element, 100, 100).build().perform();
          driver.quit();
      }
  }
 
### JavascriptExecutor
- **Описание**: Выполнение JavaScript для управления страницей, например, скроллинг или изменение элементов.
- **Применение в AQA**: Обход ограничений UI, тестирование скрытых элементов.
- **Пример на Java**:
  ```java
  import org.openqa.selenium.JavascriptExecutor;
  import org.openqa.selenium.WebDriver;
  import org.openqa.selenium.WebElement;
  import org.openqa.selenium.chrome.ChromeDriver;
  import org.openqa.selenium.By;

  public class JavascriptExecutorExample {
      public static void main(String[] args) {
          WebDriver driver = new ChromeDriver();
          driver.get("https://example.com");
          WebElement element = driver.findElement(By.id("hidden"));
          JavascriptExecutor js = (JavascriptExecutor) driver;
          js.executeScript("arguments[0].scrollIntoView(true);", element);
          driver.quit();
      }
  }
  
**Связь с другими темами**: Взаимодействие с элементами (например, cookies) связано с API-тестированием для проверки сессий.

---

## Лучшие практики и рекомендации

- **Используйте явные ожидания**: Предпочитайте `WebDriverWait` над `implicitlyWait` для стабильности.
  ```java
  WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
  wait.until(ExpectedConditions.elementToBeClickable(By.id("submit"))).click();
  ```
- **Оптимизируйте локаторы**: Используйте `id` и `cssSelector` вместо `xpath` для скорости и читаемости.
  ```java
  driver.findElement(By.cssSelector("#form input[type='submit']")).click();
  ```
- **Обрабатывайте исключения**: Используйте `try-catch` для управления ошибками.
  ```java
  try {
      driver.findElement(By.id("nonexistent")).click();
  } catch (NoSuchElementException e) {
      System.out.println("Элемент не найден: " + e.getMessage());
  }
  ```
- **Интеграция с CI/CD**: Настройте тесты в GitHub Actions.
  ```yaml
  name: UI Tests
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
        - name: Run Selenium tests
          run: mvn test
  ```
- **Документируйте тесты**: Добавляйте комментарии для читаемости.
  ```java
  // Проверка кликабельности кнопки логина
  WebElement button = wait.until(ExpectedConditions.elementToBeClickable(By.id("submit")));
  button.click();
  ```

---

## Связь с другими темами
- **Клиент-серверная модель**: UI-тесты с Selenium проверяют данные, полученные от API (например, валидация текста после ответа API).
  ```java
  Response apiResponse = given().get("https://api.example.com/users/1");
  String name = apiResponse.jsonPath().getString("name");
  assertEquals(name, driver.findElement(By.id("username")).getText());
  ```
- **Локаторы**: Используются для поиска элементов в UI-тестах, обеспечивая точное взаимодействие.
- **API-тестирование**: Интеграция UI- и API-тестов для проверки целостности данных.

---

## Источники
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/webdriver/)
- [ExpectedConditions API](https://www.selenium.dev/selenium/docs/api/java/org/openqa/selenium/support/ui/ExpectedConditions.html)
- [W3Schools CSS Selectors](https://www.w3schools.com/cssref/css_selectors.asp)
- [JavaScriptExecutor in Selenium](https://www.guru99.com/execute-javascript-selenium-webdriver.html)
- [Cypress Documentation](https://docs.cypress.io/)

---
[**← Назад к оглавлению**](../README.md)