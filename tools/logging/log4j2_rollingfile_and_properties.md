
# Настройка ротации логов и логирования ошибок с Log4j2 в автотестах

Логирование в автотестах помогает отслеживать выполнение тестов, фиксировать ошибки и анализировать данные, но со временем лог-файлы могут занимать много места. Ротация логов и выделение ошибок в отдельный файл решают эту проблему, обеспечивая управляемость и удобство анализа. SLF4J (Simple Logging Facade for Java) с Log4j2 позволяет гибко настраивать логирование, включая ротацию файлов и фильтрацию сообщений по уровням. Эта статья описывает, как настроить ротацию логов с помощью `RollingFileAppender`, выделить ошибки в отдельный файл и использовать `log4j2.properties` вместо XML для конфигурации в автотестах.

## Зачем нужна ротация логов и отдельный файл для ошибок

- **Ротация логов**: Ограничивает размер лог-файлов, автоматически архивируя или удаляя старые, чтобы избежать переполнения диска.
- **Логирование ошибок в отдельный файл**: Упрощает анализ сбоев, позволяя быстро находить критические проблемы без просмотра всех логов.
- **Гибкость конфигурации**: Формат `log4j2.properties` проще для быстрого редактирования и поддерживает те же функции, что и XML.
- **Удобство в CI/CD**: Ротируемые логи и выделенные файлы ошибок легко интегрируются в системы непрерывной интеграции.

## Подключение SLF4J и Log4j2 в Maven

Для работы с Log4j2 через SLF4J необходимо добавить зависимости в проект.

### Шаг 1: Добавление зависимостей в `pom.xml`

- **`slf4j-api`**: Интерфейс для логирования.
- **`log4j-slf4j2-impl`**: Связывает SLF4J с Log4j2.
- **`log4j-api`**: Программный интерфейс Log4j2.
- **`log4j-core`**: Реализует функциональность логирования, включая ротацию.

### Шаг 2: Настройка ротации и логов ошибок через `log4j2.properties`

Создайте файл `log4j2.properties` в директории `src/test/resources` для конфигурации ротации логов и выделения ошибок в отдельный файл.

```properties
# Внутренний уровень логирования Log4j2
status = INFO

# Определение аппендеров
appenders = Console, RollingFile, ErrorFile

# Консольный аппендер
appender.Console.type = Console
appender.Console.name = Console
appender.Console.target = SYSTEM_OUT
appender.Console.layout.type = PatternLayout
appender.Console.layout.pattern = %d{yyyy-MM-dd HH:mm:ss} [%t] %-5level %c{1} - %msg%n

# RollingFile аппендер для ротации логов
appender.RollingFile.type = RollingFile
appender.RollingFile.name = RollingFile
appender.RollingFile.fileName = logs/tests.log
appender.RollingFile.filePattern = logs/tests-%d{yyyy-MM-dd}-%i.log.gz
appender.RollingFile.layout.type = PatternLayout
appender.RollingFile.layout.pattern = %d{yyyy-MM-dd HH:mm:ss} [%t] %-5level %c{1} - %msg%n
appender.RollingFile.policies.type = Policies
appender.RollingFile.policies.time.type = TimeBasedTriggeringPolicy
appender.RollingFile.policies.time.interval = 1
appender.RollingFile.policies.time.modulate = true
appender.RollingFile.policies.size.type = SizeBasedTriggeringPolicy
appender.RollingFile.policies.size.size = 10MB
appender.RollingFile.strategy.type = DefaultRolloverStrategy
appender.RollingFile.strategy.max = 5

# Аппендер для ошибок
appender.ErrorFile.type = File
appender.ErrorFile.name = ErrorFile
appender.ErrorFile.fileName = logs/errors.log
appender.ErrorFile.layout.type = PatternLayout
appender.ErrorFile.layout.pattern = %d{yyyy-MM-dd HH:mm:ss} [%t] %-5level %c{1} - %msg%n
appender.ErrorFile.filter.threshold.type = ThresholdFilter
appender.ErrorFile.filter.threshold.level = ERROR

# Логгеры
logger.ui.name = com.example.ui
logger.ui.level = info
logger.ui.additivity = false
logger.ui.appenderRefs = RollingFile, Console

logger.api.name = com.example.api
logger.api.level = info
logger.api.additivity = false
logger.api.appenderRefs = RollingFile, Console

logger.error.name = com.example
logger.error.level = error
logger.error.additivity = false
logger.error.appenderRefs = ErrorFile

# Корневой логгер
rootLogger.level = info
rootLogger.appenderRefs = Console
```

### Детальный разбор конфигурации

