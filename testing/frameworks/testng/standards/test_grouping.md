# Группировка тестов

Группировка тестов в TestNG с использованием аннотации `@Test(groups = {...})` позволяет логически организовать тесты, упрощая их запуск, управление и поддержку. Группы, такие как `smoke`, `regression`, `critical`, `ui` и `api`, помогают классифицировать тесты по их назначению и типу. В статье рассматривается использование групп, настройка их запуска через `testng.xml` или командную строку (CLI), а также интеграция с другими инструментами, дополняя предыдущие статьи новым контентом.

## Логическая организация тестов с `@Test(groups = {...})`

Аннотация `@Test(groups = {...})` позволяет присваивать тестам одну или несколько групп, которые затем используются для выборочного запуска. Это особенно полезно для разделения тестов по типу, приоритету или слою.

### Пример кода (TestNG с группами)

```java
import io.qameta.allure.Description;
import org.openqa.selenium.support.PageFactory;
import org.testng.annotations.Test;
import static org.testng.Assert.assertTrue;

public class GroupedTests extends BaseTest {

    @Test(groups = {"ui", "smoke", "critical"})
    @Description("Проверка загрузки главной страницы")
    public void testHomePageLoad() {
        // Arrange
        HomePage homePage = PageFactory.initElements(driver, HomePage.class);
        driver.get("https://example.com");

        // Act
        boolean isPageLoaded = homePage.isPageDisplayed();

        // Assert
        assertTrue(isPageLoaded, "Home page failed to load");
    }

    @Test(groups = {"api", "regression"})
    @Description("Проверка API эндпоинта здоровья")
    public void testApiHealthCheck() {
        // Arrange
        // Act
        given()
                .when()
                .get("/api/health")
                .then()
                .statusCode(200)
                .body("status", equalTo("UP"));
    }

    @Test(groups = {"ui", "regression"})
    @Description("Проверка формы логина")
    public void testLoginForm() {
        // Arrange
        LoginPage loginPage = PageFactory.initElements(driver, LoginPage.class);
        driver.get("https://example.com/login");

        // Act
        loginPage.enterCredentials("validUser", "validPass");
        boolean isLoggedIn = loginPage.submitLogin();

        // Assert
        assertTrue(isLoggedIn, "Login was not successful");
    }
}
```

### Пояснения
- **Группы**:
  - `smoke`: Быстрые тесты для проверки основного функционала.
  - `regression`: Полный набор тестов для проверки регрессий.
  - `critical`: Тесты критически важных функций.
  - `ui`: Тесты пользовательского интерфейса.
  - `api`: Тесты API.
- **Многогрупповость**: Тест может принадлежать нескольким группам (например, `ui` и `smoke`).
- **Allure**: Аннотация `@Description` улучшает читаемость отчётов.

### Лучшие практики
- **Осмысленные имена групп**: Используйте понятные имена, отражающие назначение (например, `smoke`, `auth`, `payment`).
- **Минимизация групп**: Избегайте избыточного числа групп, чтобы упростить управление.
- **Иерархия пакетов**: Организуйте тесты в пакеты (`tests.ui`, `tests.api`) для соответствия группам.
- **Документирование**: Используйте Allure для описания тестов и групп в отчётах.

## Запуск групп через `testng.xml`

Файл `testng.xml` позволяет настраивать запуск тестов по группам, включая или исключая их.

### Пример `testng.xml`

```xml
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="GroupedTestSuite">
    <test name="SmokeTests">
        <groups>
            <run>
                <include name="smoke"/>
            </run>
        </groups>
        <packages>
            <package name="tests.*"/>
        </packages>
    </test>
    <test name="RegressionTests">
        <groups>
            <run>
                <include name="regression"/>
                <exclude name="api"/>
            </run>
        </groups>
        <packages>
            <package name="tests.*"/>
        </packages>
    </test>
    <test name="CriticalTests">
        <groups>
            <run>
                <include name="critical"/>
            </run>
        </groups>
        <packages>
            <package name="tests.*"/>
        </packages>
    </test>
</suite>
```

### Пояснения
- **`<include>`**: Указывает группы для запуска (например, `smoke`).
- **`<exclude>`**: Исключает группы (например, `api` из регрессионных тестов).
- **`<packages>`**: Указывает пакеты, где искать тесты.

## Запуск групп через CLI

Для запуска тестов по группам через командную строку используйте Maven с параметром `-Dgroups`.

### Примеры команд
- Запуск smoke-тестов:
  ```bash
  mvn test -Dgroups=smoke
  ```
- Запуск регрессионных тестов, исключая API:
  ```bash
  mvn test -Dgroups=regression -DexcludedGroups=api
  ```
