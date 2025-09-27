# Модульное тестирование в Java

Модульное тестирование — это фундаментальный уровень проверки программного обеспечения, при котором тестируются отдельные компоненты программы изолированно от остальных. В Java модульное тестирование является базой для автоматизации и активно используется в CI/CD процессах для обеспечения стабильности и предсказуемости системы.

---

## 🧪 Что такое модульное тестирование (`unit testing`)

Модульный тест — это автоматизированная проверка одного метода или класса. Он должен быть:

* **Изолированным** — без внешних зависимостей (БД, сеть, UI).
* **Быстрым** — выполняться за миллисекунды.
* **Повторяемым** — одинаковый результат при каждом запуске.

### Пример на JUnit 5

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;

class CalculatorTest {

    @Test
    void testAddition() {
        Calculator calc = new Calculator();
        int result = calc.add(2, 3);
        assertEquals(5, result);
    }
}
```

### Пример на TestNG

```java
import org.testng.annotations.Test;
import static org.testng.Assert.assertEquals;

public class CalculatorTest {

    @Test
    public void testAddition() {
        Calculator calc = new Calculator();
        int result = calc.add(2, 3);
        assertEquals(result, 5);
    }
}
```

---

## 🧱 Пирамида тестирования

```
        UI Tests
     Integration Tests
   Unit Tests (основание)
```

### Уровни:

* **Unit Tests** — основа пирамиды, быстрые, дешёвые в поддержке.
* **Integration Tests** — проверяют связку модулей (например, сервис + БД).
* **UI Tests** — тестируют поведение интерфейса через браузер.

⚡ В современном Java-стеке:

* Unit — JUnit 5 / TestNG + AssertJ
* Integration — Rest Assured, Testcontainers
* UI — Selenium WebDriver + WebDriverManager

---

## 🧭 Виды тестирования

* **Unit** — тестируем классы/методы (Calculator, Validator).
* **Integration** — проверяем взаимодействие (API + DB).
* **System** — тест всей системы (например, end-to-end через UI).
* **Regression** — проверяем, что баги не вернулись.
* **Smoke** — базовая проверка критического функционала.

💡 Автоматизаторы чаще всего пишут **unit + integration + UI**.

---

## 🧠 Разработка через тестирование (TDD)

**TDD** — цикл "Red → Green → Refactor".

### Пример:

```java
@Test
void shouldReturnHello() {
    Greeter g = new Greeter();
    assertEquals("Hello", g.sayHello());
}
```

Пишем тест → создаём `Greeter.sayHello()` → тест проходит.

⚡ В TestNG то же самое:

```java
@Test
public void shouldReturnHello() {
    Greeter g = new Greeter();
    assertEquals(g.sayHello(), "Hello");
}
```

TDD дисциплинирует и помогает писать чистый код.

---

## 🧰 Библиотеки и фреймворки для модульного тестирования на Java

### JUnit 5 (Jupiter)

* `@Test`, `@BeforeEach`, `@AfterEach`
* `@DisplayName`, `@ParameterizedTest`, `@Tag`
* Расширяемость через `Extension API`

```java
@DisplayName("Проверка деления")
@Test
void testDivision() {
    assertEquals(2, calculator.divide(10, 5));
}
```

### TestNG

* `@Test`, `@BeforeMethod`, `@AfterMethod`
* Группировка тестов: `@Test(groups = {"smoke"})`
* Параметризация через `@DataProvider`

```java
@DataProvider(name = "numbers")
public Object[][] data() {
    return new Object[][]{{10, 5, 2}, {20, 4, 5}};
}

@Test(dataProvider = "numbers")
public void testDivision(int a, int b, int expected) {
    Calculator calc = new Calculator();
    assertEquals(calc.divide(a, b), expected);
}
```

---

## 🔌 Интеграционное тестирование (API, DB)

### Rest Assured (тест API)

```java
import static io.restassured.RestAssured.*;
import static org.hamcrest.Matchers.*;

@Test
void testGetUsers() {
    given()
        .baseUri("https://reqres.in")
    .when()
        .get("/api/users?page=2")
    .then()
        .statusCode(200)
        .body("data.size()", greaterThan(0));
}
```

### Testcontainers (тестирование с реальной БД в Docker)

```java
@Container
static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15");

@Test
void testWithDb() throws SQLException {
    try (Connection conn = DriverManager.getConnection(
            postgres.getJdbcUrl(), postgres.getUsername(), postgres.getPassword())) {
        assertNotNull(conn);
    }
}
```

---

## 🌐 UI-тестирование (Selenium + WebDriverManager)

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.*;

public class GoogleTest {
    WebDriver driver;

    @BeforeClass
    public void setup() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
    }

    @Test
    public void openGoogle() {
        driver.get("https://google.com");
        assertEquals(driver.getTitle(), "Google");
    }

    @AfterClass
    public void tearDown() {
        driver.quit();
    }
}
```

---

## 📊 AssertJ — более читаемые проверки

```java
import static org.assertj.core.api.Assertions.*;

@Test
void testList() {
    var list = List.of("Java", "JUnit", "Selenium");
    assertThat(list)
        .hasSize(3)
        .contains("Java")
        .allMatch(s -> s.length() > 2);
}
```

---

## 📈 Allure — красивые отчёты

```java
@Epic("UI Tests")
@Feature("Login")
@Story("User can log in with valid credentials")
@Test
void loginTest() {
    step("Открыть страницу логина", () -> driver.get("https://example.com/login"));
    step("Ввести данные", () -> { /* actions */ });
    step("Проверить результат", () -> assertEquals("Dashboard", driver.getTitle()));
}
```

Запуск с Maven:

```xml
<plugin>
  <groupId>io.qameta.allure</groupId>
  <artifactId>allure-maven</artifactId>
  <version>2.11.2</version>
</plugin>
```

---

## 🔗 Источники

* [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
* [TestNG Documentation](https://testng.org/doc/)
* [Rest Assured Docs](https://rest-assured.io/)
* [Selenium WebDriver Docs](https://www.selenium.dev/documentation/)
* [WebDriverManager (Bonigarcia)](https://github.com/bonigarcia/webdrivermanager)
* [Testcontainers](https://www.testcontainers.org/)
* [AssertJ Docs](https://assertj.github.io/doc/)
* [Allure Framework](https://docs.qameta.io/allure/)
* [Martin Fowler — TDD](https://martinfowler.com/bliki/TestDrivenDevelopment.html)
* [Testing Pyramid Explained](https://www.thoughtworks.com/insights/blog/test-pyramid)

---

[**← Назад к оглавлению**](../README.md)

---
