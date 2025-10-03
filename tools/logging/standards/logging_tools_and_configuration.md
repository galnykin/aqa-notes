# Инструменты и конфигурация логирования

Эффективное логирование в тестах требует выбора подходящих библиотек и их правильной конфигурации. В статье рассматриваются библиотеки Log4j2, SLF4J и Logback, варианты конфигурации (XML, YAML, .properties) и поддержка асинхронного логирования. Используется технологический стек: Java 17, Maven, JUnit 5, TestNG, Selenium WebDriver, RestAssured и Allure.

## Используемые библиотеки

### SLF4J

SLF4J (Simple Logging Facade for Java) — это фасад, обеспечивающий единый API для различных систем логирования (Logback, Log4j2 и др.). Он позволяет переключать реализации без изменения кода.

**Зависимость Maven**:

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>2.0.16</version>
</dependency>
```

### Logback

Logback — это реализация логирования для SLF4J, известная своей гибкостью и производительностью. Используется для локальных файлов, консоли и интеграции с внешними системами.

**Зависимость Maven**:

```xml
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.5.8</version>
</dependency>
```

### Log4j2

Log4j2 — мощная альтернатива Logback с поддержкой асинхронного логирования и высокой производительностью. Требует мост для работы с SLF4J.

**Зависимости Maven**:

```xml
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.24.0</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.24.0</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j-impl</artifactId>
    <version>2.24.0</version>
</dependency>
```

## Конфигурация библиотек

Конфигурация логирования задаётся в файлах XML, YAML или .properties. Рассмотрим примеры для Logback и Log4j2.

### Logback: XML

Файл `logback.xml` в `src/main/resources`:

```xml
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/tests.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>logs/tests.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>10MB</maxFileSize>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="info">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

### Logback: YAML

Logback не поддерживает YAML напрямую, но можно использовать библиотеку `snakeyaml` для парсинга YAML в Logback. Пример файла `logback.yaml`:

```yaml
configuration:
  appenders:
    console:
      name: CONSOLE
      target: SYSTEM_OUT
      patternLayout:
        pattern: "%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n"
    rollingFile:
      name: FILE
      fileName: logs/tests.log
      filePattern: logs/tests.%d{yyyy-MM-dd}.%i.log
      sizeBasedTriggeringPolicy:
        maxFileSize: 10MB
      timeBasedRollingPolicy:
        maxHistory: 30
      patternLayout:
        pattern: "%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n"
  loggers:
    root:
      level: info
      appender-ref:
        - ref: CONSOLE
        - ref: FILE
```

**Зависимость для YAML**:

```xml
<dependency>
    <groupId>org.yaml</groupId>
    <artifactId>snakeyaml</artifactId>
    <version>2.3</version>
</dependency>
```

### Log4j2: XML

Файл `log4j2.xml` в `src/main/resources`:

```xml
<Configuration status="INFO">
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1} - %m%n"/>
        </Console>
        <RollingFile name="File" fileName="logs/tests.log"
                     filePattern="logs/tests-%d{yyyy-MM-dd}-%i.log">
            <Policies>
                <SizeBasedTriggeringPolicy size="10MB"/>
                <TimeBasedTriggeringPolicy/>
            </Policies>
            <DefaultRolloverStrategy max="30"/>
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1} - %m%n"/>
        </RollingFile>
    </Appenders>
    <Loggers>
        <Root level="info">
            <AppenderRef ref="Console"/>
            <AppenderRef ref="File"/>
        </Root>
    </Loggers>
</Configuration>
```

### Log4j2: YAML

Файл `log4j2.yaml` в `src/main/resources`:

```yaml
Configuration:
  status: INFO
  Appenders:
    Console:
      name: Console
      target: SYSTEM_OUT
      PatternLayout:
        Pattern: "%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1} - %m%n"
    RollingFile:
      name: File
      fileName: logs/tests.log
      filePattern: logs/tests-%d{yyyy-MM-dd}-%i.log
      Policies:
        SizeBasedTriggeringPolicy:
          size: 10MB
        TimeBasedTriggeringPolicy: {}
      DefaultRolloverStrategy:
        max: 30
      PatternLayout:
        Pattern: "%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1} - %m%n"
  Loggers:
    Root:
      level: info
      AppenderRef:
        - ref: Console
        - ref: File
```

