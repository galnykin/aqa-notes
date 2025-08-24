### Подробное руководство по аннотациям в Java в контексте AQA

Аннотации в Java — это метаданные, которые добавляются к коду для предоставления дополнительной информации компилятору, инструментам или фреймворкам. В автоматизации тестирования (AQA) аннотации активно используются в фреймворках, таких как JUnit, TestNG и Selenium, для управления тестами, настройки окружения и обработки данных. Это руководство подробно рассматривает аннотации в Java, их использование в AQA, типы аннотаций, их создание и применение в тестах.

Материал также связывается с другими темами, такими как тестирование, клиент-серверная модель и Page Object Model (POM), чтобы показать применимость аннотаций в реальных задачах AQA.

---

## 1. Что такое аннотации в Java?

- **Описание**: Аннотации — это специальные метки в коде, начинающиеся с символа `@`, которые предоставляют метаданные о классах, методах, полях или других элементах программы. Они не влияют напрямую на логику программы, но используются компилятором, JVM или фреймворками для обработки кода.
- **Цель**:
    - Управление поведением тестов (например, `@Test` в JUnit).
    - Конфигурация окружения (например, `@BeforeEach` для подготовки).
    - Упрощение кода через автоматическую обработку (например, `@FindBy` в Selenium).
- **Применение в AQA**:
    - Аннотации JUnit/TestNG для структурирования тестов.
    - Аннотации Selenium (`@FindBy`) для поиска веб-элементов.
    - Аннотации для параметризации тестов.

**Пример**:
```java
import org.junit.jupiter.api.Test;

public class ExampleTest {
    @Test
    void testExample() {
        System.out.println("Тест выполнен");
    }
}
```

**Связь с другими темами**:
- **Тестирование**: Аннотации, такие как `@Test`, определяют тест-кейсы, аналогично структуре AAA (Arrange-Act-Assert).
- **Клиент-серверная модель**: Аннотации в Selenium упрощают взаимодействие с сервером (браузером) через локаторы.

---

## 2. Типы аннотаций в Java

### 2.1. Встроенные аннотации Java
Java предоставляет встроенные аннотации, определённые в пакете `java.lang`:
1. **@Override**:
    - Указывает, что метод переопределяет метод суперкласса или интерфейса.
    - Применение: Проверка корректности переопределения в классах POM.
    - Пример:
      ```java
      public class BasePage {
          public void navigate() {
              System.out.println("Переход на страницу");
          }
      }
 
      public class LoginPage extends BasePage {
          @Override
          public void navigate() {
              System.out.println("Переход на страницу логина");
          }
      }
      ```

2. **@Deprecated**:
    - Помечает устаревший код, который не рекомендуется использовать.
    - Применение: Отметка устаревших методов в тестовых фреймворках.
    - Пример:
      ```java
      @Deprecated
      public void oldLoginMethod() {
          System.out.println("Устаревший метод логина");
      }
      ```

3. **@SuppressWarnings**:
    - Подавляет предупреждения компилятора (например, "unused").
    - Применение: Используется в тестах для игнорирования предупреждений о неиспользуемых переменных.
    - Пример:
      ```java
      @SuppressWarnings("unused")
      private WebDriver driver;
      ```

### 2.2. Аннотации JUnit 5
JUnit 5 — популярный фреймворк для тестирования, широко используемый в AQA. Основные аннотации:
1. **@Test**:
    - Помечает метод как тестовый.
    - Пример:
      ```java
      import org.junit.jupiter.api.Test;
      import static org.junit.jupiter.api.Assertions.assertEquals;
 
      public class LoginTest {
          @Test
          void testLogin() {
              String actual = "Welcome";
              assertEquals("Welcome", actual, "Сообщение не соответствует");
          }
      }
      ```

2. **@BeforeEach**:
    - Выполняется перед каждым тестом (например, для настройки драйвера).
    - Пример:
      ```java
      @BeforeEach
      void setUp() {
          WebDriver driver = new ChromeDriver();
          driver.get("https://example.com");
      }
      ```

3. **@AfterEach**:
    - Выполняется после каждого теста (например, для закрытия драйвера).
    - Пример:
      ```java
      @AfterEach
      void tearDown() {
          driver.quit();
      }
      ```

4. **@BeforeAll** / **@AfterAll**:
    - Выполняется один раз перед/после всех тестов в классе (должны быть статическими).
    - Пример:
      ```java
      @BeforeAll
      static void initAll() {
          System.setProperty("webdriver.chrome.driver", "path/to/chromedriver");
      }
      ```

