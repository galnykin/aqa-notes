# Практики автоматизации тестирования

## Один тест — один сценарий

Каждый тест должен проверять ровно один сценарий, чтобы упростить отладку и обеспечить независимость тестов. Это снижает сложность анализа результатов и минимизирует влияние ошибок одного теста на другие.

### Пример теста (JUnit 5)
```java
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

import static org.junit.jupiter.api.Assertions.assertEquals;

public class SingleScenarioTest {
    private WebDriver driver;

    @Test
    void verifyPageTitle() {
        driver = new ChromeDriver();
        try {
            driver.get("https://example.com");
            String actualTitle = driver.getTitle();
            assertEquals("Example Domain", actualTitle, "Page title does not match");
        } finally {
            driver.quit();
        }
    }
}
```

### Пример теста (TestNG)
```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.Test;

import static org.testng.Assert.assertEquals;

public class SingleScenarioTest {
    private WebDriver driver;

    @Test
    public void verifyPageTitle() {
        driver = new ChromeDriver();
        driver.get("https://example.com");
        String actualTitle = driver.getTitle();
        assertEquals(actualTitle, "Example Domain", "Page title does not match");
    }

    @AfterMethod
    public void tearDown() {
        if (driver != null) {
            driver.quit();
        }
    }
}
```

### Рекомендации
- Пишите тесты, проверяющие одну функциональность (например, проверка заголовка страницы или отправка формы).
- Избегайте включения нескольких проверок в один тест, чтобы упростить диагностику ошибок.
- Используйте аннотации `@DisplayName` (JUnit 5) или `description` (TestNG) для описания сценария.

## Минимизация дублирования кода

Для устранения дублирования используйте шаблон Page Object Model (POM), утилитные классы и аннотации для повторяющихся действий.

### Пример Page Object
```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.openqa.selenium.support.ui.WebDriverWait;

import java.time.Duration;

public class LoginPage {
    private final WebDriver driver;
    @FindBy(id = "username") private WebElement usernameField;
    @FindBy(id = "password") private WebElement passwordField;
    @FindBy(id = "loginButton") private WebElement loginButton;

    public LoginPage(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this);
    }

    public void login(String username, String password) {
        usernameField.sendKeys(username);
        passwordField.sendKeys(password);
        loginButton.click();
        new WebDriverWait(driver, Duration.ofSeconds(5))
                .until(d -> loginButton.isDisplayed());
    }
}
```

### Пример теста с POM (JUnit 5)
```java
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

public class LoginTest {
    private WebDriver driver;

    @Test
    void testLogin() {
        driver = new ChromeDriver();
        LoginPage loginPage = new LoginPage(driver);
        driver.get("https://example.com/login");
        loginPage.login("user", "password");
        // Проверка результата
    }

    @AfterEach
    void tearDown() {
        if (driver != null) {
            driver.quit();
        }
    }
}
```

### Рекомендации
- Используйте POM для инкапсуляции работы с элементами страницы.
- Создавайте утилитные методы для повторяющихся операций (например, ожидания или проверки).
- Применяйте Lombok для сокращения шаблонного кода (например, геттеров/сеттеров).

## Чёткие названия методов и тестов

Названия тестов и методов должны быть понятными, описывать цель теста и ожидаемый результат.

### Пример именования
```java
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

import static org.junit.jupiter.api.Assertions.assertTrue;

public class ClearNamingTest {
    @Test
    void shouldDisplayErrorMessageOnInvalidLogin() {
        WebDriver driver = new ChromeDriver();
        try {
            driver.get("https://example.com/login");
            LoginPage loginPage = new LoginPage(driver);
            loginPage.login("invalid", "wrong");
            assertTrue(loginPage.isErrorMessageDisplayed(), "Error message not displayed");
        } finally {
            driver.quit();
        }
    }
}
```

### Рекомендации
- Используйте формат `should[Action][Condition]` для тестов (например, `shouldLoginWithValidCredentials`).
- Избегайте общих названий вроде `test1` или `check`.
- Включайте контекст в названия методов Page Object (например, `enterUsername` вместо `enterText`).

## Устойчивость к изменениям UI

Для устойчивости тестов к изменениям UI используйте стабильные локаторы и ожидания.

