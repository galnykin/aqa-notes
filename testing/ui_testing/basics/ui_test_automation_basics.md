# Основы автоматизации тестирования пользовательского интерфейса (UI)

Автоматизация тестирования пользовательского интерфейса позволяет проверять поведение веб-приложений без участия человека. Она включает взаимодействие с элементами страницы, проверку их состояния, заполнение форм и анализ результатов. В основе лежит фреймворк Selenium WebDriver, а также архитектурные и проектные подходы, обеспечивающие надёжность и масштабируемость тестов.

---

## 🧰 Фреймворк Selenium WebDriver

**Selenium WebDriver** — это инструмент для автоматизации браузера. Он позволяет управлять браузером программно: открывать страницы, взаимодействовать с элементами, проверять результаты.

### Общие сведения:
- Поддерживает браузеры: Chrome, Firefox, Edge, Safari.
- Работает с языками: Java, Python, C#, JavaScript.
- Используется в UI-тестах, smoke-тестах, регрессионных проверках.

### Архитектура:
- **WebDriver API** — интерфейс для взаимодействия с браузером.
- **Browser Driver** — компонент, связывающий WebDriver с конкретным браузером (например, ChromeDriver).
- **Selenium Server/Grid** — для распределённого запуска тестов.

### Компоненты:
- `WebDriver` — основной интерфейс.
- `WebElement` — представление элемента на странице.
- `By` — стратегии поиска.
- `Actions`, `Select`, `Alert` — дополнительные классы для сложных взаимодействий.

---

## ⚙️ Инициализация и настройка WebDriver

Для запуска тестов необходимо создать экземпляр драйвера и открыть нужную страницу.

Пример:
```java
WebDriver driver = new ChromeDriver();
driver.get("https://example.com");
```

### Настройка:
- Использование `WebDriverManager` для автоматической загрузки драйверов:
```java
WebDriverManager.chromedriver().setup();
WebDriver driver = new ChromeDriver();
```

- Настройка ожиданий:
```java
driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
```

- Закрытие браузера:
```java
driver.quit();
```

---

## 🔍 Стратегии поиска веб-элементов (`By`)

Поиск элементов осуществляется с помощью класса `By`. Наиболее надёжные селекторы — по `id`, `name`, затем `class`, `XPath`.

### Основные стратегии:
```java
driver.findElement(By.id("username"));
driver.findElement(By.name("password"));
driver.findElement(By.className("btn-primary"));
driver.findElement(By.tagName("input"));
driver.findElement(By.xpath("//input[@name='height']"));
```

### XPath:
- Абсолютный путь: `/html/body/p`
- Относительный путь: `//body/p`
- По атрибуту: `//input[@name='height']`
- По индексу: `//table/tbody/tr[2]/td[2]/input`

XPath — мощный язык запросов, позволяющий обращаться к узлам DOM (Document Object Model). Он поддерживает вертикальные и горизонтальные оси, фильтрацию, индексацию.

---

## 📄 Шаблон проектирования PageObject

**PageObject** — это архитектурный шаблон, при котором каждый веб-страница или компонент описывается отдельным классом.

Пример:
```java
public class LoginPage {
    private WebDriver driver;

    private By usernameField = By.id("username");
    private By passwordField = By.id("password");
    private By loginButton = By.tagName("button");

    public LoginPage(WebDriver driver) {
        this.driver = driver;
    }

    public void login(String user, String pass) {
        driver.findElement(usernameField).sendKeys(user);
        driver.findElement(passwordField).sendKeys(pass);
        driver.findElement(loginButton).click();
    }
}
```

Преимущества:
- Повторное использование кода
- Упрощение поддержки
- Отделение логики теста от структуры страницы

---

## 🧪 Шаблон AAA (Arrange-Act-Assert)

Структура теста должна быть понятной и предсказуемой. Шаблон **AAA** — один из самых распространённых:

1. **Arrange** — подготовка данных и окружения.
2. **Act** — выполнение действия.
3. **Assert** — проверка результата.

Пример:
```java
@Test
void shouldLoginSuccessfully() {
    // Arrange
    LoginPage loginPage = new LoginPage(driver);

    // Act
    loginPage.login("admin", "1234");

    // Assert
    assertTrue(driver.findElement(By.id("welcome")).isDisplayed());
}
```

Также используется шаблон **given-when-then**, особенно в BDD.

---

## 🧠 Техники тест-дизайна для UI

Тест-дизайн — это методика выбора и организации тестов для максимального покрытия.

### Основные техники:
- **Эквивалентное разбиение** — группировка входных данных по типам.
- **Анализ граничных значений** — проверка крайних допустимых значений.
- **Попарное тестирование** — минимизация количества комбинаций.
- **Тестирование состояния UI** — проверка видимости, доступности, активности элементов.
- **Тестирование форм** — проверка обязательных полей, валидации, сообщений об ошибках.

Пример HTML-формы:
```html
<form>
  <p><input type="text" name="height" size="5"/></p>
  <p><input type="radio" name="gender" value="male"/></p>
  <p><input type="checkbox" name="subscribe"/></p>
  <button type="submit">Отправить</button>
</form>
```

Тесты должны проверять корректность ввода, реакцию на пустые поля, отображение сообщений.

---

## 🔗 Источники:
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/webdriver/)
- [W3C Standards](https://www.w3.org/)
- [W3Schools HTML Reference](https://www.w3schools.com/html/)
- [XPath Syntax](https://www.w3schools.com/xml/xpath_syntax.asp)
- [Тестовая страница для практики](https://svyatoslav.biz/testlab/wt/index.php)

---
[**← Назад к оглавлению**](../README.md)