5. **@ParameterizedTest**:
    - Позволяет запускать тест с разными входными данными.
    - Пример:
      ```java
      import org.junit.jupiter.params.ParameterizedTest;
      import org.junit.jupiter.params.provider.ValueSource;
 
      public class ParameterizedLoginTest {
          @ParameterizedTest
          @ValueSource(strings = {"John", "Alice"})
          void testLoginWithUsers(String username) {
              WebDriver driver = new ChromeDriver();
              driver.get("https://example.com");
              driver.findElement(By.id("username")).sendKeys(username);
              driver.quit();
          }
      }
      ```

### 2.3. Аннотации Selenium
Selenium использует аннотации для упрощения работы с веб-элементами в Page Object Model:
1. **@FindBy**:
    - Определяет локатор для веб-элемента в POM.
    - Пример:
      ```java
      import org.openqa.selenium.WebDriver;
      import org.openqa.selenium.WebElement;
      import org.openqa.selenium.support.FindBy;
      import org.openqa.selenium.support.PageFactory;
 
      public class LoginPage {
          @FindBy(id = "username")
          private WebElement inputUsername;
 
          @FindBy(id = "submit")
          private WebElement submitButton;
 
          public LoginPage(WebDriver driver) {
              PageFactory.initElements(driver, this);
          }
 
          public void login(String username) {
              inputUsername.sendKeys(username);
              submitButton.click();
          }
      }
      ```

2. **@FindBys** / **@FindAll**:
    - **@FindBys**: Логическое И для поиска (все условия должны выполняться).
    - **@FindAll**: Логическое ИЛИ (хотя бы одно условие).
    - Пример:
      ```java
      @FindBys({
          @FindBy(className = "button"),
          @FindBy(tagName = "input")
      })
      private WebElement specificButton;
 
      @FindAll({
          @FindBy(id = "button1"),
          @FindBy(id = "button2")
      })
      private List<WebElement> buttons;
      ```

### 2.4. Пользовательские аннотации
Можно создавать собственные аннотации для специфических задач в AQA.
- **Создание аннотации**:
  ```java
  import java.lang.annotation.ElementType;
  import java.lang.annotation.Retention;
  import java.lang.annotation.RetentionPolicy;
  import java.lang.annotation.Target;

  @Retention(RetentionPolicy.RUNTIME)
  @Target(ElementType.METHOD)
  public @interface CustomTest {
      String description() default "Тест без описания";
  }
  ```
- **Использование**:
  ```java
  public class CustomTestExample {
      @CustomTest(description = "Проверка логина")
      public void testLogin() {
          System.out.println("Тест логина выполнен");
      }
  }
  ```
- **Обработка аннотации**:
    - Используйте рефлексию для чтения аннотаций:
      ```java
      import java.lang.reflect.Method;
  
      public class TestRunner {
          public static void main(String[] args) throws Exception {
              Method method = CustomTestExample.class.getMethod("testLogin");
              if (method.isAnnotationPresent(CustomTest.class)) {
                  CustomTest annotation = method.getAnnotation(CustomTest.class);
                  System.out.println("Описание теста: " + annotation.description());
              }
          }
      }
      ```

**Применение в AQA**:
- Пользовательские аннотации могут использоваться для логирования тестов или маркировки критических тест-кейсов.

**Связь с другими темами**:
- **ООП**: Аннотации `@FindBy` интегрируются в Page Object Model.
- **Тестирование**: Аннотации JUnit упрощают структуру AAA.

---

## 3. Использование аннотаций в AQA

### 3.1. Структура теста с JUnit
Пример полного теста с аннотациями:
```java
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;

import static org.junit.jupiter.api.Assertions.assertEquals;

public class LoginTest {
    private WebDriver driver;

    public class LoginPage {
        @FindBy(id = "username")
        private WebElement inputUsername;

        @FindBy(id = "submit")
        private WebElement submitButton;

        public LoginPage(WebDriver driver) {
            PageFactory.initElements(driver, this);
        }

        public void login(String username) {
            inputUsername.sendKeys(username);
            submitButton.click();
        }

        public String getMessage() {
            return driver.findElement(By.id("message")).getText();
        }
    }

    @BeforeEach
    void setUp() {
        driver = new ChromeDriver();
        driver.get("https://example.com");
    }

    @Test
    void testLoginSuccess() {
        LoginPage page = new LoginPage(driver);
        page.login("John");
        assertEquals("Welcome, John", page.getMessage(), "Сообщение не соответствует");
    }

    @AfterEach
    void tearDown() {
        driver.quit();
    }
}
```

