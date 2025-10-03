# Конфигурация testng.xml

Файл `testng.xml` в TestNG предоставляет гибкий способ управления тестами, включая структурирование тестовых наборов, настройку параметров, использование слушателей (listeners) и управление параллельным выполнением. Эта статья дополняет предыдущие материалы, фокусируясь на создании структурированного `testng.xml`, поддержке параметров через `<parameter>`, применении слушателей и настройке параллельного выполнения с использованием `parallel` и `thread-count`.

## Структурирование testng.xml

Файл `testng.xml` позволяет организовать тесты в иерархию: `<suite>` (тестовая свита) содержит `<test>` (тестовые наборы), которые, в свою очередь, включают классы, методы или группы. Структурирование помогает разделять тесты по типу, окружению или другим критериям.

### Пример `testng.xml`

```xml
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="MainTestSuite" parallel="tests" thread-count="2">
    <test name="UITests">
        <parameter name="env" value="qa"/>
        <groups>
            <run>
                <include name="ui"/>
                <include name="smoke"/>
            </run>
        </groups>
        <classes>
            <class name="com.example.tests.ui.UserAuthTest"/>
        </classes>
    </test>
    <test name="APITests">
        <parameter name="env" value="prod"/>
        <groups>
            <run>
                <include name="api"/>
                <include name="regression"/>
            </run>
        </groups>
        <classes>
            <class name="com.example.tests.api.UserApiTest"/>
        </classes>
    </test>
    <listeners>
        <listener class-name="com.example.tests.listeners.CustomTestListener"/>
    </listeners>
</suite>
```

### Пояснения
- **`<suite>`**: Определяет тестовую свиту с атрибутами `name`, `parallel` и `thread-count`.
- **`<test>`**: Разделяет тесты на логические наборы (например, UI и API).
- **`<parameter>`**: Задаёт параметры, доступные в тестах.
- **`<groups>`**: Указывает, какие группы тестов запускать.
- **`<classes>`**: Содержит классы с тестами.
- **`<listeners>`**: Подключает кастомные слушатели для обработки событий.

### Лучшие практики
- **Логическое разделение**: Разделяйте тесты по слоям (UI, API), окружениям (qa, prod) или типам (smoke, regression).
- **Чёткие имена**: Используйте понятные имена для `<suite>` и `<test>`, например, `MainTestSuite`, `UITests`.
- **Модульность**: Создавайте отдельные `testng.xml` для разных сценариев (например, `testng-smoke.xml`, `testng-regression.xml`).

## Поддержка параметров через `<parameter>`

Параметры в `testng.xml` задаются с помощью тега `<parameter>` и доступны в тестах через аннотацию `@Parameters`. Это позволяет настраивать тесты для разных окружений или конфигураций.

### Пример кода (TestNG с параметрами)

```java
import io.qameta.allure.Description;
import org.openqa.selenium.support.PageFactory;
import org.testng.annotations.Parameters;
import org.testng.annotations.Test;
import static org.testng.Assert.assertTrue;

public class ParameterizedTest extends BaseTest {

    @Test(groups = {"ui", "smoke"})
    @Parameters("env")
    @Description("Проверка загрузки главной страницы в указанном окружении")
    public void testHomePageLoad(String env) {
        // Arrange
        String baseUrl = env.equals("qa") ? "https://qa.example.com" : "https://prod.example.com";
        HomePage homePage = PageFactory.initElements(driver, HomePage.class);
        driver.get(baseUrl);

        // Act
        boolean isPageLoaded = homePage.isPageDisplayed();

        // Assert
        assertTrue(isPageLoaded, "Home page failed to load in " + env + " environment");
    }
}
```

### Пояснения
- **`@Parameters("env")`**: Получает значение параметра `env` из `testng.xml`.
- **Гибкость**: Параметр `env` позволяет менять поведение теста в зависимости от окружения.
- **Логирование**: Используйте SLF4J для записи значений параметров.

## Использование слушателей (listeners)

Слушатели в TestNG позволяют перехватывать события, такие как начало теста, успех или провал. Они полезны для логирования, кастомной обработки ошибок или интеграции с Allure.

### Пример кода (Кастомный слушатель)

```java
import io.qameta.allure.Attachment;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testng.ITestContext;
import org.testng.ITestListener;
import org.testng.ITestResult;

public class CustomTestListener implements ITestListener {
    private static final Logger logger = LoggerFactory.getLogger(CustomTestListener.class);

    @Override
    public void onTestStart(ITestResult result) {
        logger.info("Starting test: {}", result.getMethod().getMethodName());
    }

    @Override
    public void onTestSuccess(ITestResult result) {
        logger.info("Test succeeded: {}", result.getMethod().getMethodName());
    }

    @Override
    public void onTestFailure(ITestResult result) {
        logger.error("Test failed: {}, cause: {}", result.getMethod().getMethodName(), result.getThrowable().getMessage());
        attachScreenshot(); // Пример прикрепления скриншота для UI-тестов
    }

    @Attachment(value = "Screenshot", type = "image/png")
    private byte[] attachScreenshot() {
        // Логика создания скриншота (например, с использованием Selenium)
        return new byte[0]; // Заглушка
    }
}
```

### Пояснения
- **Слушатель**: Реализует интерфейс `ITestListener` для обработки событий.
- **Логирование**: Использует SLF4J для записи событий.
- **Allure**: Метод `attachScreenshot` прикрепляет скриншот к отчёту при провале теста.

