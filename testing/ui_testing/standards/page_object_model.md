# Выбор и структура Page Object модели

Page Object Model (POM) — это шаблон проектирования, используемый в автоматизации UI-тестирования для повышения читаемости, повторного использования и поддерживаемости кода. Он разделяет логику страниц и тестов, позволяя изолировать работу с элементами интерфейса от тестовых сценариев. В статье описаны ключевые аспекты выбора и реализации POM, включая разделение логики, структуру классов, использование компонентов и лучшие практики.

## Основы Page Object модели

POM предполагает, что каждая страница или компонент веб-приложения представлен отдельным классом. Это упрощает поддержку тестов: изменения в интерфейсе затрагивают только соответствующий класс страницы, а тесты остаются неизменными.

- **Один класс — одна страница или компонент**: Каждый класс (`LoginPage`, `DashboardPage`, `Header`) отвечает за конкретную страницу или компонент интерфейса.
- **Методы описывают действия и проверки**: Например, методы `login(String username, String password)` или `isUserLoggedIn()` отражают пользовательские действия, а не технические детали, такие как локаторы или ожидания.
- **Разделение логики**: Логика взаимодействия с UI (например, клики, ввод текста) находится в классах страниц, а тесты содержат только сценарии и проверки.

Пример реализации страницы логина с использованием Selenide и Page Object:

```java
import com.codeborne.selenide.SelenideElement;
import lombok.Getter;
import org.openqa.selenium.support.FindBy;
import static com.codeborne.selenide.Selenide.$;

@Getter
public class LoginPage {
    // Локаторы элементов страницы
    private final SelenideElement usernameField = $("#username");
    private final SelenideElement passwordField = $("#password");
    private final SelenideElement loginButton = $("#login-button");

    // Действие: ввод логина и пароля, клик по кнопке
    public DashboardPage login(String username, String password) {
        usernameField.setValue(username);
        passwordField.setValue(password);
        loginButton.click();
        return new DashboardPage();
    }

    // Проверка: отображается ли сообщение об ошибке
    public boolean isErrorMessageDisplayed() {
        return $(".error-message").isDisplayed();
    }
}
```

Пример теста с использованием `LoginPage`:

```java
import org.junit.jupiter.api.Test;
import static org.assertj.core.api.Assertions.assertThat;

public class LoginTest extends BaseTest {
    @Test
    void shouldLoginWithValidCredentials() {
        // Arrange
        LoginPage loginPage = new LoginPage();
        String username = "user";
        String password = "pass";

        // Act
        DashboardPage dashboardPage = loginPage.login(username, password);

        // Assert
        assertThat(dashboardPage.isUserLoggedIn()).isTrue();
    }
}
```

## Использование базовых классов

Для избежания дублирования кода создаются базовые классы `BasePage` и `BaseTest`, содержащие общие функции, такие как настройка WebDriver, ожидания и логирование.

### BasePage

Класс `BasePage` содержит методы, общие для всех страниц, например, ожидания видимости элемента или инициализацию WebDriver.

```java
import com.codeborne.selenide.Configuration;
import com.codeborne.selenide.Selenide;
import com.codeborne.selenide.WebDriverRunner;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public abstract class BasePage {
    protected final Logger logger = LoggerFactory.getLogger(this.getClass());
    protected final WebDriver driver;

    public BasePage() {
        WebDriverManager.chromedriver().setup();
        this.driver = WebDriverRunner.getWebDriver();
        Configuration.timeout = 10000; // Глобальный таймаут для Selenide
    }

    // Ожидание видимости элемента
    public void waitForElementVisible(SelenideElement element) {
        element.shouldBe(visible, Duration.ofSeconds(5));
    }

    // Получение текущего URL
    public String getCurrentUrl() {
        return driver.getCurrentUrl();
    }
}
```

### BaseTest

Класс `BaseTest` настраивает окружение для тестов, включая WebDriver, Allure и логирование.

```java
import io.qameta.allure.junit5.AllureJunit5;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.extension.ExtendWith;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@ExtendWith(AllureJunit5.class)
public abstract class BaseTest {
    protected final Logger logger = LoggerFactory.getLogger(this.getClass());

    @BeforeEach
    void setUp() {
        WebDriverManager.chromedriver().setup();
        Configuration.browser = "chrome";
        Configuration.startMaximized = true;
        logger.info("Browser initialized");
    }

    @AfterEach
    void tearDown() {
        Selenide.closeWebDriver();
        logger.info("Browser closed");
    }
}
```

## Компоненты в POM

Компоненты (например, `Header`, `Sidebar`, `Modal`) представляют повторяющиеся элементы интерфейса, которые можно вынести в отдельные классы. Это уменьшает дублирование кода и упрощает поддержку.

Пример реализации компонента `Header`:

