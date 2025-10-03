
# Настройка файла конфигурации Log4j2 для логирования в автотестах

Логирование в автотестах — незаменимый инструмент для отслеживания шагов выполнения, диагностики ошибок и анализа данных запросов/ответов. SLF4J (Simple Logging Facade for Java) предоставляет удобный интерфейс для работы с логгерами, а Log4j2 — мощная имплементация с гибкими возможностями настройки. Ключевой частью работы с Log4j2 является файл конфигурации `log4j2-test.xml`, который определяет, куда и как выводятся логи. Эта статья подробно разбирает настройку файла конфигурации Log4j2 для автотестов, включая разделение логов для UI- и API-тестов, настройку аппендеров, форматов вывода и уровней логирования.

## Зачем нужна настройка конфигурации Log4j2

Файл конфигурации Log4j2 (`log4j2-test.xml`) позволяет:
- Настраивать вывод логов в различные места: консоль, файлы, JSON.
- Разделять логи по типам тестов (например, UI и API) для удобства анализа.
- Контролировать уровни логирования (DEBUG, INFO, WARN, ERROR) для разных классов или пакетов.
- Форматировать логи для читаемости или интеграции с аналитическими системами.

## Подключение SLF4J и Log4j2 в Maven

Для работы с Log4j2 через SLF4J добавьте необходимые зависимости в проект.

### Шаг 1: Добавление зависимостей в `pom.xml`

Добавьте зависимости для SLF4J и Log4j2 в `pom.xml`. Зависимости включают `log4j-api` для программного интерфейса, `log4j-slf4j2-impl` для связки с SLF4J и `log4j-core` для реализации логирования.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>log4j2-config</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <slf4j.version>2.0.9</slf4j.version>
        <log4j.version>2.20.0</log4j.version>
    </properties>

    <dependencies>
        <!-- SLF4J API -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <!-- Log4j2 SLF4J Binding -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-slf4j2-impl</artifactId>
            <version>${log4j.version}</version>
            <scope>compile</scope>
        </dependency>
        <!-- Log4j2 API -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-api</artifactId>
            <version>${log4j.version}</version>
            <scope>compile</scope>
        </dependency>
        <!-- Log4j2 Core -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>${log4j.version}</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>
</project>
```

- **`slf4j-api`**: Интерфейс для логирования, используемый в коде.
- **`log4j-slf4j2-impl`**: Связывает SLF4J с Log4j2.
- **`log4j-api`**: Программный интерфейс Log4j2, необходимый для работы ядра.
- **`log4j-core`**: Реализует функциональность логирования (аппендеры, лейауты).

### Шаг 2: Создание файла конфигурации Log4j2

Создайте файл `log4j2-test.xml` в директории `src/test/resources`. Пример конфигурации с разделением логов для UI- и API-тестов:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="INFO">
    <Appenders>
        <!-- Console output -->
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} [%t] %-5level %c{1} - %msg%n"/>
        </Console>
        <!-- UI log file -->
        <File name="UIFile" fileName="logs/ui.log">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} [%t] %-5level %c{1} - %msg%n"/>
        </File>
        <!-- API log file -->
        <File name="APIFile" fileName="logs/api.log">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} [%t] %-5level %c{1} - %msg%n"/>
        </File>
    </Appenders>
    <Loggers>
        <!-- UI logger -->
        <Logger name="com.example.ui" level="info" additivity="false">
            <AppenderRef ref="UIFile"/>
            <AppenderRef ref="Console"/>
        </Logger>
        <!-- API logger -->
        <Logger name="com.example.api" level="info" additivity="false">
            <AppenderRef ref="APIFile"/>
            <AppenderRef ref="Console"/>
        </Logger>
        <!-- Root logger -->
        <Root level="info">
            <AppenderRef ref="Console"/>
        </Root>
    </Loggers>
</Configuration>
```

### Детальный разбор конфигурации

#### Элемент `<Configuration>`

- **Атрибут `status`**: Устанавливает уровень логирования для внутренних сообщений Log4j2 (например, `INFO`, `DEBUG`, `ERROR`). Используйте `DEBUG` для отладки самой конфигурации.
  ```xml
  <Configuration status="INFO">
  ```

#### Элемент `<Appenders>`

Аппендеры определяют, куда выводятся логи. В примере используются три аппендера:

1. **`<Console>`**:
    - **Атрибут `name`**: Уникальное имя аппендера (`Console`).
    - **Атрибут `target`**: `SYSTEM_OUT` для вывода в консоль (или `SYSTEM_ERR` для ошибок).
    - **`<PatternLayout>`**: Определяет формат логов.
        - **Атрибут `pattern`**: Формат строки лога:
            - `%d{yyyy-MM-dd HH:mm:ss}`: Дата и время.
            - `[%t]`: Имя потока.
            - `%-5level`: Уровень лога (например, INFO, DEBUG), выровненный по 5 символам.
            - `%c{1}`: Имя класса (последний сегмент полного имени).
            - `%msg%n`: Сообщение лога и перенос строки.
   ```xml
   <Console name="Console" target="SYSTEM_OUT">
       <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} [%t] %-5level %c{1} - %msg%n"/>
   </Console>
   ```