### 3.2. Параметризованные тесты
Использование `@ParameterizedTest` для проверки разных пользователей:
```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.CsvSource;

public class ParameterizedLoginTest {
    private WebDriver driver;

    @BeforeEach
    void setUp() {
        driver = new ChromeDriver();
        driver.get("https://example.com");
    }

    @ParameterizedTest
    @CsvSource({
        "John, Welcome, John",
        "Alice, Welcome, Alice"
    })
    void testLoginWithMultipleUsers(String username, String expectedMessage) {
        LoginPage page = new LoginPage(driver);
        page.login(username);
        assertEquals(expectedMessage, page.getMessage());
    }

    @AfterEach
    void tearDown() {
        driver.quit();
    }
}
```

### 3.3. Пользовательская аннотация в AQA
Создание аннотации для логирования тестов:
```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@interface LogTest {
    String priority() default "Medium";
}

public class LoggedTest {
    @LogTest(priority = "High")
    @Test
    void testCriticalLogin() {
        System.out.println("Критический тест логина");
    }

    public static void main(String[] args) throws Exception {
        Method method = LoggedTest.class.getMethod("testCriticalLogin");
        if (method.isAnnotationPresent(LogTest.class)) {
            LogTest log = method.getAnnotation(LogTest.class);
            System.out.println("Приоритет теста: " + log.priority());
        }
    }
}
```

---

## 4. Связь с другими темами

- **Тестирование**:
    - Аннотации JUnit (`@Test`, `@BeforeEach`) структурируют тесты по AAA.
    - Пример: `@Test` для проверки логина соответствует assert в тест-кейсе.

- **Клиент-серверная модель**:
    - Аннотации `@FindBy` упрощают взаимодействие с сервером (браузером) через локаторы.
    - Пример: `@FindBy(id = "username")` заменяет `By.id("username")`.

- **ООП**:
    - Аннотации интегрируются в Page Object Model для инкапсуляции локаторов.
    - Пример: `@FindBy` в классе `LoginPage`.

- **SQL**:
    - Аннотации могут использоваться для маркировки тестов, проверяющих данные в БД:
      ```java
      @Test
      void testDatabaseUser() {
          String query = "SELECT * FROM users WHERE username = 'John'";
          // Логика SQL-запроса
      }
      ```

---

## 5. Преимущества и недостатки аннотаций

### 5.1. Преимущества
- Упрощают код: `@FindBy` заменяет длинные цепочки `findElement`.
- Управляют тестами: `@BeforeEach` автоматизирует настройку.
- Гибкость: Параметризация через `@ParameterizedTest`.
- Расширяемость: Пользовательские аннотации для специфических задач.

### 5.2. Недостатки
- Сложность: Пользовательские аннотации требуют рефлексии.
- Зависимость от фреймворков: `@FindBy` работает только с `PageFactory`.
- Ограниченная читаемость: Много аннотаций могут усложнить код.

---

## 6. Рекомендации

- **Используйте стандартные аннотации**:
    - `@Test`, `@BeforeEach`, `@AfterEach` для базовых тестов.
    - `@FindBy` для POM в Selenium.
- **Параметризуйте тесты**:
    - Используйте `@ParameterizedTest` для проверки разных данных:
      ```java
      @ParameterizedTest
      @ValueSource(strings = {"John", "Alice"})
      void testLogin(String username) {
          driver.findElement(By.id("username")).sendKeys(username);
      }
      ```
- **Создавайте пользовательские аннотации**:
    - Для логирования или маркировки тестов:
      ```java
      @Retention(RetentionPolicy.RUNTIME)
      @Target(ElementType.METHOD)
      @interface CriticalTest {}
      ```
- **Интеграция с CI/CD**:
    - Настройте JUnit в Maven для запуска тестов:
      ```xml
      <dependency>
          <groupId>org.junit.jupiter</groupId>
          <artifactId>junit-jupiter-engine</artifactId>
          <version>5.11.0</version>
          <scope>test</scope>
      </dependency>
      ```
    - Запуск в GitHub Actions:
      ```yaml
      name: Run Tests
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
            - name: Run tests
              run: mvn test
      ```

- **Избегайте переусложнения**:
    - Не используйте слишком много аннотаций в одном классе.
- **Проверяйте локаторы**:
    - Используйте `@FindBy` для стабильных локаторов:
      ```java
      @FindBy(css = "[data-testid='submit']")
      private WebElement submitButton;
      ```

---

## 7. Источники
- [Java Annotations Documentation](https://docs.oracle.com/javase/tutorial/java/annotations/)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [Selenium Documentation](https://www.selenium.dev/documentation/)
- [Baeldung: Java Annotations](https://www.baeldung.com/java-annotations)

---
