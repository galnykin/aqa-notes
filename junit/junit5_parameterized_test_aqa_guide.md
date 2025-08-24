### Подробное руководство по аннотации `@ParameterizedTest` в JUnit 5 для проверки разных данных в контексте AQA

Аннотация `@ParameterizedTest` из JUnit 5 позволяет запускать один и тот же тестовый метод с разными наборами входных данных, что особенно полезно в автоматизации тестирования (AQA) для проверки поведения приложения на различных сценариях. Это руководство подробно рассматривает использование `@ParameterizedTest`, источники данных (такие как `@ValueSource`, `@CsvSource`, `@MethodSource` и другие), примеры применения в тестах с Selenium, преимущества, недостатки и рекомендации. Поскольку контекст запроса указывает на Java, примеры кода предоставлены только на этом языке, в соответствии с заданными инструкциями.

Материал связан с другими аспектами AQA, такими как локаторы, Page Object Model (POM), тестирование и клиент-серверная модель, чтобы показать применимость параметризованных тестов в реальных задачах автоматизации.

---

## 1. Что такое `@ParameterizedTest`?

- **Описание**: `@ParameterizedTest` — аннотация JUnit 5, которая позволяет выполнять один тестовый метод с разными входными данными, предоставляемыми через источники данных (например, `@ValueSource`, `@CsvSource`). Она является частью модуля `junit-jupiter-params`.
- **Назначение**:
    - Проверка одного сценария с множеством входных данных (например, тестирование формы логина с разными именами пользователей и паролями).
    - Уменьшение дублирования кода: один метод вместо нескольких тестов.
    - Упрощение тестирования граничных случаев и негативных сценариев.
- **Применение в AQA**:
    - Тестирование UI с разными данными ввода (Selenium).
    - Проверка API-ответов на разные запросы.
    - Валидация данных в базе данных (SQL).
- **Зависимости**: Требуется модуль JUnit Jupiter Params:
  ```xml
  <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter-params</artifactId>
      <version>5.11.0</version>
      <scope>test</scope>
  </dependency>
  ```

**Связь с другими темами**:
- **Тестирование**: `@ParameterizedTest` поддерживает структуру AAA (Arrange-Act-Assert), повторяя Act и Assert для разных данных.
- **Локаторы**: Используется с `@FindBy` в POM для проверки элементов UI с разными входными параметрами.
- **Клиент-серверная модель**: Проверяет взаимодействие клиента (тестов) с сервером (браузером, API) на разных данных.

---

## 2. Основные источники данных для `@ParameterizedTest`

JUnit 5 предоставляет несколько способов передачи данных в параметризованные тесты:

### 2.1. `@ValueSource`
- **Описание**: Передаёт массив простых значений (строки, числа, булевы значения и т.д.).
- **Применение**: Проверка одного параметра.
- **Пример**: Тестирование формы логина с разными именами пользователей.
  ```java
  import org.junit.jupiter.params.ParameterizedTest;
  import org.junit.jupiter.params.provider.ValueSource;
  import org.openqa.selenium.WebDriver;
  import org.openqa.selenium.chrome.ChromeDriver;
  import org.openqa.selenium.support.FindBy;
  import org.openqa.selenium.support.PageFactory;

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

          public void enterUsername(String username) {
              inputUsername.sendKeys(username);
              submitButton.click();
          }
      }

      @BeforeEach
      void setUp() {
          System.setProperty("webdriver.chrome.driver", "/path/to/chromedriver");
          driver = new ChromeDriver();
          driver.get("https://example.com/login");
      }

      @ParameterizedTest
      @ValueSource(strings = {"John", "Alice", "Bob"})
      void testLoginWithDifferentUsers(String username) {
          LoginPage page = new LoginPage(driver);
          page.enterUsername(username);
          assertEquals("Welcome, " + username, driver.findElement(By.id("message")).getText());
      }

      @AfterEach
      void tearDown() {
          driver.quit();
      }
  }
  ```