2. **`<File name="UIFile">`**:
    - **Атрибут `fileName`**: Путь к файлу логов для UI-тестов (`logs/ui.log`).
    - Использует тот же `PatternLayout`, что и консоль, для единообразия.
   ```xml
   <File name="UIFile" fileName="logs/ui.log">
       <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} [%t] %-5level %c{1} - %msg%n"/>
   </File>
   ```

3. **`<File name="APIFile">`**:
    - Аналогично, но для API-тестов, логи пишутся в `logs/api.log`.
   ```xml
   <File name="APIFile" fileName="logs/api.log">
       <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} [%t] %-5level %c{1} - %msg%n"/>
   </File>
   ```

#### Элемент `<Loggers>`

Логгеры определяют, какие классы или пакеты используют какие аппендеры и уровни логирования.

1. **`<Logger name="com.example.ui">`**:
    - **Атрибут `name`**: Пакет для UI-тестов (`com.example.ui`).
    - **Атрибут `level`**: Уровень логирования (`info` для ключевых событий, можно установить `debug` для детализации).
    - **Атрибут `additivity="false"`**: Предотвращает передачу логов в родительский логгер (Root), избегая дублирования.
    - **`<AppenderRef>`**: Ссылается на аппендеры `UIFile` и `Console`.
   ```xml
   <Logger name="com.example.ui" level="info" additivity="false">
       <AppenderRef ref="UIFile"/>
       <AppenderRef ref="Console"/>
   </Logger>
   ```

2. **`<Logger name="com.example.api">`**:
    - Аналогично, но для API-тестов, использует `APIFile` и `Console`.
   ```xml
   <Logger name="com.example.api" level="info" additivity="false">
       <AppenderRef ref="APIFile"/>
       <AppenderRef ref="Console"/>
   </Logger>
   ```

3. **`<Root>`**:
    - Корневой логгер для всех классов, не подпадающих под другие логгеры.
    - Уровень `info`, вывод только в `Console`.
   ```xml
   <Root level="info">
       <AppenderRef ref="Console"/>
   </Root>
   ```

### Шаг 3: Использование логирования в коде

Добавьте SLF4J-логгер в тестовые классы, размещённые в пакетах `com.example.ui` или `com.example.api`, чтобы использовать соответствующую конфигурацию.

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
        String password = "password123";
        logger.debug("Подготовка данных: username={}, password={}", username, password);
        login(username, password);
        logger.info("Проверка перехода на страницу dashboard");
        assertThat(driver.getCurrentUrl()).contains("/dashboard");
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

- Логи для этого класса пишутся в `logs/ui.log` и консоль.

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

- Логи для этого класса пишутся в `logs/api.log` и консоль.

## Настройка JSON-логирования

Для структурированного вывода логов в JSON добавьте зависимость:

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.15.2</version>
</dependency>
```

Обновите `log4j2-test.xml`, добавив JSON-аппендер:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="INFO">
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} [%t] %-5level %c{1} - %msg%n"/>
        </Console>
        <File name="UIFile" fileName="logs/ui.log">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} [%t] %-5level %c{1} - %msg%n"/>
        </File>
        <File name="APIFile" fileName="logs/api.log">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} [%t] %-5level %c{1} - %msg%n"/>
        </File>
        <File name="JsonFile" fileName="logs/test.json">
            <JsonLayout compact="true" eventEol="true"/>
        </File>
    </Appenders>
    <Loggers>
        <Logger name="com.example.ui" level="info" additivity="false">
            <AppenderRef ref="UIFile"/>
            <AppenderRef ref="Console"/>
            <AppenderRef ref="JsonFile"/>
        </Logger>
        <Logger name="com.example.api" level="info" additivity="false">
            <AppenderRef ref="APIFile"/>
            <AppenderRef ref="Console"/>
            <AppenderRef ref="JsonFile"/>
        </Logger>
        <Root level="info">
            <AppenderRef ref="Console"/>
        </Root>
    </Loggers>
</Configuration>
```

- **JsonFile**: Пишет логи в `logs/test.json` в формате JSON.
- Логи для `com.example.ui` и `com.example.api` также записываются в `JsonFile`.

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

- **Разделение логов**: Используйте отдельные файлы (`ui.log`, `api.log`) для UI- и API-тестов.
- **Уровни логирования**: `DEBUG` для детальной отладки, `INFO` для ключевых событий, `ERROR` для исключений.
- **Именование логгеров**: Указывайте пакеты (`com.example.ui`, `com.example.api`) для точной маршрутизации.
- **CI/CD**: Сохраняйте файлы логов как артефакты.
- **Производительность**: Отключайте `DEBUG` в CI/CD для ускорения тестов.

## Источники
- [SLF4J Documentation](http://www.slf4j.org/manual.html)
- [Log4j2 Documentation](https://logging.apache.org/log4j/2.x/manual/index.html)
- [Log4j2 Configuration](https://logging.apache.org/log4j/2.x/manual/configuration.html)
- [Log4j2 JSON Layout](https://logging.apache.org/log4j/2.x/manual/layouts.html#JSONLayout)

---
[**← Назад к оглавлению**](README.md)