#### Общие настройки
- **status**: Уровень логирования для внутренних сообщений Log4j2 (`INFO` для стандартной работы, `DEBUG` для отладки конфигурации).
  ```properties
  status = INFO
  ```

#### Аппендеры
Аппендеры определяют, куда выводятся логи.

1. **Console**:
    - Выводит логи в консоль (`SYSTEM_OUT`).
    - Формат: дата, время, поток, уровень, имя класса, сообщение.
   ```properties
   appender.Console.type = Console
   appender.Console.name = Console
   appender.Console.target = SYSTEM_OUT
   appender.Console.layout.type = PatternLayout
   appender.Console.layout.pattern = %d{yyyy-MM-dd HH:mm:ss} [%t] %-5level %c{1} - %msg%n
   ```

2. **RollingFile**:
    - Ротирует логи, сохраняя их в `logs/tests.log`.
    - **filePattern**: Шаблон для архивных файлов (`logs/tests-%d{yyyy-MM-dd}-%i.log.gz` создаёт файлы типа `tests-2025-10-03-1.log.gz`).
    - **TimeBasedTriggeringPolicy**: Запускает ротацию ежедневно (`interval = 1`).
    - **SizeBasedTriggeringPolicy**: Запускает ротацию при размере файла 10 МБ.
    - **DefaultRolloverStrategy**: Хранит до 5 архивных файлов (`max = 5`).
   ```properties
   appender.RollingFile.type = RollingFile
   appender.RollingFile.name = RollingFile
   appender.RollingFile.fileName = logs/tests.log
   appender.RollingFile.filePattern = logs/tests-%d{yyyy-MM-dd}-%i.log.gz
   appender.RollingFile.policies.time.type = TimeBasedTriggeringPolicy
   appender.RollingFile.policies.time.interval = 1
   appender.RollingFile.policies.size.size = 10MB
   appender.RollingFile.strategy.max = 5
   ```

3. **ErrorFile**:
    - Пишет только сообщения уровня `ERROR` в `logs/errors.log`.
    - **ThresholdFilter**: Фильтрует сообщения, пропуская только `ERROR` и выше.
   ```properties
   appender.ErrorFile.type = File
   appender.ErrorFile.name = ErrorFile
   appender.ErrorFile.fileName = logs/errors.log
   appender.ErrorFile.filter.threshold.type = ThresholdFilter
   appender.ErrorFile.filter.threshold.level = ERROR
   ```

#### Логгеры
Логгеры определяют, какие классы используют какие аппендеры.

1. **UI Logger**:
    - Для пакета `com.example.ui`, уровень `info`.
    - Пишет в `RollingFile` и `Console`.
    - `additivity = false` предотвращает дублирование в корневом логгере.
   ```properties
   logger.ui.name = com.example.ui
   logger.ui.level = info
   logger.ui.additivity = false
   logger.ui.appenderRefs = RollingFile, Console
   ```

2. **API Logger**:
    - Для пакета `com.example.api`, уровень `info`.
    - Пишет в `RollingFile` и `Console`.
   ```properties
   logger.api.name = com.example.api
   logger.api.level = info
   logger.api.additivity = false
   logger.api.appenderRefs = RollingFile, Console
   ```

3. **Error Logger**:
    - Для пакета `com.example`, уровень `error`.
    - Пишет только в `ErrorFile`.
   ```properties
   logger.error.name = com.example
   logger.error.level = error
   logger.error.additivity = false
   logger.error.appenderRefs = ErrorFile
   ```

4. **Root Logger**:
    - Для всех классов, не подпадающих под другие логгеры, уровень `info`.
    - Пишет только в `Console`.
   ```properties
   rootLogger.level = info
   rootLogger.appenderRefs = Console
   ```

### Шаг 3: Использование логирования в коде

Добавьте SLF4J-логгер в тестовые классы для фиксации шагов и ошибок.

#### Пример UI-теста (в пакете `com.example.ui`)

```java
package com.example.ui;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import io.github.bonigarcia.wdm.WebDriverManager;
import static org.assertj.core.api.Assertions.assertThat;

public class LoginTest {
    private static final Logger logger = LoggerFactory.getLogger(LoginTest.class);
    private WebDriver driver;

    @BeforeEach
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        logger.info("Инициализация WebDriver");
        driver.get("https://example.com/login");
        logger.debug("Открыта страница логина: {}", driver.getCurrentUrl());
    }

    @Test
    void testLogin() {
        String username = "testuser";
        String password = "wrongpassword";
        logger.debug("Подготовка данных: username={}, password={}", username, password);
        try {
            login(username, password);
            logger.info("Проверка перехода на страницу dashboard");
            assertThat(driver.getCurrentUrl()).contains("/dashboard");
        } catch (Exception e) {
            logger.error("Ошибка логина: {}", e.getMessage());
            throw e;
        }
    }

    private void login(String username, String password) {
        logger.debug("Ввод логина: {}", username);
        // Эмуляция ввода логина и пароля
        logger.info("Логин выполнен");
    }

    @AfterEach
    void tearDown() {
        logger.info("Закрытие WebDriver");
        driver.quit();
    }
}
```