### Log4j2: .properties

Файл `log4j2.properties` в `src/main/resources`:

```properties
status=INFO
appenders=Console,File
appender.console.type=Console
appender.console.name=Console
appender.console.layout.type=PatternLayout
appender.console.layout.pattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1} - %m%n
appender.file.type=RollingFile
appender.file.name=File
appender.file.fileName=logs/tests.log
appender.file.filePattern=logs/tests-%d{yyyy-MM-dd}-%i.log
appender.file.layout.type=PatternLayout
appender.file.layout.pattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1} - %m%n
appender.file.policies.type=Policies
appender.file.policies.size.type=SizeBasedTriggeringPolicy
appender.file.policies.size.size=10MB
appender.file.policies.time.type=TimeBasedTriggeringPolicy
appender.file.strategy.type=DefaultRolloverStrategy
appender.file.strategy.max=30
rootLogger.level=info
rootLogger.appenderRefs=console,file
rootLogger.appenderRef.console.ref=Console
rootLogger.appenderRef.file.ref=File
```

## Поддержка асинхронного логирования

Асинхронное логирование повышает производительность, особенно в многопоточных тестах, минимизируя задержки при записи логов.

### Logback: Асинхронное логирование

Используйте `AsyncAppender` для асинхронного логирования.

**Конфигурация Logback**:

```xml
<configuration>
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/tests.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>logs/tests.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>10MB</maxFileSize>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <appender name="ASYNC" class="ch.qos.logback.classic.AsyncAppender">
        <appender-ref ref="FILE"/>
        <queueSize>500</queueSize>
        <discardingThreshold>0</discardingThreshold>
    </appender>
    <root level="info">
        <appender-ref ref="ASYNC"/>
    </root>
</configuration>
```

- `queueSize`: Размер очереди для асинхронных логов.
- `discardingThreshold`: Порог для отбрасывания логов при переполнении очереди.

### Log4j2: Асинхронное логирование

Log4j2 поддерживает асинхронное логирование через `AsyncLogger`.

**Конфигурация Log4j2 (XML)**:

```xml
<Configuration status="INFO">
    <Appenders>
        <RollingFile name="File" fileName="logs/tests.log"
                     filePattern="logs/tests-%d{yyyy-MM-dd}-%i.log">
            <Policies>
                <SizeBasedTriggeringPolicy size="10MB"/>
                <TimeBasedTriggeringPolicy/>
            </Policies>
            <DefaultRolloverStrategy max="30"/>
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1} - %m%n"/>
        </RollingFile>
    </Appenders>
    <Loggers>
        <AsyncLogger name="root" level="info" additivity="false">
            <AppenderRef ref="File"/>
        </AsyncLogger>
    </Loggers>
</Configuration>
```

Для глобального асинхронного логирования добавьте системное свойство:

```java
System.setProperty("log4j2.contextSelector", "org.apache.logging.log4j.core.async.AsyncLoggerContextSelector");
```

**Зависимость для асинхронного логирования**:

```xml
<dependency>
    <groupId>com.lmax</groupId>
    <artifactId>disruptor</artifactId>
    <version>4.0.0</version>
</dependency>
```

## Пример использования в тестах

### JUnit 5: Логирование с Log4j2

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

import static org.assertj.core.api.Assertions.assertThat;

public class LoginTest {
    private static final Logger logger = LogManager.getLogger(LoginTest.class);
    private WebDriver driver;

    @BeforeEach
    void setUp() {
        logger.info("Инициализация теста");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
    }

    @Test
    void testLogin() {
        logger.info("Запуск теста логина");
        driver.get("https://example.com/login");
        logger.info("Проверка URL");
        assertThat(driver.getCurrentUrl()).contains("login");
    }

