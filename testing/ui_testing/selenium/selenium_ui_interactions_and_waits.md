# Автоматизация тестирования пользовательского интерфейса (UI) с Selenium

Автоматизация UI-тестирования позволяет проверять поведение веб-приложений через программное взаимодействие с браузером. Selenium WebDriver — один из самых популярных инструментов для этой задачи. Он обеспечивает гибкость, кросс-браузерность и богатый набор возможностей для работы с веб-элементами, ожиданиями и пользовательскими действиями.

---

## ⏳ Selenium и ожидания

Ожидания — это механизм, позволяющий тесту дождаться появления или изменения состояния элемента на странице. Без них тесты могут быть нестабильными из-за асинхронной загрузки контента.

### Виды ожиданий:

#### Implicit Wait (неявное ожидание)
Устанавливается один раз и применяется ко всем `findElement` вызовам.

```java
driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
```

Недостатки:
- Применяется ко всем элементам.
- Может замедлить тесты при ошибках.

#### Explicit Wait (явное ожидание)
Ожидание конкретного условия для конкретного элемента.

```java
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
WebElement element = wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("login")));
```

Популярные `ExpectedConditions`:
- `visibilityOfElementLocated`
- `elementToBeClickable`
- `presenceOfElementLocated`
- `textToBePresentInElement`

#### FluentWait
Позволяет задать интервал опроса и игнорируемые исключения.

```java
Wait<WebDriver> wait = new FluentWait<>(driver)
    .withTimeout(Duration.ofSeconds(15))
    .pollingEvery(Duration.ofMillis(500))
    .ignoring(NoSuchElementException.class);

WebElement element = wait.until(driver -> driver.findElement(By.id("submit")));
```

FluentWait полезен при нестабильных или динамических интерфейсах.

---

## 🔍 Стратегии поиска веб-элементов

Selenium предоставляет гибкие способы поиска элементов на странице через класс `By`.

### Основные стратегии:

```java
driver.findElement(By.className("btn-primary"));
driver.findElement(By.cssSelector(".form input[type='text']"));
driver.findElement(By.linkText("Подробнее"));
driver.findElement(By.partialLinkText("Подробнее о"));
```

- `className` — по имени класса.
- `cssSelector` — мощный и быстрый способ, особенно для вложенных элементов.
- `linkText` — по точному тексту ссылки.
- `partialLinkText` — по части текста ссылки.

### Поиск нескольких элементов:
```java
List<WebElement> inputs = driver.findElements(By.tagName("input"));
```

Использование `findElements` возвращает список, даже если он пуст, что удобно для проверок.

---

## 🧩 Взаимодействие с веб-элементами

Selenium позволяет работать с различными типами элементов, включая всплывающие окна, cookies, фреймы и JavaScript.

### Alerts (всплывающие окна)
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

---

## 🔗 Источники:
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/webdriver/)
- [ExpectedConditions API](https://www.selenium.dev/selenium/docs/api/java/org/openqa/selenium/support/ui/ExpectedConditions.html)
- [W3Schools CSS Selectors](https://www.w3schools.com/cssref/css_selectors.asp)
- [JavaScriptExecutor in Selenium](https://www.guru99.com/execute-javascript-selenium-webdriver.html)

---
[**← Назад к оглавлению**](../../../README.md)