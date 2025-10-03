# Продвинутое управление зависимостями в тестах

Управление зависимостями в тестах с использованием `dependsOnMethods` и `dependsOnGroups` в TestNG позволяет строить сложные цепочки выполнения, но требует осторожного подхода, чтобы избежать избыточности и обеспечить устойчивость. Эта статья дополняет базовые концепции, рассмотренные ранее, и фокусируется на продвинутых сценариях: комбинирование зависимостей с параметризацией, использование мягких зависимостей, интеграция с Testcontainers и Allure TestOps, а также обработка сложных случаев падения зависимых тестов.

## Комбинирование зависимостей с параметризацией

Зависимости можно комбинировать с `@DataProvider` для создания гибких тестовых сценариев, где тесты зависят от успешного выполнения подготовительных шагов с разными входными данными.

### Пример кода (TestNG с зависимостями и `@DataProvider`)

```java
import org.testng.annotations.DataProvider;
import org.testng.annotations.Test;
import static org.testng.Assert.assertTrue;

public class ParameterizedDependencyTest {
    @Test(groups = "setup")
    @Description("Инициализация тестовых данных")
    public void setupTestData() {
        // Arrange
        boolean isDataPrepared = prepareData();
        // Assert
        assertTrue(isDataPrepared, "Test data setup failed");
    }

    @Test(dependsOnMethods = "setupTestData", dataProvider = "userData")
    @Description("Проверка создания пользователя с разными данными")
    public void testUserCreation(String username, String role) {
        // Arrange
        // Act
        boolean isUserCreated = createUser(username, role);
        // Assert
        assertTrue(isUserCreated, "User creation failed for username: " + username);
    }

    @DataProvider(name = "userData")
    public Object[][] provideUserData() {
        return new Object[][] {
                {"user1", "admin"},
                {"user2", "guest"},
                {"user3", "editor"}
        };
    }

    private boolean prepareData() {
        // Логика подготовки данных
        return true;
    }

    private boolean createUser(String username, String role) {
        // Логика создания пользователя
        return true;
    }
}
```

### Пояснения
- **Зависимость**: Тест `testUserCreation` зависит от успешного выполнения `setupTestData`.
- **Параметризация**: `@DataProvider` предоставляет разные комбинации данных для `testUserCreation`.
- **Лучшие практики**:
  - Используйте осмысленные имена для методов и провайдеров данных.
  - Логируйте параметры в тестах для удобства отладки.

## Мягкие зависимости (Soft Dependencies)

TestNG поддерживает мягкие зависимости через атрибут `alwaysRun = true`, позволяя выполнять тесты даже при падении зависимостей. Это полезно для диагностики или частичного выполнения тестов.

### Пример кода (TestNG с мягкими зависимостями)

```java
import io.qameta.allure.Description;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testng.annotations.Test;
import static org.testng.Assert.assertTrue;

public class SoftDependencyTest {
    private static final Logger logger = LoggerFactory.getLogger(SoftDependencyTest.class);

    @Test(groups = "init")
    @Description("Инициализация сервиса")
    public void initService() {
        logger.info("Initializing service");
        boolean isInitialized = initializeService();
        assertTrue(isInitialized, "Service initialization failed");
    }

    @Test(dependsOnGroups = "init", alwaysRun = true)
    @Description("Проверка функциональности даже при сбое инициализации")
    public void testFeatureWithSoftDependency() {
        logger.info("Running feature test with soft dependency");
        // Act
        String result = performFeatureAction();
        // Assert
        assertTrue(result.contains("success"), "Feature action failed");
    }

    private boolean initializeService() {
        // Логика инициализации (для примера возвращаем false)
        return false;
    }

    private String performFeatureAction() {
        // Логика действия
        return "success";
    }
}
```

### Пояснения
- **`alwaysRun = true`**: Тест `testFeatureWithSoftDependency` выполнится даже при падении группы `init`.
- **Логирование**: SLF4J с Logback помогает отслеживать выполнение тестов.
- **Использование**: Применяйте мягкие зависимости для диагностических тестов или когда падение зависимости не критично.

## Интеграция с Testcontainers

Testcontainers можно использовать для настройки зависимостей, например, запуск базы данных или сервиса перед выполнением тестов.

### Пример кода (TestNG с Testcontainers)

```java
import org.testcontainers.containers.PostgreSQLContainer;
import org.testng.annotations.Test;
import static org.testng.Assert.assertTrue;

public class ContainerDependencyTest {
    private static final PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16")
            .withDatabaseName("testdb")
            .withUsername("user")
            .withPassword("pass");

    @Test(groups = "db-setup")
    @Description("Запуск контейнера базы данных")
    public void setupDatabase() {
        postgres.start();
        boolean isRunning = postgres.isRunning();
        assertTrue(isRunning, "Database container failed to start");
    }

    @Test(dependsOnGroups = "db-setup")
    @Description("Проверка запроса к базе данных")
    public void testDatabaseQuery() {
        // Arrange
        String jdbcUrl = postgres.getJdbcUrl();
        // Act
        String result = executeQuery(jdbcUrl);
        // Assert
        assertTrue(result.contains("success"), "Database query failed");
    }

    private String executeQuery(String jdbcUrl) {
        // Логика выполнения запроса
        return "success";
    }
}
```

