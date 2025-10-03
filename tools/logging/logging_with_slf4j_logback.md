
# Добавление логирования в автотесты с использованием SLF4J и Logback

Логирование — критически важный инструмент в автотестах, который позволяет фиксировать шаги выполнения, ошибки и данные запросов/ответов, упрощая отладку и анализ результатов. SLF4J (Simple Logging Facade for Java) предоставляет гибкий интерфейс для работы с логгерами, такими как Logback, который является рекомендуемой имплементацией благодаря своей производительности и простоте настройки. Эта статья описывает, как подключить SLF4J с Logback в проекте автотестов, настроить вывод логов в консоль и файл, а также использовать логирование для UI- и API-тестов.

## Зачем нужно логирование

Логирование в автотестах помогает:
- Отслеживать действия теста, такие как клики, отправка запросов или проверки.
- Фиксировать ошибки и их контекст для быстрой диагностики.
- Записывать данные API (запросы, ответы) для анализа.
- Настраивать гибкий вывод логов в консоль, файлы или JSON с уровнями детализации: DEBUG, INFO, WARN, ERROR.

## Почему SLF4J с Logback предпочтительнее

SLF4J — это фасад, который позволяет переключаться между различными логгерами (например, Logback, Log4j2, JUL) без изменения кода. Logback выбирается как основная имплементация по следующим причинам:
- **Производительность**: Logback быстрее Log4j2 и JUL благодаря оптимизированной архитектуре.
- **Простота настройки**: Конфигурация через XML или Groovy интуитивна и поддерживает сложные сценарии.
- **Встроенная поддержка SLF4J**: Logback не требует дополнительных биндингов, в отличие от Log4j2.
- **Активное развитие**: Logback регулярно обновляется и широко используется в Java-проектах.
- **Гибкость**: Поддерживает JSON-логирование и интеграцию с аналитическими системами (например, ELK Stack).

Другие логгеры, такие как Log4j2 или JUL, существуют, но Log4j2 сложнее в настройке из-за дополнительных зависимостей, а JUL ограничен в функциональности и менее гибок.

## Подключение SLF4J с Logback в Maven

Для использования Logback через SLF4J необходимо добавить зависимости в проект и настроить конфигурацию логгера.

### Шаг 1: Добавление зависимостей в `pom.xml`

Добавьте зависимости для SLF4J и Logback в `pom.xml`.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>logging-autotests</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <slf4j.version>2.0.9</slf4j.version>
        <logback.version>1.4.11</logback.version>
    </properties>

    <dependencies>
        <!-- SLF4J API -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <!-- Logback -->
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>${logback.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

- **`slf4j-api`**: Интерфейс для логирования, используемый в коде.
- **`logback-classic`**: Включает ядро Logback и биндинг для SLF4J.

### Шаг 2: Создание конфигурации Logback

Создайте файл `logback-test.xml` в директории `src/test/resources` для настройки вывода логов.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- Console output -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- File output -->
    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
        <file>logs/test.log</file>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- Root logger -->
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
    </root>

    <!-- Specific logger for test classes -->
    <logger name="com.example.tests" level="DEBUG" additivity="false">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
    </logger>
</configuration>
```

- **CONSOLE**: Выводит логи в консоль для локальной отладки.
- **FILE**: Сохраняет логи в файл `logs/test.log` для истории.
- **Уровни логирования**: `DEBUG` для тестовых классов (`com.example.tests`), `INFO` для остального кода.
- **Формат**: Включает дату, время, поток, уровень, имя класса и сообщение.

### Шаг 3: Использование логирования в коде

Добавьте SLF4J-логгер в тестовый класс для фиксации шагов, ошибок и данных.

#### Пример UI-теста с Selenium и JUnit 5

```java
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
        // Arrange
        String username = "testuser";
        String password = "password123";
        logger.debug("Подготовка данных: username={}, password={}", username, password);
        
        // Act
        login(username, password);
        
        // Assert
        logger.info("Проверка перехода на страницу dashboard");
        assertThat(driver.getCurrentUrl()).contains("/dashboard");
    }

    private void login(String username, String password) {
        logger.debug("Ввод логина: {}", username);
        // Эмуляция ввода логина и пароля (здесь должен быть код Selenium)
        logger.info("Логин выполнен");
    }

    @AfterEach
    void tearDown() {
        logger.info("Закрытие WebDriver");
        driver.quit();
    }
}
```

- **logger.debug**: Для детальных данных (URL, параметры).
- **logger.info**: Для ключевых шагов (инициализация, завершение).

#### Пример API-теста с RestAssured

```java
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
        // Arrange
        RestAssured.baseURI = "https://api.example.com";
        logger.info("Настройка базового URI: {}", RestAssured.baseURI);
        
        // Act
        Response response = getUsers();
        logger.debug("Получен ответ API: {}", response.asString());
        
        // Assert
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

- Логи фиксируют URI, запросы, ответы и статусы для упрощения отладки.

## Преимущества логирования с SLF4J и Logback

- **Гибкость**: SLF4J позволяет переключиться на Log4j2 или JUL без изменения кода.
- **Многоканальный вывод**: Логи пишутся в консоль и файл (`logs/test.log`) для отладки и хранения.
- **Уровни логирования**: Настройка `DEBUG`, `INFO`, `WARN`, `ERROR` по классам или пакетам.
- **JSON-логирование**: Поддержка структурированного формата для интеграции с аналитическими системами.

### JSON-логирование с Logback

Для вывода логов в JSON добавьте зависимость:

```xml
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>7.4</version>
</dependency>
```

Обновите `logback-test.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <appender name="JSON" class="ch.qos.logback.core.FileAppender">
        <file>logs/test.json</file>
        <encoder class="net.logstash.logback.encoder.LogstashEncoder"/>
    </appender>

    <root level="INFO">
        <appender-ref ref="JSON"/>
    </root>
</configuration>
```

JSON-логи удобны для парсинга в CI/CD или системах, таких как ELK Stack.

## Интеграция с Allure

SLF4J с Logback можно комбинировать с Allure для прикрепления логов к отчётам:

```java
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

- **Уровни логирования**: `DEBUG` для отладки, `INFO` для ключевых событий, `ERROR` для исключений.
- **Именование логгеров**: Используйте `LoggerFactory.getLogger(ClassName.class)` для каждого класса.
- **Контекст**: Логируйте параметры, URL, ответы API для воспроизводимости.
- **CI/CD**: Сохраняйте файлы логов (`logs/test.log`) как артефакты.
- **Производительность**: Отключайте `DEBUG` в CI/CD, чтобы не замедлять тесты.

## Источники
- [SLF4J Documentation](http://www.slf4j.org/manual.html)
- [Logback Documentation](https://logback.qos.ch/documentation.html)
- [Logstash Logback Encoder](https://github.com/logstash/logstash-logback-encoder)

---
[**← Назад к оглавлению**](README.md)