- Логи пишутся в `logs/tests.log` и консоль, ошибки — в `logs/errors.log`.

#### Пример API-теста (в пакете `com.example.api`)

```java
package com.example.api;

import io.restassured.RestAssured;
import io.restassured.response.Response;
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import static io.restassured.RestAssured.given;
import static org.assertj.core.api.Assertions.assertThat;

public class ApiTest {
    private static final Logger logger = LoggerFactory.getLogger(ApiTest.class);

    @Test
    void testApi() {
        RestAssured.baseURI = "https://api.example.com";
        logger.info("Настройка базового URI: {}", RestAssured.baseURI);
        try {
            Response response = getUsers();
            logger.debug("Получен ответ API: {}", response.asString());
            logger.info("Проверка статуса ответа: {}", response.getStatusCode());
            assertThat(response.getStatusCode()).isEqualTo(200);
        } catch (Exception e) {
            logger.error("Ошибка API-запроса: {}", e.getMessage());
            throw e;
        }
    }

    private Response getUsers() {
        logger.debug("Отправка GET-запроса на /users");
        Response response = given()
            .when()
            .get("/users")
            .then()
            .extract()
            .response();
        logger.debug("Ответ получен: {}", response.getStatusCode());
        return response;
    }
}
```

- Логи пишутся в `logs/tests.log` и консоль, ошибки — в `logs/errors.log`.

## JSON-логирование для ротации

Для структурированного вывода добавьте зависимость:

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.15.2</version>
</dependency>
```

Обновите `log4j2.properties`:

```properties
appender.JsonRollingFile.type = RollingFile
appender.JsonRollingFile.name = JsonRollingFile
appender.JsonRollingFile.fileName = logs/tests.json
appender.JsonRollingFile.filePattern = logs/tests-%d{yyyy-MM-dd}-%i.json.gz
appender.JsonRollingFile.layout.type = JsonLayout
appender.JsonRollingFile.layout.compact = true
appender.JsonRollingFile.layout.eventEol = true
appender.JsonRollingFile.policies.type = Policies
appender.JsonRollingFile.policies.time.type = TimeBasedTriggeringPolicy
appender.JsonRollingFile.policies.time.interval = 1
appender.JsonRollingFile.policies.size.type = SizeBasedTriggeringPolicy
appender.JsonRollingFile.policies.size.size = 10MB
appender.JsonRollingFile.strategy.type = DefaultRolloverStrategy
appender.JsonRollingFile.strategy.max = 5

logger.ui.appenderRefs = RollingFile, Console, JsonRollingFile
logger.api.appenderRefs = RollingFile, Console, JsonRollingFile
```

- Логи в JSON ротируются аналогично текстовым логам.

## Интеграция с Allure

Логи можно прикреплять к отчётам Allure:

```java
package com.example.utils;

import io.qameta.allure.Attachment;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class AllureIntegration {
    private static final Logger logger = LoggerFactory.getLogger(AllureIntegration.class);

    @Attachment(value = "Log", type = "text/plain")
    public String attachLog(String message) {
        logger.debug("Прикрепление лога к Allure: {}", message);
        return message;
    }
}
```

## Лучшие практики

- **Ротация**: Настраивайте `SizeBasedTriggeringPolicy` и `TimeBasedTriggeringPolicy` для контроля размера и частоты ротации.
- **Ошибки**: Используйте отдельный файл для `ERROR` с `ThresholdFilter` для быстрого анализа сбоев.
- **Формат `.properties`**: Используйте для простых проектов, XML — для сложных конфигураций.
- **CI/CD**: Сохраняйте ротируемые логи и `errors.log` как артефакты.
- **Производительность**: Отключайте `DEBUG` в CI/CD для ускорения тестов.

## Источники
- [SLF4J Documentation](http://www.slf4j.org/manual.html)
- [Log4j2 Documentation](https://logging.apache.org/log4j/2.x/manual/index.html)
- [Log4j2 Configuration](https://logging.apache.org/log4j/2.x/manual/configuration.html)
- [Log4j2 RollingFileAppender](https://logging.apache.org/log4j/2.x/manual/appenders.html#RollingFileAppender)

---

[**← Назад к оглавлению**](README.md)