### 2.2. `@CsvSource`
- **Описание**: Передаёт данные в формате CSV (строки, разделённые запятыми), где каждая строка — набор параметров.
- **Применение**: Проверка нескольких параметров (например, логин и пароль).
- **Пример**: Тестирование формы логина с парами логин-пароль.
  ```java
  import org.junit.jupiter.params.ParameterizedTest;
  import org.junit.jupiter.params.provider.CsvSource;

  public class LoginCsvTest {
      private WebDriver driver;

      public class LoginPage {
          @FindBy(id = "username")
          private WebElement inputUsername;

          @FindBy(id = "password")
          private WebElement inputPassword;

          @FindBy(id = "submit")
          private WebElement submitButton;

          public LoginPage(WebDriver driver) {
              PageFactory.initElements(driver, this);
          }

          public void login(String username, String password) {
              inputUsername.sendKeys(username);
              inputPassword.sendKeys(password);
              submitButton.click();
          }

          public String getMessage() {
              return driver.findElement(By.id("message")).getText();
          }
      }

      @BeforeEach
      void setUp() {
          System.setProperty("webdriver.chrome.driver", "/path/to/chromedriver");
          driver = new ChromeDriver();
          driver.get("https://example.com/login");
      }

      @ParameterizedTest
      @CsvSource({
          "John, pass123, Welcome, John",
          "Alice, pass456, Welcome, Alice"
      })
      void testLoginWithCredentials(String username, String password, String expectedMessage) {
          LoginPage page = new LoginPage(driver);
          page.login(username, password);
          assertEquals(expectedMessage, page.getMessage());
      }

      @AfterEach
      void tearDown() {
          driver.quit();
      }
  }
  ```

### 2.3. `@MethodSource`
- **Описание**: Позволяет использовать метод для предоставления данных.
- **Применение**: Сложные данные (объекты, списки).
- **Пример**: Тестирование с пользовательскими объектами.
  ```java
  import org.junit.jupiter.params.ParameterizedTest;
  import org.junit.jupiter.params.provider.MethodSource;
  import java.util.stream.Stream;

  public class LoginMethodSourceTest {
      private WebDriver driver;

      static class User {
          String username;
          String password;
          String expectedMessage;

          User(String username, String password, String expectedMessage) {
              this.username = username;
              this.password = password;
              this.expectedMessage = expectedMessage;
          }
      }

      static Stream<User> userProvider() {
          return Stream.of(
              new User("John", "pass123", "Welcome, John"),
              new User("Alice", "pass456", "Welcome, Alice")
          );
      }

      public class LoginPage {
          @FindBy(id = "username")
          private WebElement inputUsername;

          @FindBy(id = "password")
          private WebElement inputPassword;

          @FindBy(id = "submit")
          private WebElement submitButton;

          public LoginPage(WebDriver driver) {
              PageFactory.initElements(driver, this);
          }

          public void login(String username, String password) {
              inputUsername.sendKeys(username);
              inputPassword.sendKeys(password);
              submitButton.click();
          }

          public String getMessage() {
              return driver.findElement(By.id("message")).getText();
          }
      }

      @BeforeEach
      void setUp() {
          System.setProperty("webdriver.chrome.driver", "/path/to/chromedriver");
          driver = new ChromeDriver();
          driver.get("https://example.com/login");
      }

      @ParameterizedTest
      @MethodSource("userProvider")
      void testLoginWithUserObject(User user) {
          LoginPage page = new LoginPage(driver);
          page.login(user.username, user.password);
          assertEquals(user.expectedMessage, page.getMessage());
      }

      @AfterEach
      void tearDown() {
          driver.quit();
      }
  }
  ```

### 2.4. `@CsvFileSource`
- **Описание**: Читает данные из CSV-файла.
- **Применение**: Для больших наборов тестовых данных.
- **Пример**: Файл `testdata.csv`:
  ```
  username,password,expectedMessage
  John,pass123,Welcome, John
  Alice,pass456,Welcome, Alice
  ```
  Код теста:
  ```java
  import org.junit.jupiter.params.ParameterizedTest;
  import org.junit.jupiter.params.provider.CsvFileSource;

  public class LoginCsvFileTest {
      private WebDriver driver;

      public class LoginPage {
          @FindBy(id = "username")
          private WebElement inputUsername;

          @FindBy(id = "password")
          private WebElement inputPassword;

          @FindBy(id = "submit")
          private WebElement submitButton;

          public LoginPage(WebDriver driver) {
              PageFactory.initElements(driver, this);
          }

          public void login(String username, String password) {
              inputUsername.sendKeys(username);
              inputPassword.sendKeys(password);
              submitButton.click();
          }

          public String getMessage() {
              return driver.findElement(By.id("message")).getText();
          }
      }

      @BeforeEach
      void setUp() {
          System.setProperty("webdriver.chrome.driver", "/path/to/chromedriver");
          driver = new ChromeDriver();
          driver.get("https://example.com/login");
      }

      @ParameterizedTest
      @CsvFileSource(resources = "/testdata.csv")
      void testLoginWithCsvFile(String username, String password, String expectedMessage) {
          LoginPage page = new LoginPage(driver);
          page.login(username, password);
          assertEquals(expectedMessage, page.getMessage());
      }

      @AfterEach
      void tearDown() {
          driver.quit();
      }
  }
  ```

