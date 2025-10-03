# Управление зависимостями в тестах

Управление зависимостями в тестах позволяет задавать порядок выполнения тестов и управлять их выполнением в случае падения зависимых тестов. В TestNG для этого используются аннотации `dependsOnMethods` и `dependsOnGroups`. В статье рассматриваются их использование, лучшие практики, обработка падений зависимых тестов и интеграция с другими инструментами, такими как Allure и CI/CD, с акцентом на новый контент, дополняющий предыдущие статьи.

## Использование `dependsOnMethods` и `dependsOnGroups`

Аннотации `dependsOnMethods` и `dependsOnGroups` в TestNG позволяют задавать зависимости между тестами или группами тестов. Тест, зависящий от другого метода или группы, выполняется только в случае успешного завершения зависимостей.

### Пример кода (TestNG с `dependsOnMethods`)

```java
import org.testng.annotations.Test;
import static org.testng.Assert.assertTrue;

public class DependencyTest {

    @Test
    public void setupEnvironment() {
        // Arrange: Настройка окружения
        boolean isSetupSuccessful = initializeEnvironment();
        // Assert
        assertTrue(isSetupSuccessful, "Environment setup failed");
    }

    @Test(dependsOnMethods = "setupEnvironment")
    public void testApiCall() {
        // Arrange: Подготовка данных для API
        String response = makeApiCall();
        // Assert
        assertTrue(response.contains("success"), "API call failed");
    }

    private boolean initializeEnvironment() {
        // Логика инициализации окружения
        return true;
    }

    private String makeApiCall() {
        // Логика вызова API
        return "success";
    }
}
```

В этом примере `testApiCall` выполняется только после успешного завершения `setupEnvironment`.

### Пример кода (TestNG с `dependsOnGroups`)

```java
import org.testng.annotations.Test;
import static org.testng.Assert.assertTrue;

public class GroupDependencyTest {

    @Test(groups = "init")
    public void initDatabase() {
        // Arrange: Инициализация базы данных
        boolean isInitialized = setupDatabase();
        // Assert
        assertTrue(isInitialized, "Database initialization failed");
    }

    @Test(groups = "init")
    public void initConfig() {
        // Arrange: Инициализация конфигурации
        boolean isConfigured = setupConfig();
        // Assert
        assertTrue(isConfigured, "Configuration setup failed");
    }

    @Test(dependsOnGroups = "init")
    public void testFeature() {
        // Arrange: Подготовка данных
        String result = performFeatureAction();
        // Assert
        assertTrue(result.contains("completed"), "Feature action failed");
    }

    private boolean setupDatabase() {
        // Логика настройки базы данных
        return true;
    }

    private boolean setupConfig() {
        // Логика настройки конфигурации
        return true;
    }

    private String performFeatureAction() {
        // Логика выполнения действия
        return "completed";
    }
}
```

Здесь `testFeature` зависит от группы `init`, включающей `initDatabase` и `initConfig`.

### Лучшие практики
- **Используйте зависимости только при необходимости**: Зависимости оправданы, если тест действительно требует предварительного выполнения другого теста (например, настройка окружения или создание данных).
- **Избегайте избыточности**: Не создавайте сложные цепочки зависимостей, которые усложняют отладку и поддержку.
- **Группы для масштабируемости**: Используйте `dependsOnGroups` для логически связанных тестов, чтобы упростить управление зависимостями.
- **Ясные имена методов и групп**: Называйте методы и группы так, чтобы их назначение было понятно (например, `init`, `auth`, `cleanup`).

## Обработка падений зависимых тестов

Если зависимый тест падает, TestNG автоматически помечает зависящие от него тесты как пропущенные (`SKIPPED`). Это предотвращает выполнение тестов, которые заведомо не пройдут из-за отсутствия необходимых условий. Однако важно правильно обрабатывать такие случаи, чтобы отчёты и логи были информативными.

### Пример обработки с Allure

Использование Allure для отображения зависимостей и причин пропуска тестов:

```java
import io.qameta.allure.Description;
import io.qameta.allure.Step;
import org.testng.annotations.Test;
import static org.testng.Assert.assertTrue;

public class DependencyWithAllureTest {

    @Test
    @Description("Инициализация окружения")
    public void setupEnvironment() {
        // Arrange
        boolean isSetupSuccessful = initializeEnvironment();
        // Assert
        assertTrue(isSetupSuccessful, "Environment setup failed");
    }

    @Test(dependsOnMethods = "setupEnvironment")
    @Description("Тестирование функциональности после инициализации")
    public void testFeature() {
        // Arrange
        String result = performFeatureAction();
        // Assert
        assertTrue(result.contains("success"), "Feature action failed");
    }

    @Step("Инициализация окружения")
    private boolean initializeEnvironment() {
        // Логика инициализации
        return true; // Для демонстрации можно заменить на false, чтобы эмулировать сбой
    }

    @Step("Выполнение действия функциональности")
    private String performFeatureAction() {
        // Логика действия
        return "success";
    }
}
```

Если `setupEnvironment` завершится неудачно, Allure отобразит `testFeature` как пропущенный с указанием причины (зависимость не выполнена).

### Логирование с SLF4J и Logback

