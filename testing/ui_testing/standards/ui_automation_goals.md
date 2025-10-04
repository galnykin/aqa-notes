# Цели и принципы UI-автоматизации

UI-автоматизация направлена на проверку пользовательских сценариев через интерфейс приложения, обеспечивая стабильность, скорость и качество тестирования. Эта статья раскрывает цели UI-автоматизации, ключевые принципы написания тестов и подходы к их реализации с использованием Java, Selenium WebDriver, Selenide и Page Object Model.

## Цели UI-автоматизации

UI-автоматизация решает несколько задач, связанных с тестированием пользовательского интерфейса:

- **Проверка пользовательских сценариев**: Тесты имитируют действия пользователя (клики, ввод данных, навигация), чтобы убедиться, что интерфейс работает корректно.
- **Раннее выявление дефектов**: Автоматизация помогает обнаружить визуальные (например, некорректное отображение элементов) и функциональные (например, ошибки в обработке данных) проблемы на ранних стадиях.
- **Ускорение регрессионного тестирования**: Автоматизированные тесты выполняются быстрее, чем ручные, что позволяет проверять продукт перед каждым релизом.
- **Обеспечение стабильности при частых релизах**: Тесты гарантируют, что новые изменения не ломают существующий функционал.

## Принципы UI-автоматизации

Для создания эффективных и устойчивых тестов следуют четырём ключевым принципам:

- **Читаемость**: Код тестов должен быть понятным, с ясной структурой и комментариями.
- **Повторяемость**: Тесты должны давать одинаковый результат при многократном запуске в неизменных условиях.
- **Независимость**: Каждый тест выполняется изолированно, без зависимости от других тестов или внешних факторов.
- **Устойчивость**: Тесты должны быть устойчивы к изменениям интерфейса, используя стабильные локаторы и ожидания.

## Реализация UI-тестов с использованием Page Object Model

Page Object Model (POM) — это шаблон проектирования, который улучшает читаемость и поддерживаемость тестов. Каждый экран или компонент приложения представлен отдельным классом, содержащим методы для взаимодействия с элементами.

### Пример: Page Object для страницы логина

```java
import com.codeborne.selenide.SelenideElement;
import static com.codeborne.selenide.Selenide.$;

public class LoginPage {
    // Локаторы элементов
    private final SelenideElement usernameField = $("#username");
    private final SelenideElement passwordField = $("#password");
    private final SelenideElement loginButton = $("#login-button");

    // Метод для ввода логина и пароля
    public void enterCredentials(String username, String password) {
        usernameField.setValue(username); // Ввод имени пользователя
        passwordField.setValue(password); // Ввод пароля
    }

    // Метод для нажатия на кнопку логина
    public void clickLoginButton() {
        loginButton.click(); // Клик по кнопке
    }

    // Проверка, что кнопка логина активна
    public boolean isLoginButtonEnabled() {
        return loginButton.isEnabled();
    }
}
```

### Пример теста с JUnit 5

```java
import com.codeborne.selenide.Configuration;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;
import static com.codeborne.selenide.Selenide.open;
import static org.assertj.core.api.Assertions.assertThat;

public class LoginTest {
    private final LoginPage loginPage = new LoginPage();

    @BeforeAll
    static void setUp() {
        Configuration.browser = "chrome";
        Configuration.baseUrl = "https://example.com";
    }

    @Test
    void shouldLoginWithValidCredentials() {
        // Arrange
        open("/login");
        
        // Act
        loginPage.enterCredentials("user", "password");
        loginPage.clickLoginButton();

        // Assert
        assertThat(loginPage.isLoginButtonEnabled()).isTrue();
    }
}
```

### Пример теста с TestNG

```java
import com.codeborne.selenide.Configuration;
import org.testng.annotations.BeforeClass;
import org.testng.annotations.Test;
import static com.codeborne.selenide.Selenide.open;
import static org.testng.Assert.assertTrue;

public class LoginTestNG {
    private final LoginPage loginPage = new LoginPage();

    @BeforeClass
    void setUp() {
        Configuration.browser = "chrome";
        Configuration.baseUrl = "https://example.com";
    }

    @Test
    void shouldLoginWithValidCredentials() {
        // Given
        open("/login");
        
        // When
        loginPage.enterCredentials("user", "password");
        loginPage.clickLoginButton();

        // Then
        assertTrue(loginPage.isLoginButtonEnabled(), "Login button should be enabled");
    }
}
```

## Локаторы и ожидания

### Локаторы

Для поиска элементов в UI-тестах используются стабильные локаторы, такие как `id`, `name`, или `data-testid`. CSS-селекторы и XPath допустимы, но их стоит избегать для динамических элементов, чтобы повысить устойчивость тестов.

Пример стабильного локатора:
```java
SelenideElement submitButton = $("#submit-button"); // По ID
```

### Ожидания

Для обработки асинхронных операций (например, загрузки страницы) применяются явные ожидания. Selenide упрощает их использование, автоматически ожидая видимость или доступность элемента.

Пример явного ожидания:
```java
import static com.codeborne.selenide.Condition.visible;

submitButton.shouldBe(visible); // Ожидание видимости кнопки
```

## Интеграция с CI/CD

Автоматизированные UI-тесты интегрируются в CI/CD-пайплайны для запуска при каждом коммите или по расписанию. Пример конфигурации для GitHub Actions:

```yaml
name: UI Tests
on:
  push:
    branches: [ main ]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
      - name: Run UI tests
        run: mvn clean test
```

Пример `pom.xml` для UI-тестов:
```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>ui-tests</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>com.codeborne</groupId>
            <artifactId>selenide</artifactId>
            <version>6.19.1</version>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.10.3</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testng</groupId>
            <artifactId>testng</artifactId>
            <version>7.10.2</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.assertj</groupId>
            <artifactId>assertj-core</artifactId>
            <version>3.26.3</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.5.0</version>
            </plugin>
        </plugins>
    </build>
</project>
```

## Отладка UI-тестов

Для отладки тестов в IntelliJ IDEA:

1. Установите точку останова (breakpoint) в коде теста.
2. Запустите тест в режиме Debug (Shift+F9).
3. Используйте вкладку Debugger для проверки значений переменных и состояния элементов.
4. Для локаторов: откройте DevTools в браузере (F12) и протестируйте селектор в консоли.

Типичные ошибки:
- Нестабильные локаторы (например, XPath, зависящий от структуры DOM).
- Отсутствие ожиданий для асинхронных операций.
- Зависимость тестов друг от друга.

## Дополнительно

- **Allure для отчётности**: Используйте Allure для генерации читаемых отчётов о прохождении тестов. Пример интеграции:
  ```xml
  <dependency>
      <groupId>io.qameta.allure</groupId>
      <artifactId>allure-junit5</artifactId>
      <version>2.27.0</version>
  </dependency>
  ```
- **WebDriverManager**: Автоматизирует управление драйверами браузера (например, ChromeDriver).
  ```java
  import io.github.bonigarcia.wdm.WebDriverManager;
  
  WebDriverManager.chromedriver().setup();
  ```

- **Логирование**: Используйте SLF4J с Logback для записи действий теста.
  ```java
  import org.slf4j.Logger;
  import org.slf4j.LoggerFactory;
  
  private static final Logger logger = LoggerFactory.getLogger(LoginTest.class);
  
  logger.info("Starting login test");
  ```

## Источники

- [Selenide Documentation](https://selenide.org/documentation.html)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [TestNG Documentation](https://testng.org/doc/)
- [Allure Framework](https://allurereport.org/docs/)

---
[**← Назад к оглавлению**](../README.md)