### Пояснения
- **Зависимость**: Тест `testDatabaseQuery` зависит от группы `db-setup`, обеспечивая запуск контейнера.
- **Testcontainers**: Управляет жизненным циклом контейнера, минимизируя ручную настройку.

## Интеграция с Allure TestOps

Allure TestOps позволяет управлять зависимостями тестов на уровне тест-менеджмента, связывая тесты с требованиями и отслеживая их выполнение.

### Пример настройки
1. Настройте Allure TestOps для интеграции с TestNG через плагин `allure-testng`.
2. Добавьте аннотации `@Issue` и `@TmsLink` для связи тестов с задачами:
   ```java
   import io.qameta.allure.Issue;
   import io.qameta.allure.TmsLink;
   import org.testng.annotations.Test;

   public class AllureTestOpsDependencyTest {

       @Test(groups = "setup")
       @Issue("BUG-123")
       @TmsLink("TMS-456")
       public void setupEnvironment() {
           // Логика настройки
           assertTrue(true, "Environment setup failed");
       }

       @Test(dependsOnMethods = "setupEnvironment")
       @Issue("BUG-124")
       @TmsLink("TMS-457")
       public void testFeature() {
           // Логика теста
           assertTrue(true, "Feature test failed");
       }
   }
   ```
3. Настройте Jenkins или GitHub Actions для отправки результатов в Allure TestOps:
   ```bash
   mvn allure:report
   allurectl upload target/allure-results
   ```

## Обработка сложных случаев падения зависимых тестов

Для обработки падений зависимых тестов используйте механизмы повторных попыток (Retry) и кастомные слушатели TestNG.

### Пример кода (TestNG с кастомным слушателем)

```java
import org.testng.IInvokedMethod;
import org.testng.IInvokedMethodListener;
import org.testng.ITestResult;
import org.testng.annotations.Test;
import static org.testng.Assert.assertTrue;

public class DependencyListenerTest implements IInvokedMethodListener {
    private static final Logger logger = LoggerFactory.getLogger(DependencyListenerTest.class);

    @Override
    public void beforeInvocation(IInvokedMethod method, ITestResult testResult) {
        if (method.isTestMethod() && method.getTestMethod().getGroupsDependedUpon().length > 0) {
            logger.info("Test {} depends on groups: {}", method.getTestMethod().getMethodName(),
                    String.join(", ", method.getTestMethod().getGroupsDependedUpon()));
        }
    }

    @Override
    public void afterInvocation(IInvokedMethod method, ITestResult testResult) {
        if (testResult.getStatus() == ITestResult.SKIP) {
            logger.warn("Test {} was skipped due to dependency failure", method.getTestMethod().getMethodName());
        }
    }

    @Test(groups = "setup")
    public void setupEnvironment() {
        // Логика настройки (для примера падает)
        assertTrue(false, "Environment setup failed");
    }

    @Test(dependsOnGroups = "setup")
    public void testFeature() {
        // Логика теста
        assertTrue(true, "Feature test failed");
    }
}
```

### Пояснения
- **Слушатель**: Логирует информацию о зависимостях и пропущенных тестах.
- **Конфигурация**: Добавьте слушатель в `testng.xml`:
  ```xml
  <listeners>
      <listener class-name="DependencyListenerTest"/>
  </listeners>
  ```

## Интеграция с CI/CD

Для запуска тестов с зависимостями в CI/CD используйте Maven с указанием групп или методов.

### Пример настройки GitHub Actions

```yaml
name: Run Dependent Tests
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
      - name: Run tests with dependencies
        run: mvn test -Dgroups=setup,feature -Dtestng.dtd.http=true
      - name: Generate Allure report
        run: mvn allure:report
      - name: Publish Allure report
        uses: actions/upload-artifact@v4
        with:
          name: allure-report
          path: target/site/allure-maven-plugin
```

## Дополнительно

### Проверка качества кода

Используйте SpotBugs для анализа зависимостей в коде. Пример конфигурации в `pom.xml`:

```xml
<plugin>
    <groupId>com.github.spotbugs</groupId>
    <artifactId>spotbugs-maven-plugin</artifactId>
    <version>4.8.6.4</version>
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
mvn spotbugs:check
```

### Отладка

Для отладки в IntelliJ IDEA:
1. Установите точки останова в зависимых методах или слушателях.
2. Настройте TestNG Run/Debug Configuration с указанием групп или `testng.xml`.
3. Используйте логи SLF4J для анализа выполнения.

## Распространённые ошибки

- **Избыточные зависимости**: Используйте зависимости только там, где они необходимы, чтобы избежать усложнения структуры.
- **Ошибки в группах**: Убедитесь, что имена групп в `dependsOnGroups` совпадают с реальными группами.
- **Пропуск тестов без логов**: Настройте слушатели или Allure для фиксации причин пропуска.

## Источники
- [TestNG Dependencies Documentation](https://testng.org/doc/documentation-main.html#dependencies)
- [Allure TestOps Documentation](https://allurereport.org/docs/testops/)
- [Testcontainers Documentation](https://testcontainers.org/)
- [SpotBugs Documentation](https://spotbugs.github.io/)

---
[**← Назад к оглавлению**](../README.md)