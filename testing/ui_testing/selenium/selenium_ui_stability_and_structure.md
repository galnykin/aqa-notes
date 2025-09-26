# Практика и устойчивость UI-тестов: Selenium, ожидания, взаимодействие

Автоматизация пользовательского интерфейса (UI) — важный этап обеспечения качества веб-приложений. Она позволяет имитировать действия пользователя, проверять поведение элементов и выявлять ошибки, которые невозможно отследить на уровне бизнес-логики. В этой статье собраны ключевые аспекты, связанные с реализацией устойчивых UI-тестов: архитектура Selenium, стратегии поиска, ожидания, нестабильность элементов, шаблоны проектирования и отладка.

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

## 🧾 Именование переменных

Имена переменных должны отражать назначение и тип:

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

## 🐞 Отладка и распространённые ошибки

### IntelliJ Debug:
- позволяет пошагово анализировать выполнение теста;
- используется для проверки XPath, состояния элементов.

### Частые ошибки:
- `NoSuchElementException` — элемент не найден;
- `TimeoutException` — ожидание не выполнено;
- `StaleElementReferenceException` — элемент устарел.

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

Добавляются в `pom.xml` для подключения JUnit 5.

---

## 🔗 Источники:
- [Selenium WebDriver](https://www.selenium.dev/)
- [Selenium Waits](https://www.selenium.dev/documentation/webdriver/waits/)
- [JUnit Jupiter Engine](https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-engine)
- [XPath Syntax](https://www.w3schools.com/xml/xpath_syntax.asp)

---
[**← Назад к оглавлению**](../../../README.md)