    @AfterEach
    void tearDown() {
        logger.info("Завершение теста");
        if (driver != null) {
            driver.quit();
        }
    }
}
```

### TestNG: Логирование с Logback

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;

import static org.testng.Assert.assertTrue;

public class LoginTestNG {
    private static final Logger logger = LoggerFactory.getLogger(LoginTestNG.class);
    private WebDriver driver;

    @BeforeMethod
    void setUp() {
        logger.info("Инициализация теста");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
    }

    @Test
    void testLogin() {
        logger.info("Запуск теста логина");
        driver.get("https://example.com/login");
        logger.info("Проверка URL");
        assertTrue(driver.getCurrentUrl().contains("login"));
    }

    @AfterMethod
    void tearDown() {
        logger.info("Завершение теста");
        if (driver != null) {
            driver.quit();
        }
    }
}
```

## Интеграция с Allure

Allure отображает логи в отчётах. Пример с Log4j2:

```java
import io.qameta.allure.Allure;
import io.qameta.allure.Step;
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.junit.jupiter.api.Test;

public class AllureTest {
    private static final Logger logger = LogManager.getLogger(AllureTest.class);

    @Test
    void testWithLogs() {
        logStep("Запуск теста");
        Allure.addAttachment("Log File", "text/plain", "Log content from tests.log");
    }

    @Step("Шаг: {step}")
    private void logStep(String step) {
        logger.info("Allure: {}", step);
    }
}
```

## Лучшие практики и отладка

### Выбор библиотеки

- **SLF4J + Logback**: Подходит для большинства проектов благодаря простоте и гибкости.
- **Log4j2**: Используйте для высокой производительности и асинхронного логирования в крупных проектах.

### Конфигурация

- Предпочитайте XML или YAML для сложных конфигураций, .properties — для простых.
- Настройте ротацию логов (размер, время) для управления дисковым пространством.

### Асинхронное логирование

- Включайте асинхронное логирование для многопоточных тестов.
- Настройте размер очереди (`queueSize` в Logback, `disruptor` в Log4j2) для баланса между производительностью и надёжностью.

### Отладка

- Используйте IntelliJ IDEA для просмотра логов через точки останова.
- Проверяйте файлы конфигурации (`logback.xml`, `log4j2.yaml`) на синтаксические ошибки.

### Распространённые ошибки

- **Отсутствие моста SLF4J**: Без `log4j-slf4j-impl` Log4j2 не работает с SLF4J.
- **Неправильная конфигурация**: Ошибки в XML/YAML могут привести к молчаливому сбою.
- **Перегрузка очереди**: Слишком маленький `queueSize` в асинхронном режиме может отбросить логи.

## Дополнительно

### Интеграция с Jenkins

Настройте Jenkins для архивирования логов и генерации отчётов Allure:

```groovy
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh 'mvn clean test'
            }
        }
        stage('Allure Report') {
            steps {
                allure([
                    includeProperties: false,
                    jdk: '',
                    results: [[path: 'target/allure-results']]
                ])
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'logs/*.log', allowEmptyArchive: true
        }
    }
}
```

### Использование Testcontainers

Testcontainers полезен для тестирования интеграции с внешними системами логирования.

```java
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testcontainers.containers.GenericContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

@Testcontainers
public class LogTest {
    private static final Logger logger = LoggerFactory.getLogger(LogTest.class);

    @Container
    private static final GenericContainer<?> logstash = new GenericContainer<>("logstash:8.15.2")
            .withExposedPorts(5000);

    @Test
    void testLogstash() {
        logger.info("Отправка лога в Logstash на {}:{}", logstash.getHost(), logstash.getFirstMappedPort());
    }
}
```

## Источники

- [SLF4J User Manual](http://www.slf4j.org/manual.html)
- [Logback Documentation](https://logback.qos.ch/documentation.html)
- [Log4j2 Documentation](https://logging.apache.org/log4j/2.x/)
- [Allure Framework](https://allurereport.org/docs/)
- [Testcontainers Documentation](https://testcontainers.org/)

---
[**← Назад к оглавлению**](../README.md)