### Подключение слушателя
Добавьте слушатель в `testng.xml` (см. пример выше) или используйте аннотацию `@Listeners`:

```java
import org.testng.annotations.Listeners;

@Listeners(CustomTestListener.class)
public class ListenerTest extends BaseTest {
    @Test(groups = {"ui", "smoke"})
    public void testWithListener() {
        // Тест
    }
}
```

## Параллельное выполнение с `parallel` и `thread-count`

Атрибуты `parallel` и `thread-count` в `<suite>` или `<test>` позволяют запускать тесты параллельно, ускоряя выполнение.

- **Режимы `parallel`**:
  - `tests`: Параллельный запуск тестовых наборов (`<test>`).
  - `classes`: Параллельный запуск классов.
  - `methods`: Параллельный запуск методов.
- **`thread-count`**: Количество потоков для параллельного выполнения.

### Пример кода (Параллельное выполнение)

```java
import io.qameta.allure.Description;
import org.testng.annotations.Test;

public class ParallelTest extends BaseTest {

    @Test(groups = {"ui", "regression"})
    @Description("Проверка формы логина")
    public void testLoginForm() throws InterruptedException {
        // Arrange
        LoginPage loginPage = new LoginPage(driver);
        driver.get("https://example.com/login");

        // Act
        loginPage.enterCredentials("validUser", "validPass");
        boolean isLoggedIn = loginPage.submitLogin();

        // Assert
        assertTrue(isLoggedIn, "Login failed");
        Thread.sleep(1000); // Симуляция длительного теста
    }
}
```

### Обновлённый `testng.xml` для параллельного выполнения

```xml
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="ParallelTestSuite" parallel="methods" thread-count="4">
    <test name="UITests">
        <parameter name="env" value="qa"/>
        <groups>
            <run>
                <include name="ui"/>
            </run>
        </groups>
        <classes>
            <class name="com.example.tests.ui.ParallelTest"/>
        </classes>
    </test>
    <listeners>
        <listener class-name="com.example.tests.listeners.CustomTestListener"/>
    </listeners>
</suite>
```

### Пояснения
- **`parallel="methods"`**: Тестовые методы выполняются в отдельных потоках.
- **`thread-count="4"`**: Ограничивает количество потоков до 4.
- **Осторожность**: Убедитесь, что тесты независимы, чтобы избежать конфликтов (например, общего WebDriver).

### Лучшие практики
- **Изоляция**: Используйте отдельные экземпляры WebDriver для каждого потока.
- **Ограничение потоков**: Устанавливайте `thread-count` в зависимости от ресурсов CI/CD.
- **Логирование**: Логируйте выполнение тестов для отслеживания параллельных запусков.

## Интеграция с CI/CD

Для запуска тестов с `testng.xml` настройте Maven и TestNG в CI/CD.

### Пример настройки GitHub Actions

```yaml
name: Run Tests with testng.xml
on:
  push:
    branches:
      - main
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
      - name: Run tests
        run: mvn test -Dtestng.dtd.http=true -DsuiteXmlFile=testng.xml
      - name: Generate Allure report
        run: mvn allure:report
      - name: Publish Allure report
        uses: actions/upload-artifact@v4
        with:
          name: allure-report
          path: target/site/allure-maven-plugin
```

## Дополнительно

### Интеграция с Testcontainers

Testcontainers можно настроить в `@BeforeSuite` через `testng.xml`.

```java
import org.testcontainers.containers.GenericContainer;
import org.testng.annotations.BeforeSuite;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class ContainerConfigTest {
    private static GenericContainer<?> apiContainer;

    @BeforeSuite
    public void setupContainer() {
        apiContainer = new GenericContainer<>("my-api-image:latest")
                .withExposedPorts(8080);
        apiContainer.start();
        RestAssured.baseURI = "http://" + apiContainer.getHost() + ":" + apiContainer.getMappedPort(8080);
    }

    @Test(groups = {"api", "smoke"})
    public void testApiHealthCheck() {
        given()
                .when()
                .get("/api/health")
                .then()
                .statusCode(200)
                .body("status", equalTo("UP"));
    }
}
```

### Проверка качества кода

Используйте OWASP Dependency-Check для анализа зависимостей. Пример конфигурации в `pom.xml`:

```xml
<plugin>
    <groupId>org.owasp</groupId>
    <artifactId>dependency-check-maven</artifactId>
    <version>10.0.4</version>
    <executions>
        <execution>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

Запуск:
```bash
mvn dependency-check:check
```

### Отладка

Для отладки в IntelliJ IDEA:
1. Убедитесь, что `testng.xml` указан в TestNG Run/Debug Configuration.
2. Установите точки останова в тестах или слушателях.
3. Проверяйте логи SLF4J для анализа выполнения.

## Распространённые ошибки

- **Неправильные параметры**: Убедитесь, что имена параметров в `@Parameters` совпадают с `testng.xml`.
- **Конфликты при параллельном запуске**: Используйте изолированные ресурсы (например, отдельные WebDriver) для каждого потока.
- **Отсутствие слушателей**: Подключайте слушатели для обработки ошибок и логирования.

## Источники
- [TestNG XML Documentation](https://testng.org/doc/documentation-main.html#testng-xml)
- [Allure Framework](https://allurereport.org/docs/)
- [Testcontainers Documentation](https://testcontainers.org/)
- [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/)

---
[**← Назад к оглавлению**](../README.md)