```java
import com.codeborne.selenide.SelenideElement;
import lombok.Getter;
import static com.codeborne.selenide.Selenide.$;

@Getter
public class HeaderComponent {
    private final SelenideElement userMenu = $("#user-menu");
    private final SelenideElement logoutButton = $("#logout-button");

    public void logout() {
        userMenu.click();
        logoutButton.click();
    }

    public boolean isUserMenuDisplayed() {
        return userMenu.isDisplayed();
    }
}
```

Использование компонента в классе страницы:

```java
public class DashboardPage extends BasePage {
    private final HeaderComponent header = new HeaderComponent();

    public HeaderComponent getHeader() {
        return header;
    }

    public boolean isUserLoggedIn() {
        return header.isUserMenuDisplayed();
    }
}
```

## Лучшие практики и устойчивость тестов

Для создания стабильных и поддерживаемых тестов в POM важно учитывать следующие аспекты:

- **Стабильные локаторы**: Используйте `id`, `data-*` атрибуты или уникальные CSS/XPath-селекторы. Избегайте динамических или хрупких локаторов, таких как длинные XPath (`//div[3]/span[2]`).
- **Ожидания**: Применяйте `WebDriverWait` или Selenide для ожидания видимости, кликабельности или других состояний элементов.
- **Проверки**: Используйте методы `isDisplayed()`, `isEnabled()` для проверки состояния элементов перед взаимодействием.
- **Логирование**: Логируйте ключевые действия (например, клики, ввод текста) через SLF4J для упрощения отладки.
- **PageFactory (опционально)**: Если не используете Selenide, можно применять `@FindBy` с PageFactory для инициализации элементов.

Пример использования ожиданий в Selenide:

```java
import static com.codeborne.selenide.Condition.visible;
import static com.codeborne.selenide.Selenide.$;

public class LoginPage {
    private final SelenideElement submitButton = $("#submit-button");

    public void clickSubmitButton() {
        submitButton.shouldBe(visible).click();
    }
}
```

## Настройка CI/CD

Для интеграции тестов в CI/CD (например, Jenkins) настройте Maven в pipeline. Пример конфигурации `Jenkinsfile`:

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Report') {
            steps {
                allure([
                    includeProperties: false,
                    jdk: '',
                    results: [[path: 'target/allure-results']]
                ])
            }
        }
    }
}
```

Необходимые зависимости в `pom.xml`:

```xml
<dependencies>
    <dependency>
        <groupId>io.qameta.allure</groupId>
        <artifactId>allure-junit5</artifactId>
        <version>2.21.0</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>com.codeborne</groupId>
        <artifactId>selenide</artifactId>
        <version>6.19.1</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-simple</artifactId>
        <version>2.0.9</version>
    </dependency>
    <dependency>
        <groupId>io.github.bonigarcia</groupId>
        <artifactId>webdrivermanager</artifactId>
        <version>5.9.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

## Отладка и распространённые ошибки

- **Отладка в IntelliJ IDEA**:
  - Устанавливайте точки останова в тестах или классах страниц.
  - Используйте DevTools браузера для проверки локаторов.
  - Логируйте действия через SLF4J для анализа последовательности выполнения.
- **Распространённые ошибки**:
  - **Flaky тесты**: Возникают из-за отсутствия ожиданий или нестабильных локаторов. Решение: используйте `Selenide` или `WebDriverWait`.
  - **Дублирование кода**: Избегайте копирования методов, выносите общую логику в `BasePage` или компоненты.
  - **Сложные локаторы**: Упрощайте локаторы, отдавая предпочтение `id` или `data-*`.

## Связь с клиент-серверной моделью

POM изолирует взаимодействие с UI, но тесты могут включать проверки клиент-серверных взаимодействий:
- **UI-тестирование**: Проверяйте, что данные, отображаемые на странице, соответствуют ответам API.
- **API-тестирование**: Используйте RestAssured для проверки серверных ответов перед взаимодействием с UI.

Пример проверки API с RestAssured:

```java
import io.restassured.RestAssured;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class ApiTest {
    @Test
    void shouldReturnUserData() {
        RestAssured.baseURI = "https://api.example.com";
        given()
            .when()
            .get("/user/1")
            .then()
            .statusCode(200)
            .body("username", equalTo("user"));
    }
}
```

## Дополнительно

- **Testcontainers**: Используйте для запуска браузеров в Docker-контейнерах, чтобы обеспечить воспроизводимость тестов.
- **Checkstyle/PMD/SpotBugs**: Настройте статический анализ кода в Maven для поддержания качества.
- **Allure TestOps**: Интегрируйте Allure с Jenkins для централизованного хранения отчётов.

## Источники
- [Selenide Documentation](https://selenide.org/documentation.html)
- [Allure Framework](https://docs.qameta.io/allure/)
- [WebDriverManager](https://bonigarcia.dev/webdrivermanager/)

---
[**← Назад к оглавлению**](../README.md)