### 2.5. `@ArgumentsSource`
- **Описание**: Позволяет создавать пользовательский источник данных.
- **Применение**: Для сложных данных, не поддерживаемых стандартными источниками.
- **Пример**:
  ```java
  import org.junit.jupiter.params.ParameterizedTest;
  import org.junit.jupiter.params.provider.Arguments;
  import org.junit.jupiter.params.provider.ArgumentsSource;

  public class CustomArgumentsTest {
      static class CustomProvider implements ArgumentsProvider {
          @Override
          public Stream<? extends Arguments> provideArguments(ExtensionContext context) {
              return Stream.of(
                  Arguments.of("John", "pass123"),
                  Arguments.of("Alice", "pass456")
              );
          }
      }

      @ParameterizedTest
      @ArgumentsSource(CustomProvider.class)
      void testLoginWithCustomProvider(String username, String password) {
          WebDriver driver = new ChromeDriver();
          driver.get("https://example.com/login");
          LoginPage page = new LoginPage(driver);
          page.login(username, password);
          assertEquals("Welcome, " + username, page.getMessage());
          driver.quit();
      }
  }
  ```

---

## 3. Преимущества и недостатки `@ParameterizedTest`

### 3.1. Преимущества
- **Уменьшение дублирования**: Один тестовый метод для множества данных.
- **Гибкость**: Поддержка различных источников данных (`@ValueSource`, `@CsvSource`, `@MethodSource`).
- **Читаемость**: Тестовые данные явно заданы в аннотациях или файлах.
- **Покрытие**: Упрощает тестирование граничных случаев и негативных сценариев.
- **Интеграция с POM**: Хорошо сочетается с `@FindBy` для UI-тестов.

### 3.2. Недостатки
- **Сложность отладки**: Ошибка в одном наборе данных может затруднить анализ.
- **Ограничения источников**: `@ValueSource` не поддерживает сложные объекты.
- **Производительность**: Множество тестов увеличивает время выполнения.
- **Зависимость от данных**: Неверные данные в CSV или методе могут привести к сбоям.

**Связь с другими темами**:
- **Локаторы**: `@ParameterizedTest` использует `@FindBy` для проверки элементов с разными данными.
- **ООП**: Интеграция с Page Object Model для структурированных тестов.
- **Тестирование**: Поддерживает AAA, где Act и Assert повторяются для разных данных.

---

## 4. Практические примеры в AQA

### 4.1. Негативное тестирование с `@CsvSource`
Проверка реакции на некорректные данные:
```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.CsvSource;

public class NegativeLoginTest {
    private WebDriver driver;

    public class LoginPage {
        @FindBy(id = "username")
        private WebElement inputUsername;

        @FindBy(id = "password")
        private WebElement inputPassword;

        @FindBy(id = "submit")
        private WebElement submitButton;

        @FindBy(className = "error")
        private WebElement errorMessage;

        public LoginPage(WebDriver driver) {
            PageFactory.initElements(driver, this);
        }

        public void login(String username, String password) {
            inputUsername.sendKeys(username);
            inputPassword.sendKeys(password);
            submitButton.click();
        }

        public String getErrorMessage() {
            return errorMessage.getText();
        }
    }

    @BeforeEach
    void setUp() {
        System.setProperty("webdriver.chrome.driver", "/path/to/chromedriver");
        driver = new ChromeDriver();
        driver.get("https://example.com/login");
    }

    @ParameterizedTest
    @CsvSource({
        "John, '', Password is required",
        "'', pass123, Username is required",
        "invalid, wrong, Invalid credentials"
    })
    void testNegativeLogin(String username, String password, String expectedError) {
        LoginPage page = new LoginPage(driver);
        page.login(username, password);
        assertEquals(expectedError, page.getErrorMessage());
    }

    @AfterEach
    void tearDown() {
        driver.quit();
    }
}
```