- Запуск критических тестов с указанием `testng.xml`:
  ```bash
  mvn test -Dtestng.dtd.http=true -DsuiteXmlFile=testng.xml
  ```

### Лучшие практики
- **Стабильность путей**: Убедитесь, что `testng.xml` находится в корне проекта или укажите правильный путь.
- **Логирование**: Настройте SLF4J с Logback для записи логов выполнения тестов.
- **Allure-отчёты**: Используйте команду `mvn allure:report` для генерации отчётов после запуска.

## Интеграция с CI/CD

Для запуска групп тестов в CI/CD настройте Maven и TestNG в Jenkins или GitHub Actions.

### Пример настройки GitHub Actions

```yaml
name: Run Grouped Tests
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
      - name: Run smoke tests
        run: mvn test -Dgroups=smoke
      - name: Run regression tests
        run: mvn test -Dgroups=regression -DexcludedGroups=api
      - name: Generate Allure report
        run: mvn allure:report
      - name: Publish Allure report
        uses: actions/upload-artifact@v4
        with:
          name: allure-report
          path: target/site/allure-maven-plugin
```

### Пример настройки Jenkins

1. Создайте проект в Jenkins.
2. Настройте SCM (Git) для клонирования репозитория.
3. Добавьте шаги сборки:
   ```bash
   mvn test -Dgroups=smoke
   mvn test -Dgroups=regression
   mvn allure:report
   ```
4. Установите плагин Allure Jenkins для публикации отчётов.

## Дополнительно

### Интеграция с Allure

Allure улучшает визуализацию групп в отчётах, позволяя фильтровать тесты по группам.

```java
import io.qameta.allure.Feature;
import io.qameta.allure.Story;
import org.testng.annotations.Test;

public class AllureGroupedTests extends BaseTest {

    @Test(groups = {"ui", "smoke", "critical"})
    @Feature("UI Testing")
    @Story("Home Page")
    @Description("Проверка загрузки главной страницы")
    public void testHomePageLoad() {
        // Arrange
        HomePage homePage = PageFactory.initElements(driver, HomePage.class);
        driver.get("https://example.com");

        // Act
        boolean isPageLoaded = homePage.isPageDisplayed();

        // Assert
        assertTrue(isPageLoaded, "Home page failed to load");
    }
}
```

Добавьте зависимость в `pom.xml`:

```xml
<dependency>
    <groupId>io.qameta.allure</groupId>
    <artifactId>allure-testng</artifactId>
    <version>2.27.0</version>
</dependency>
```

### Использование Testcontainers для API-тестов

Testcontainers можно использовать для запуска сервисов, необходимых для `api`-группы.

```java
import io.restassured.RestAssured;
import org.testcontainers.containers.GenericContainer;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

public class ApiGroupTest extends BaseTest {
    private GenericContainer<?> apiContainer;

    @BeforeMethod
    @Override
    public void setUp() {
        apiContainer = new GenericContainer<>("my-api-image:latest")
                .withExposedPorts(8080);
        apiContainer.start();
        RestAssured.baseURI = "http://" + apiContainer.getHost() + ":" + apiContainer.getMappedPort(8080);
    }

    @Test(groups = {"api", "smoke"})
    @Description("Проверка эндпоинта здоровья")
    public void testApiHealthCheck() {
        // Arrange
        // Act
        given()
                .when()
                .get("/api/health")
                .then()
                .statusCode(200)
                .body("status", equalTo("UP"));
    }

    @AfterMethod
    @Override
    public void tearDown() {
        if (apiContainer != null) {
            apiContainer.stop();
        }
        super.tearDown();
    }
}
```

### Проверка качества кода

Используйте OWASP Dependency-Check для анализа зависимостей в тестах. Пример конфигурации в `pom.xml`:

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

Запуск проверки:
```bash
mvn dependency-check:check
```

### Отладка

Для отладки групп в IntelliJ IDEA:
1. Настройте TestNG Run/Debug Configuration с указанием групп (например, `smoke`).
2. Установите точки останова в тестовых методах.
3. Используйте логи SLF4J для отслеживания выполнения.

## Распространённые ошибки

- **Неправильные имена групп**: Убедитесь, что группы в `testng.xml` и тестах совпадают.
- **Избыточные группы**: Избегайте создания лишних групп, чтобы не усложнять конфигурацию.
- **Пропуск тестов**: Если тест не входит в указанную группу, он не запустится; проверяйте конфигурацию `testng.xml`.

## Источники
- [TestNG Groups Documentation](https://testng.org/doc/documentation-main.html#test-groups)
- [Allure Framework](https://allurereport.org/docs/)
- [Testcontainers Documentation](https://testcontainers.org/)
- [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/)

---
[**← Назад к оглавлению**](../README.md)