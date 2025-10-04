
# Подключение логирования с SLF4J и Log4j2 в автотестах

Логирование в автотестах играет ключевую роль, позволяя фиксировать шаги выполнения, ошибки и данные запросов/ответов, что упрощает отладку и анализ результатов. SLF4J (Simple Logging Facade for Java) предоставляет универсальный интерфейс для работы с логгерами, а Log4j2 — мощная имплементация с высокой производительностью и гибкой настройкой. Эта статья описывает, как настроить SLF4J с Log4j2 для использования в автотестах, создать конфигурацию для вывода логов в консоль и файл, и применять логирование в UI- и API-тестах.

## Зачем нужно логирование

Логирование помогает:
- Отслеживать шаги теста (например, клики, запросы, проверки).
- Фиксировать ошибки и их контекст для быстрого устранения.
- Записывать данные API (запросы, ответы) для анализа.
- Настраивать вывод логов в консоль, файлы или JSON с уровнями детализации: DEBUG, INFO, WARN, ERROR.

## Настройка SLF4J и Log4j2 в Maven

Для использования Log4j2 через SLF4J необходимо добавить зависимости в проект и настроить конфигурацию логгера.

### Шаг 1: Добавление зависимостей в `pom.xml`

Добавьте зависимости для SLF4J и Log4j2 в `pom.xml`. Зависимость `log4j-api` обеспечивает программный интерфейс Log4j2, `log4j-slf4j2-impl` связывает SLF4J с Log4j2, а `log4j-core` реализует функциональность логирования.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>logging-slf4j-log4j2-api</artifactId>
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
- **`log4j-core`**: Реализует функциональность логирования (например, аппендеры, лейауты).

### Шаг 2: Создание конфигурации Log4j2

Создайте файл `log4j2-test.xml` в директории `src/test/resources` для настройки вывода логов в консоль и файл.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="INFO">
    <Appenders>
        <!-- Console output -->
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} [%t] %-5p %c{1} - %m%n"/>
        </Console>
        <!-- File output -->
        <File name="File" fileName="logs/test.log">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} [%t] %-5p %c{1} - %m%n"/>
        </File>
    </Appenders>
    <Loggers>
        <!-- Logger for test classes -->
        <Logger name="com.example.tests" level="debug" additivity="false">
            <AppenderRef ref="Console"/>
            <AppenderRef ref="File"/>
        </Logger>
        <!-- Root logger -->
        <Root level="info">
            <AppenderRef ref="Console"/>
            <AppenderRef ref="File"/>
        </Root>
    </Loggers>
</Configuration>
```

- **Console**: Выводит логи в консоль для локальной отладки.
- **File**: Сохраняет логи в файл `logs/test.log` для хранения истории.
- **Уровни логирования**: `DEBUG` для тестовых классов (`com.example.tests`), `INFO` для остального кода.
- **Формат логов**: Включает дату, время, поток, уровень, имя класса и сообщение.

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

- **logger.debug**: Фиксирует детальные данные (URL, параметры).
- **logger.info**: Отмечает ключевые шаги (инициализация, завершение).

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

- Логи фиксируют URI, запросы, ответы и их статусы для упрощения отладки.

## Преимущества логирования с SLF4J и Log4j2

- **Гибкость**: SLF4J позволяет переключаться на другие логгеры (например, Logback) без изменения кода.
- **Многоканальный вывод**: Логи пишутся в консоль и файл (`logs/test.log`) для отладки и хранения.
- **Уровни логирования**: Настройка `DEBUG`, `INFO`, `WARN`, `ERROR` по классам или пакетам через `log4j2-test.xml`.
- **Производительность**: Log4j2 оптимизирован для асинхронного логирования, что снижает нагрузку на тесты.

### JSON-логирование с Log4j2

Для вывода логов в JSON добавьте зависимость:

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.15.2</version>
</dependency>
```

Обновите `log4j2-test.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="INFO">
    <Appenders>
        <File name="JsonFile" fileName="logs/test.json">
            <JsonLayout compact="true" eventEol="true"/>
        </File>
    </Appenders>
    <Loggers>
        <Root level="info">
            <AppenderRef ref="JsonFile"/>
        </Root>
    </Loggers>
</Configuration>
```

JSON-логи удобны для интеграции с аналитическими системами, такими как ELK Stack.

## Интеграция с Allure

Логи SLF4J можно комбинировать с Allure для прикрепления к отчётам:

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

- **Уровни логирования**: Используйте `DEBUG` для детальной отладки, `INFO` для ключевых событий, `ERROR` для исключений.
- **Именование логгеров**: Создавайте логгер для каждого класса (`LoggerFactory.getLogger(ClassName.class)`).
- **Контекст**: Логируйте параметры, URL, ответы API для воспроизводимости.
- **CI/CD**: Сохраняйте файлы логов (`logs/test.log`) как артефакты в GitHub Actions или Jenkins.
- **Производительность**: Отключайте `DEBUG` в CI/CD для ускорения тестов.

## Источники
- [SLF4J Documentation](http://www.slf4j.org/manual.html)
- [Log4j2 Documentation](https://logging.apache.org/log4j/2.x/manual/index.html)
- [Log4j2 JSON Layout](https://logging.apache.org/log4j/2.x/manual/layouts.html#JSONLayout)

---
[**← Назад к оглавлению**](README.md)