### 4.2. Тестирование с `@MethodSource` для сложных данных
Проверка формы с пользовательскими объектами:
```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.MethodSource;
import java.util.stream.Stream;

public class ComplexLoginTest {
    private WebDriver driver;

    static class LoginData {
        String username;
        String password;
        boolean shouldPass;
        String expectedMessage;

        LoginData(String username, String password, boolean shouldPass, String expectedMessage) {
            this.username = username;
            this.password = password;
            this.shouldPass = shouldPass;
            this.expectedMessage = expectedMessage;
        }
    }

    static Stream<LoginData> loginDataProvider() {
        return Stream.of(
            new LoginData("John", "pass123", true, "Welcome, John"),
            new LoginData("invalid", "wrong", false, "Invalid credentials")
        );
    }

    public class LoginPage {
        @FindBy(id = "username")
        private WebElement inputUsername;

        @FindBy(id = "password")
        private WebElement inputPassword;

        @FindBy(id = "submit")
        private WebElement submitButton;

        @FindBy(id = "message")
        private WebElement message;

        public LoginPage(WebDriver driver) {
            PageFactory.initElements(driver, this);
        }

        public void login(String username, String password) {
            inputUsername.sendKeys(username);
            inputPassword.sendKeys(password);
            submitButton.click();
        }

        public String getMessage() {
            return message.getText();
        }
    }

    @BeforeEach
    void setUp() {
        System.setProperty("webdriver.chrome.driver", "/path/to/chromedriver");
        driver = new ChromeDriver();
        driver.get("https://example.com/login");
    }

    @ParameterizedTest
    @MethodSource("loginDataProvider")
    void testComplexLogin(LoginData data) {
        LoginPage page = new LoginPage(driver);
        page.login(data.username, data.password);
        assertEquals(data.expectedMessage, page.getMessage());
        assertEquals(data.shouldPass, page.getMessage().startsWith("Welcome"));
    }

    @AfterEach
    void tearDown() {
        driver.quit();
    }
}
```

---

## 5. Рекомендации

- **Выбор источника данных**:
    - Используйте `@ValueSource` для простых тестов с одним параметром.
    - Применяйте `@CsvSource` для парного ввода (логин-пароль).
    - Используйте `@MethodSource` для сложных объектов или динамических данных.
    - Храните большие наборы данных в CSV-файлах с `@CsvFileSource`.

- **Интеграция с POM**:
    - Комбинируйте `@ParameterizedTest` с `@FindBy` для структурированных UI-тестов:
      ```java
      @ParameterizedTest
      @ValueSource(strings = {"John", "Alice"})
      void testLogin(String username) {
          LoginPage page = new LoginPage(driver);
          page.login(username, "pass123");
      }
      ```

- **Ожидания**:
    - Добавляйте явные ожидания для динамических элементов:
      ```java
      WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
      wait.until(ExpectedConditions.elementToBeClickable(By.id("submit")));
      ```

- **Интеграция с CI/CD**:
    - Настройте запуск параметризованных тестов в GitHub Actions:
      ```yaml
      name: Run Parameterized Tests
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
            - name: Install ChromeDriver
              run: |
                sudo apt-get update
                sudo apt-get install -y chromium-chromedriver
            - name: Run tests
              run: mvn test
      ```

- **Отладка**:
    - Используйте IntelliJ IDEA для отладки: установите точку останова в тесте и проверяйте параметры.
    - Логируйте данные:
      ```java
      System.out.println("Тестируем: username=" + username + ", password=" + password);
      ```

- **Стабильные локаторы**:
    - Используйте `@FindBy` с устойчивыми селекторами (например, `data-testid`):
      ```java
      @FindBy(css = "[data-testid='submit']")
      private WebElement submitButton;
      ```

- **Документирование**:
    - Добавляйте комментарии к тестам:
      ```java
      @ParameterizedTest
      @CsvSource({
          "John, pass123, Welcome, John  // Валидные данные",
          "invalid, wrong, Invalid credentials  // Негативный тест"
      })
      void testLogin(String username, String password, String expectedMessage) {
          // ...
      }
      ```

---

## 6. Связь с другими темами

- **Локаторы**:
    - `@ParameterizedTest` использует `@FindBy` для проверки элементов с разными данными.
    - Пример: Тестирование формы с разными значениями логина.
- **ООП**:
    - Интеграция с Page Object Model:
      ```java
      public class LoginPage {
          @FindBy(id = "username")
          private WebElement inputUsername;
  
          public LoginPage(WebDriver driver) {
              PageFactory.initElements(driver, this);
          }
      }
      ```
- **Тестирование**:
    - Поддерживает AAA: Arrange (настройка страницы), Act (ввод данных), Assert (проверка результата).
- **Клиент-серверная модель**:
    - Проверяет взаимодействие клиента (Selenium) с сервером (браузер) на разных данных.

---

## 7. Источники
- [JUnit 5 User Guide: Parameterized Tests](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests)
- [Baeldung: JUnit 5 Parameterized Tests](https://www.baeldung.com/parameterized-tests-junit-5)
- [Selenium Documentation](https://www.selenium.dev/documentation/)
- [Guru99: JUnit Parameterized Tests](https://www.guru99.com/parameterized-test-in-junit.html)

---
