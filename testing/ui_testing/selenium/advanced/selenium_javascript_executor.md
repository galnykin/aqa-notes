# JavaScriptExecutor в Selenium: расширенные возможности взаимодействия с веб-страницей

В UI-автоматизации часто встречаются ситуации, когда стандартных возможностей Selenium WebDriver недостаточно. Это бывает при работе с динамическими интерфейсами, кастомными компонентами (React, Angular, Vue), скрытыми или перекрытыми элементами. Для таких случаев используется `JavaScriptExecutor` — интерфейс, который позволяет выполнять JavaScript-код напрямую в браузере.

---

## 🧠 Что такое JavaScriptExecutor

`JavaScriptExecutor` реализуется большинством драйверов (`ChromeDriver`, `FirefoxDriver`) и предоставляет методы:

* `executeScript(String script, Object... args)` — выполняет JavaScript синхронно
* `executeAsyncScript(String script, Object... args)` — выполняет асинхронный JavaScript

Пример:

```java
JavascriptExecutor js = (JavascriptExecutor) driver;
js.executeScript("alert('Hello from Selenium!');");
```

---

## 🔍 Когда использовать JavaScriptExecutor

### 1. Элемент за пределами видимой области

```java
js.executeScript("arguments[0].scrollIntoView(true);", element);
```

### 2. Элемент перекрыт другим

```java
js.executeScript("arguments[0].click();", element);
```

### 3. Динамические элементы (React, Angular)

JavaScript напрямую обращается к DOM, минуя проблемы с ожиданиями.

### 4. Работа с input-элементами

```java
js.executeScript("document.getElementById('username').value='admin';");
String value = (String) js.executeScript("return document.getElementById('username').value;");
```

### 5. Генерация событий

```java
js.executeScript("arguments[0].dispatchEvent(new Event('change'));", element);
```

---

## 🧪 Расширенные примеры использования

### Прокрутка страницы

```java
js.executeScript("window.scrollBy(0, 500);");
```

### Получение innerText или innerHTML

```java
String text = (String) js.executeScript("return arguments[0].innerText;", element);
String html = (String) js.executeScript("return arguments[0].innerHTML;", element);
```

### Проверка наличия элемента

```java
Boolean exists = (Boolean) js.executeScript("return document.querySelector('#submit') !== null;");
```

### Удаление или скрытие элемента

```java
js.executeScript("document.querySelector('#popup').remove();");
js.executeScript("arguments[0].style.display='none';", element);
```

### Визуальное выделение элемента (debug)

```java
js.executeScript("arguments[0].style.border='3px solid red';", element);
```

### Получение размеров окна или позиции элемента

```java
Long width = (Long) js.executeScript("return window.innerWidth;");
Long height = (Long) js.executeScript("return window.innerHeight;");
```

---

## 🧱 Интеграция с Page Object

Для удобства можно вынести методы в утилитарный класс:

```java
public class JsHelper {
    private WebDriver driver;

    public JsHelper(WebDriver driver) {
        this.driver = driver;
    }

    public void clickElement(By locator) {
        WebElement element = driver.findElement(locator);
        ((JavascriptExecutor) driver).executeScript("arguments[0].click();", element);
    }

    public void scrollToElement(WebElement element) {
        ((JavascriptExecutor) driver).executeScript("arguments[0].scrollIntoView(true);", element);
    }
}
```

Использование в Page Object:

```java
JsHelper jsHelper = new JsHelper(driver);
jsHelper.clickElement(By.id("submit"));
```

---

## ⚡ Асинхронные скрипты

Метод `executeAsyncScript()` используется для работы с коллбэками:

```java
Long duration = (Long) js.executeAsyncScript(
    "var callback = arguments[arguments.length - 1];" +
    "setTimeout(function(){ callback(123); }, 2000);"
);
System.out.println("Result: " + duration);
```

---

## ⚠️ Предостережения

* Использовать только при необходимости — стандартные методы WebDriver более надёжны
* Ошибки JS не всегда бросают исключение в Java-коде
* Возможна нестабильность тестов при частом использовании
* Прямое вмешательство в DOM может отличаться от реального поведения пользователя

---

## 🔗 Источники

* [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/webdriver/)
* [JavaScriptExecutor API](https://www.selenium.dev/selenium/docs/api/java/org/openqa/selenium/JavascriptExecutor.html)
* [MDN Web Docs — JavaScript DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)
* [Guru99 — JavaScriptExecutor in Selenium](https://www.guru99.com/execute-javascript-selenium-webdriver.html)

---

[**← Назад к оглавлению**](../README.md)
