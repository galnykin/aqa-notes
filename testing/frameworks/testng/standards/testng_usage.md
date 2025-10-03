# Цели использования TestNG

TestNG — это мощный фреймворк для автоматизации тестирования, который используется для упрощения написания, организации
и выполнения тестов на Java. Он поддерживает сложные сценарии тестирования, такие как параметризация, зависимости между
тестами и группировка, а также интегрируется с инструментами CI/CD, Allure, Selenium WebDriver и RestAssured. В этой
статье рассматриваются ключевые аспекты использования TestNG, его преимущества и практические примеры.

## Основные возможности TestNG

TestNG предоставляет гибкие инструменты для организации тестов, что делает его популярным выбором для автоматизации.
Основные возможности включают:

- **Параметризация тестов**: Позволяет запускать один и тот же тест с разными наборами данных через `@DataProvider`.
- **Зависимости между тестами**: Управление порядком выполнения тестов с помощью аннотаций `@DependsOnMethods` или
  `@DependsOnGroups`.
- **Группировка тестов**: Возможность объединять тесты в группы (например, smoke, regression) для выборочного запуска.
- **Гибкая конфигурация**: Аннотации `@BeforeSuite`, `@BeforeTest`, `@BeforeClass`, `@BeforeMethod`, `@AfterMethod` и
  другие позволяют настраивать подготовку и завершение тестов.
- **Интеграция с инструментами**: Поддержка Allure для отчётности, Jenkins для CI/CD, а также библиотек, таких как
  Selenium и RestAssured.

### Пример настройки проекта с TestNG и Maven

Для начала работы с TestNG необходимо настроить проект с использованием Maven. Ниже приведён пример файла `pom.xml`,
включающего зависимости для TestNG, Selenium, RestAssured и Allure.

```xml

<project>
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- TestNG -->
        <dependency>
            <groupId>org.testng</groupId>
            <artifactId>testng</artifactId>
            <version>7.8.0</version>
            <scope>test</scope>
        </dependency>
        <!-- Selenium WebDriver -->
        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-java</artifactId>
            <version>4.11.0</version>
        </dependency>
        <!-- WebDriverManager -->
        <dependency>
            <groupId>io.github.bonigarcia</groupId>
            <artifactId>webdrivermanager</artifactId>
            <version>5.5.3</version>
        </dependency>
        <!-- RestAssured -->
        <dependency>
            <groupId>io.rest-assured</groupId>
            <artifactId>rest-assured</artifactId>
            <version>5.3.2</version>
            <scope>test</scope>
        </dependency>
        <!-- Allure TestNG -->
        <dependency>
            <groupId>io.qameta.allure</groupId>
            <artifactId>allure-testng</artifactId>
            <version>2.24.0</version>
            <scope>test</scope>
        </dependency>
        <!-- SLF4J + Logback -->
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.4.11</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.1.2</version>
                <configuration>
                    <suiteXmlFiles>
                        <suiteXmlFile>testng.xml</suiteXmlFile>
                    </suiteXmlFiles>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

## Параметризация тестов с @DataProvider

Параметризация позволяет запускать один тест с разными входными данными, что упрощает тестирование различных сценариев.
`@DataProvider` возвращает двумерный массив объектов, который используется в тестах.

### Пример UI-теста с Selenium и @DataProvider

Этот пример демонстрирует тестирование формы логина с разными комбинациями данных.

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import org.openqa.selenium.WebElement;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.DataProvider;
import org.testng.annotations.Test;

import static org.testng.Assert.assertTrue;

public class LoginTest {
    private static final Logger logger = LoggerFactory.getLogger(LoginTest.class);
    private WebDriver driver;

    // Page Object для страницы логина
    public static class LoginPage {
        @FindBy(id = "username")
        private WebElement usernameField;

        @FindBy(id = "password")
        private WebElement passwordField;

        @FindBy(id = "loginButton")
        private WebElement loginButton;

        public LoginPage(WebDriver driver) {
            PageFactory.initElements(driver, this);
        }

        public void login(String username, String password) {
            usernameField.sendKeys(username);
            passwordField.sendKeys(password);
            loginButton.click();
        }
    }

    @BeforeMethod
    public void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        driver.get("http://example.com/login");
        logger.info("Browser started and navigated to login page");
    }

    @DataProvider(name = "loginData")
    public Object[][] loginData() {
        return new Object[][]{
                {"user1", "pass1"},
                {"user2", "pass2"}
        };
    }

    @Test(dataProvider = "loginData")
    public void testLogin(String username, String password) {
        // Arrange
        LoginPage loginPage = new LoginPage(driver);
        logger.info("Testing login with username: {}", username);

        // Act
        loginPage.login(username, password);

        // Assert
        assertTrue(driver.getCurrentUrl().contains("dashboard"), "Login failed for user: " + username);
    }

    @AfterMethod
    public void tearDown() {
        if (driver != null) {
            driver.quit();
            logger.info("Browser closed");
        }
    }
}
```