### Пример стабильных локаторов и ожиданий
```java
import com.codeborne.selenide.SelenideElement;
import org.openqa.selenium.support.FindBy;

import static com.codeborne.selenide.Condition.visible;
import static com.codeborne.selenide.Selenide.$;

public class StableLocatorPage {
    @FindBy(css = "[data-test-id='submit-button']") private SelenideElement submitButton;

    public void clickSubmitButton() {
        submitButton.shouldBe(visible).click();
    }
}
```

### Рекомендации
- Предпочитайте CSS или XPath с атрибутами `data-test-id` для локаторов.
- Используйте `WebDriverWait` или Selenide `shouldBe` для ожидания элементов.
- Проверяйте состояние элементов с помощью `isDisplayed()` и `isEnabled()` перед взаимодействием.

## Запуск в разных окружениях и браузерах

Для поддержки разных окружений (dev, qa, prod) и браузеров используйте конфигурационные файлы и параметры WebDriver.

### Пример Maven-профилей
```xml
<profiles>
    <profile>
        <id>dev</id>
        <properties>
            <env.url>https://dev.example.com</env.url>
            <browser>chrome</browser>
        </properties>
    </profile>
    <profile>
        <id>qa</id>
        <properties>
            <env.url>https://qa.example.com</env.url>
            <browser>firefox</browser>
        </properties>
    </profile>
</profiles>
```

### Пример теста с поддержкой окружений
```java
import com.codeborne.selenide.Configuration;
import org.junit.jupiter.api.Test;

import static com.codeborne.selenide.Selenide.open;

public class MultiEnvironmentTest {
    static {
        String envUrl = System.getProperty("env.url", "https://qa.example.com");
        String browser = System.getProperty("browser", "chrome");
        Configuration.baseUrl = envUrl;
        Configuration.browser = browser;
    }

    @Test
    void testOnEnvironment() {
        open("/login");
        // Логика теста
    }
}
```

### Пример запуска в CI/CD (GitHub Actions)
```yaml
name: CI Pipeline
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        env: [dev, qa]
        browser: [chrome, firefox]
    steps:
      - uses: actions/checkout@v4
      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          java-version: '17'
      - name: Run tests
        run: mvn test -P${{ matrix.env }} -Dbrowser=${{ matrix.browser }}
```

### Рекомендации
- Используйте системные переменные или Maven-профили для управления окружениями.
- Настройте Selenoid или Selenium Grid для запуска тестов в разных браузерах.
- Проверяйте доступность окружения перед запуском тестов.

## Читаемость важнее компактности

Читаемый код облегчает поддержку и совместную работу. Используйте понятные структуры и избегайте чрезмерной оптимизации за счёт ясности.

### Пример читаемого кода
```java
import com.codeborne.selenide.SelenideElement;
import org.junit.jupiter.api.Test;

import static com.codeborne.selenide.Selenide.$;
import static com.codeborne.selenide.Condition.visible;

public class ReadableTest {
    @Test
    void shouldSubmitFormWithValidData() {
        open("https://example.com/form");
        SelenideElement nameField = $("#name").shouldBe(visible);
        SelenideElement submitButton = $("#submit").shouldBe(visible);

        nameField.setValue("John Doe");
        submitButton.click();

        $("#successMessage").shouldBe(visible);
    }
}
```

### Рекомендации
- Разбивайте сложные тесты на шаги с помощью методов или `@Step` (Allure).
- Используйте понятные имена переменных (например, `nameField` вместо `field1`).
- Добавляйте комментарии только для сложной логики, избегая очевидных пояснений.

## Дополнительно

### Отладка
- **IntelliJ IDEA Debug**: Используйте точки останова для проверки локаторов и состояний элементов.
- **Логирование**: Настройте SLF4J с Logback для записи ключевых действий теста.
- **Ожидания**: Применяйте `WebDriverWait` или Selenide `shouldBe` для синхронизации.

### Распространённые ошибки
- **Сложные тесты**: Избегайте тестов, проверяющих несколько сценариев.
- **Хрупкие локаторы**: Используйте атрибуты `data-test-id` вместо классов или id, зависящих от стилей.
- **Зависимости окружений**: Убедитесь, что тесты изолированы и не зависят от конкретного окружения.

## Источники
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/)
- [Selenide Documentation](https://selenide.org/documentation.html)
- [TestNG Documentation](https://testng.org/doc/)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)

---
[**← Назад к оглавлению**](../README.md)