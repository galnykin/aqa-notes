# Архитектура UI-тестов: библиотеки, принципы, структура

Эффективная автоматизация UI-тестирования требует не только знания инструментов, но и грамотной архитектуры проекта. В этом материале структурированы ключевые компоненты: используемые библиотеки, принципы проектирования, шаблоны тестов и организация кода.

---

## 📦 Внешние библиотеки

Для построения UI-тестов на Java используются две основные библиотеки:

### JUnit 5
Фреймворк для написания и запуска модульных тестов:
- Аннотации: `@Test`, `@BeforeEach`, `@AfterEach`, `@DisplayName`, `@Disabled`
- Поддержка параметризации: `@ParameterizedTest`, `@CsvSource`
- Гибкая архитектура: `TestMethodOrder`, `@Order`

### Selenium WebDriver
Инструмент для автоматизации браузера:
- Управление браузером: `ChromeDriver`, `FirefoxDriver`, `EdgeDriver`
- Поиск элементов: `findElement(By...)`
- Взаимодействие: `click()`, `sendKeys()`, `getText()`
- Ожидания: `WebDriverWait`, `ExpectedConditions`

Обе библиотеки интегрируются через Maven:
```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.9.3</version>
</dependency>
<dependency>
    <groupId>org.seleniumhq.selenium</groupId>
    <artifactId>selenium-java</artifactId>
    <version>4.16.1</version>
</dependency>
```

---

## 🧠 Принцип DRY и модель POM

### DRY (Don't Repeat Yourself)
Принцип, согласно которому повторяющийся код должен быть вынесен в отдельные методы, классы или компоненты. В тестировании это означает:

- повторное использование локаторов;
- единые методы взаимодействия;
- общие шаги вынесены в базовые классы.

### POM (Page Object Model)
Архитектурный шаблон, при котором каждый экран или компонент описывается отдельным классом.

#### Структура Page Object:
- **Описание элементов**:
  ```java
  private final By inputLogin = By.id("username");
  private final By buttonSubmit = By.id("submit");
  ```

- **Методы взаимодействия**:
  ```java
  public void login(String user, String pass) {
      driver.findElement(inputLogin).sendKeys(user);
      driver.findElement(buttonSubmit).click();
  }
  ```

Преимущества:
- изоляция логики страницы;
- упрощение поддержки;
- повторное использование.

---

## 🧪 Структура тестов

### Аннотация `@Test`
Используется для обозначения тестового метода. В UI-тестах желательно избегать неатомарных тестов (несколько шагов и проверок в одном методе).

Пример неатомарного теста:
```java
@Test
void testLoginAndSearch() {
    loginPage.login("admin", "1234");
    assertTrue(homePage.isLoggedIn());

    searchPage.search("pizza");
    assertTrue(searchPage.hasResults());
}
```

Лучше разделить на два отдельных теста:
```java
@Test
void testLogin() { ... }

@Test
void testSearch() { ... }
```

---

## 🔄 Жизненный цикл тестов

Для подготовки и очистки окружения используются аннотации:

### `@BeforeEach`
Вызывается перед каждым тестом. Предпочтительно использовать контекстные названия:
```java
@BeforeEach
void openBrowserAndCloseCookie() {
    driver = new ChromeDriver();
    driver.get("https://example.com");
    closeCookieBanner();
}
```

### `@AfterEach`
Вызывается после каждого теста:
```java
@AfterEach
void tearDown() {
    driver.quit();
}
```

---

## 🧱 Организация тестов

Тесты группируются по функциональности:

- `LoginTest` — проверка авторизации
- `CartTest` — работа с корзиной
- `SearchTest` — поиск товаров

Каждый тест-класс наследуется от `BaseTest`, где описаны общие действия:

```java
public class BaseTest {
    protected WebDriver driver;

    @BeforeEach
    void openBrowserAndCloseCookie() { ... }

    @AfterEach
    void tearDown() { ... }
}
```

Пример наследования:
```java
public class LoginTest extends BaseTest {
    @Test
    void testSuccessfulLogin() { ... }
}
```

---

---
[**← Назад к оглавлению**](../../../README.md)