### Связь с клиент-серверной моделью

В UI-тестах, таких как приведённый выше, взаимодействие с веб-приложением происходит через клиент-серверную модель.
Selenium WebDriver отправляет HTTP-запросы через браузер (клиент) к серверу приложения. Для устойчивости тестов важно
использовать стабильные локаторы (например, `id` или `data-*` атрибуты) и ожидания (`WebDriverWait`).

## Тестирование API с RestAssured

TestNG также удобен для API-тестирования с использованием RestAssured. Ниже приведён пример проверки GET-запроса.

```java
import io.restassured.RestAssured;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testng.annotations.BeforeClass;
import org.testng.annotations.Test;

import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class ApiTest {
    private static final Logger logger = LoggerFactory.getLogger(ApiTest.class);

    @BeforeClass
    public void setUp() {
        RestAssured.baseURI = "https://api.example.com";
        logger.info("Base URI set to {}", RestAssured.baseURI);
    }

    @Test
    public void testGetUser() {
        // Given
        String userId = "123";

        // When
        given()
                .when()
                .get("/users/" + userId)
                .then()
                // Then
                .statusCode(200)
                .body("id", equalTo(userId))
                .log().all();
        logger.info("GET /users/{} request completed successfully", userId);
    }
}
```

### Отладка и распространённые ошибки

- **Flaky тесты**: Для UI-тестов используйте `WebDriverWait` для ожидания элементов. Например:
  ```java
  WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
  wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("elementId")));
  ```
- **Неправильные локаторы**: Избегайте хрупких локаторов, таких как длинные XPath. Используйте `id` или `data-test`
  атрибуты.
- **Ошибки конфигурации**: Убедитесь, что зависимости в `pom.xml` актуальны, а WebDriverManager настроен корректно.
- **Отладка в IntelliJ IDEA**: Устанавливайте точки останова в тестах, чтобы проверить значения переменных или локаторы.

## Интеграция с CI/CD (Jenkins)

TestNG легко интегрируется с Jenkins для автоматизации прогонов тестов. Пример настройки `Jenkinsfile` для
Maven-проекта:

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
        stage('Allure Report') {
            steps {
                allure([
                        includeProperties: false,
                        jdk              : '',
                        results          : [[path: 'target/allure-results']]
                ])
            }
        }
    }
}
```

## Дополнительно

### Использование Allure для отчётности

Allure интегрируется с TestNG для создания наглядных отчётов. Для включения Allure добавьте аннотации, такие как `@Step`
и `@Description`, в тесты. Пример:

```java
import io.qameta.allure.Step;

@Step("Логин пользователя {username}")
public void login(String username, String password) {
    // Логика логина
}
```

### Testcontainers для изолированных окружений

Testcontainers позволяет запускать сервисы (например, базы данных или mock-серверы) в Docker-контейнерах. Пример:

```java
import org.testcontainers.containers.GenericContainer;
import org.testng.annotations.Test;

public class TestcontainersExample {
    @Test
    public void testWithMockServer() {
        try (GenericContainer<?> mockServer = new GenericContainer<>("mockserver/mockserver")
                .withExposedPorts(1080)) {
            mockServer.start();
            String mockServerUrl = "http://" + mockServer.getHost() + ":" + mockServer.getFirstMappedPort();
// Логика теста с использованием mockServerUrl
        }
    }
}
```

### Lombok для сокращения кода

Lombok упрощает создание Page Objects, добавляя аннотации, такие как `@Getter` или `@Setter`. Пример:

```java
import lombok.Getter;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;

public class LoginPage {
    @Getter
    @FindBy(id = "username")
    private WebElement usernameField;
}
```

## Источники

- [TestNG Official Documentation](https://testng.org/doc/)
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/webdriver/)
- [RestAssured Documentation](http://rest-assured.io/)
- [Allure Framework Documentation](https://docs.qameta.io/allure/)
- [WebDriverManager GitHub](https://github.com/bonigarcia/webdrivermanager)

---
[**← Назад к оглавлению**](../README.md)