Добавьте логирование для отслеживания выполнения зависимостей. Зависимости в `pom.xml`:

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>2.0.16</version>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.5.12</version>
</dependency>
```

Пример с логированием:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testng.annotations.Test;
import static org.testng.Assert.assertTrue;

public class DependencyWithLoggingTest {
    private static final Logger logger = LoggerFactory.getLogger(DependencyWithLoggingTest.class);

    @Test
    public void setupEnvironment() {
        logger.info("Starting environment setup");
        boolean isSetupSuccessful = initializeEnvironment();
        assertTrue(isSetupSuccessful, "Environment setup failed");
        logger.info("Environment setup completed");
    }

    @Test(dependsOnMethods = "setupEnvironment")
    public void testFeature() {
        logger.info("Starting feature test");
        String result = performFeatureAction();
        assertTrue(result.contains("success"), "Feature action failed");
        logger.info("Feature test completed");
    }

    private boolean initializeEnvironment() {
        // Логика инициализации
        return true;
    }

    private String performFeatureAction() {
        // Логика действия
        return "success";
    }
}
```

Логи помогут отследить, на каком этапе произошёл сбой, и упростят отладку.

## Интеграция с CI/CD

Для запуска тестов с зависимостями в CI/CD настройте Maven и убедитесь, что тесты выполняются в правильном порядке.

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
      - name: Run tests with TestNG
        run: mvn clean test -Dtestng.dtd.http=true
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
3. Добавьте шаг сборки: `Execute shell` с командами:
   ```bash
   mvn clean test
   mvn allure:report
   ```
4. Настройте публикацию Allure-отчётов через плагин Allure Jenkins.

## Дополнительно

### Использование Testcontainers для зависимостей

Testcontainers можно использовать для настройки окружения, от которого зависят тесты (например, запуск базы данных или сервиса). Пример:

```java
import org.testcontainers.containers.PostgreSQLContainer;
import org.testng.annotations.Test;
import static org.testng.Assert.assertTrue;

public class DatabaseDependencyTest {

    private static final PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16")
            .withDatabaseName("testdb")
            .withUsername("user")
            .withPassword("pass");

    @Test
    public void setupDatabase() {
        postgres.start();
        boolean isRunning = postgres.isRunning();
        assertTrue(isRunning, "Database container failed to start");
    }

    @Test(dependsOnMethods = "setupDatabase")
    public void testDatabaseQuery() {
        // Arrange: Подключение к базе
        String jdbcUrl = postgres.getJdbcUrl();
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

Добавьте зависимости в `pom.xml`:

```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <version>1.20.2</version>
    <scope>test</scope>
</dependency>
```

### Повторные попытки для зависимых тестов

Для борьбы с нестабильными тестами можно использовать механизм повторных попыток (Retry). Пример реализации:

```java
import org.testng.IRetryAnalyzer;
import org.testng.ITestResult;

public class RetryAnalyzer implements IRetryAnalyzer {
    private int retryCount = 0;
    private static final int MAX_RETRY_COUNT = 2;

    @Override
    public boolean retry(ITestResult result) {
        if (retryCount < MAX_RETRY_COUNT) {
            retryCount++;
            return true;
        }
        return false;
    }
}
```

Применение в тесте:

```java
import org.testng.annotations.Test;

public class RetryDependencyTest {

    @Test(retryAnalyzer = RetryAnalyzer.class)
    public void setupEnvironment() {
        // Логика настройки
        boolean isSetupSuccessful = initializeEnvironment();
        Assert.assertTrue(isSetupSuccessful, "Environment setup failed");
    }

    @Test(dependsOnMethods = "setupEnvironment", retryAnalyzer = RetryAnalyzer.class)
    public void testFeature() {
        // Логика теста
        String result = performFeatureAction();
        Assert.assertTrue(result.contains("success"), "Feature action failed");
    }

    private boolean initializeEnvironment() {
        // Логика инициализации
        return true;
    }

    private String performFeatureAction() {
        // Логика действия
        return "success";
    }
}
```

### Отладка зависимостей

Для отладки в IntelliJ IDEA:
1. Установите точки останова в методах, помеченных `@Test`, и в зависимых методах.
2. Запустите тесты в режиме Debug с флагом `-Dtestng.dtd.http=true`.
3. Используйте TestNG Run/Debug Configuration для запуска отдельных тестов или групп.

## Распространённые ошибки

- **Циклические зависимости**: Избегайте ситуаций, когда тесты зависят друг от друга циклически, это приведёт к ошибке TestNG.
- **Слишком сложные цепочки**: Длинные цепочки зависимостей усложняют поддержку и отладку.
- **Игнорирование пропущенных тестов**: Проверяйте отчёты Allure, чтобы понять, почему тест был пропущен, и исправляйте первопричину.

## Источники
- [TestNG Dependencies Documentation](https://testng.org/doc/documentation-main.html#dependencies)
- [Allure Framework](https://allurereport.org/docs/)
- [Testcontainers Documentation](https://testcontainers.org/)
- [SLF4J Documentation](https://www.slf4j.org/)
- [Logback Documentation](https://logback.qos.ch/)

---
[**← Назад к оглавлению**](../README.md)