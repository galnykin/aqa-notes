# Настройка асинхронного логирования с Log4j2 в автотестах

Асинхронное логирование в автотестах повышает производительность, минимизируя задержки при записи логов, что особенно важно для больших тестовых наборов. SLF4J (Simple Logging Facade for Java) с Log4j2 позволяет настроить асинхронное логирование с помощью `AsyncLogger` или `AsyncAppender`, сохраняя гибкость и функциональность. Эта статья описывает, как настроить асинхронное логирование в Log4j2, используя конфигурацию `log4j2.properties`, для вывода логов в консоль и файл, а также объясняет преимущества и особенности использования в автотестах.

## Настройка асинхронного логирования

Log4j2 поддерживает два подхода к асинхронному логированию:
- **AsyncLogger**: Все логгеры работают асинхронно, минимизируя влияние на основной поток теста.
- **AsyncAppender**: Конкретные аппендеры (например, для файлов) обрабатываются асинхронно.

Для асинхронного логирования требуется библиотека `disruptor`, которая оптимизирует обработку логов в многопоточной среде.

### Шаг 1: Добавление зависимости для Disruptor

Для работы асинхронного логирования добавьте зависимость `disruptor` в `pom.xml`:

```xml
<dependency>
    <groupId>com.lmax</groupId>
    <artifactId>disruptor</artifactId>
    <version>3.4.4</version>
</dependency>
```

### Шаг 2: Настройка асинхронного логирования через `log4j2.properties`

Создайте файл `log4j2.properties` в директории `src/test/resources` для настройки асинхронного логирования.

```properties
# Внутренний уровень логирования Log4j2
status = INFO

# Включение асинхронных логгеров
asyncLoggerConfig = true

# Определение аппендеров
appenders = Console, AsyncFile

# Консольный аппендер
appender.Console.type = Console
appender.Console.name = Console
appender.Console.target = SYSTEM_OUT
appender.Console.layout.type = PatternLayout
appender.Console.layout.pattern = %d{yyyy-MM-dd HH:mm:ss} [%t] %-5level %c{1} - %msg%n

# Асинхронный файловый аппендер
appender.AsyncFile.type = Async
appender.AsyncFile.name = AsyncFile
appender.AsyncFile.appenderRef.ref = File
appender.AsyncFile.blocking = false

# Файловый аппендер для AsyncFile
appender.File.type = File
appender.File.name = File
appender.File.fileName = logs/tests.log
appender.File.layout.type = PatternLayout
appender.File.layout.pattern = %d{yyyy-MM-dd HH:mm:ss} [%t] %-5level %c{1} - %msg%n

# Логгеры
logger.tests.name = com.example.tests
logger.tests.level = debug
logger.tests.additivity = false
logger.tests.appenderRefs = Console, AsyncFile

# Корневой логгер
rootLogger.level = info
rootLogger.appenderRefs = Console
```

### Разбор конфигурации

#### Общие настройки
- **status**: Уровень для внутренних сообщений Log4j2 (`INFO` для стандартной работы, `DEBUG` для отладки).
  ```properties
  status = INFO
  ```
- **asyncLoggerConfig**: Включает асинхронное логирование для всех логгеров. Альтернативно можно использовать `AsyncAppender` без этой настройки.
  ```properties
  asyncLoggerConfig = true
  ```

#### Аппендеры
- **Console**: Выводит логи в консоль с форматом, включающим дату, время, поток, уровень, имя класса и сообщение.
  ```properties
  appender.Console.type = Console
  appender.Console.layout.pattern = %d{yyyy-MM-dd HH:mm:ss} [%t] %-5level %c{1} - %msg%n
  ```
- **AsyncFile**: Асинхронный аппендер, ссылающийся на файловый аппендер `File`. Параметр `blocking = false` минимизирует задержки.
  ```properties
  appender.AsyncFile.type = Async
  appender.AsyncFile.appenderRef.ref = File
  appender.AsyncFile.blocking = false
  ```
- **File**: Пишет логи в `logs/tests.log` с тем же форматом.
  ```properties
  appender.File.type = File
  appender.File.fileName = logs/tests.log
  ```

#### Логгеры
- **Tests Logger**: Для пакета `com.example.tests`, уровень `debug`, пишет в `Console` и `AsyncFile`.
  ```properties
  logger.tests.name = com.example.tests
  logger.tests.level = debug
  logger.tests.appenderRefs = Console, AsyncFile
  ```
- **Root Logger**: Для остальных классов, уровень `info`, пишет только в `Console`.
  ```properties
  rootLogger.level = info
  rootLogger.appenderRefs = Console
  ```

### Шаг 3: Использование в коде

Добавьте SLF4J-логгер в тестовые классы в пакете `com.example.tests`.

#### Пример UI-теста с Selenium

```java
package com.example.tests;

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
        String password = "password123";
        logger.debug("Подготовка данных: username={}, password={}", username, password);
        login(username, password);
        logger.info("Проверка перехода на страницу dashboard");
        assertThat(driver.getCurrentUrl()).contains("/dashboard");
    }

    private void login(String username, String password) {
        logger.debug("Ввод логина: {}", username);
        logger.info("Логин выполнен");
    }

    @AfterEach
    void tearDown() {
        logger.info("Закрытие WebDriver");
        driver.quit();
    }
}
```

#### Пример API-теста с RestAssured

```java
package com.example.tests;

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
        Response response = getUsers();
        logger.debug("Получен ответ API: {}", response.asString());
        logger.info("Проверка статуса ответа: {}", response.getStatusCode());
        assertThat(response.getStatusCode()).isEqualTo(200);
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

## Преимущества асинхронного логирования

- **Производительность**: Асинхронное логирование снижает задержки в основном потоке теста, что критично для больших тестовых наборов.
- **Гибкость**: SLF4J позволяет переключиться на другой логгер без изменения кода.
- **Многоканальный вывод**: Логи пишутся в консоль и файл (`logs/tests.log`).
- **Надёжность**: Disruptor минимизирует потери логов в многопоточной среде.

## JSON-логирование

Для вывода логов в JSON добавьте зависимость:

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.15.2</version>
</dependency>
```

Обновите `log4j2.properties`:

```properties
appender.JsonAsync.type = Async
appender.JsonAsync.name = JsonAsync
appender.JsonAsync.appenderRef.ref = JsonFile
appender.JsonAsync.blocking = false

appender.JsonFile.type = File
appender.JsonFile.name = JsonFile
appender.JsonFile.fileName = logs/tests.json
appender.JsonFile.layout.type = JsonLayout
appender.JsonFile.layout.compact = true
appender.JsonFile.layout.eventEol = true

logger.tests.appenderRefs = Console, AsyncFile, JsonAsync
```

## Интеграция с Allure

Логи можно прикреплять к отчётам Allure:

```java
package com.example.tests;

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

- **AsyncLogger vs AsyncAppender**: Используйте `AsyncLogger` для полной асинхронности или `AsyncAppender` для отдельных аппендеров.
- **Disruptor**: Всегда добавляйте зависимость `disruptor` для асинхронного логирования.
- **Уровни логирования**: `DEBUG` для отладки, `INFO` для ключевых событий, `ERROR` для исключений.
- **CI/CD**: Сохраняйте файлы логов как артефакты.
- **Производительность**: Отключайте `DEBUG` в CI/CD.

---

[**← Назад к оглавлению**](README.md)
