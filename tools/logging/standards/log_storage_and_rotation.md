# Хранилище и ротация логов

Логирование в тестах требует продуманного подхода к хранению логов и управлению их жизненным циклом. В статье рассматриваются варианты хранения логов (файлы, stdout, облако, ELK), политика ротации (по размеру, времени, архивирование) и управление доступом к логам. Используется технологический стек: Java 17, Maven, SLF4J с Logback, JUnit 5, TestNG, и интеграция с Jenkins и Allure.

## Хранилище логов

Логи могут храниться в различных местах в зависимости от потребностей проекта. Рассмотрим основные варианты.

### Локальные файлы

Логи записываются в файлы на локальной машине или сервере CI/CD. Это наиболее распространённый способ для тестов.

**Конфигурация Logback** (`logback.xml` в `src/main/resources`):

```xml
<configuration>
    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
        <file>logs/tests.log</file>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="info">
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

Логи сохраняются в `logs/tests.log`. Это удобно для локальной отладки, но требует управления дисковым пространством.

### Stdout (консоль)

Вывод логов в консоль полезен для локального запуска тестов или анализа в CI/CD.

**Конфигурация Logback**:

```xml
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="info">
        <appender-ref ref="CONSOLE"/>
    </root>
</configuration>
```

Консольные логи подходят для немедленного просмотра, но неудобны для долгосрочного хранения.

### Облако

Логи можно отправлять в облачные сервисы, такие как AWS CloudWatch или Google Cloud Logging. Это требует интеграции с их API, что выходит за рамки стандартного Logback, но может быть реализовано через дополнительные библиотеки, такие как `logback-cloudwatch`.

**Пример зависимости для AWS CloudWatch**:

```xml
<dependency>
    <groupId>io.github.dhabensky</groupId>
    <artifactId>logback-cloudwatch</artifactId>
    <version>0.1.5</version>
</dependency>
```

**Конфигурация Logback** для CloudWatch:

```xml
<configuration>
    <appender name="CLOUDWATCH" class="io.github.dhabensky.logback.cloudwatch.CloudWatchAppender">
        <accessKeyId>${AWS_ACCESS_KEY}</accessKeyId>
        <secretAccessKey>${AWS_SECRET_KEY}</secretAccessKey>
        <logGroupName>TestLogs</logGroupName>
        <logStreamName>TestStream</logStreamName>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="info">
        <appender-ref ref="CLOUDWATCH"/>
    </root>
</configuration>
```

Облачное хранение подходит для распределённых систем и долгосрочного хранения, но требует настройки доступа и затрат.

### ELK Stack (Elasticsearch, Logstash, Kibana)

ELK Stack используется для централизованного хранения и анализа логов. Logback может отправлять логи в Logstash через аппендер `LogstashTcpSocketAppender`.

**Зависимость для Logstash**:

```xml
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>7.4</version>
</dependency>
```

**Конфигурация Logback**:

```xml
<configuration>
    <appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <destination>localhost:5000</destination>
        <encoder class="net.logstash.logback.encoder.LogstashEncoder"/>
    </appender>
    <root level="info">
        <appender-ref ref="LOGSTASH"/>
    </root>
</configuration>
```

ELK Stack позволяет визуализировать логи в Kibana, что удобно для анализа больших объёмов данных.

## Политика ротации логов

Ротация логов предотвращает переполнение дискового пространства. Logback поддерживает ротацию по размеру, времени и архивирование.

### Ротация по размеру

Логи вращаются, когда файл достигает заданного размера.

**Конфигурация Logback**:

```xml
<configuration>
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/tests.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>logs/tests.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>10MB</maxFileSize>
            <maxHistory>30</maxHistory>
            <totalSizeCap>1GB</totalSizeCap>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="info">
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

- `maxFileSize`: Максимальный размер файла (10 МБ).
- `maxHistory`: Хранить файлы за последние 30 дней.
- `totalSizeCap`: Общий лимит размера логов (1 ГБ).

### Ротация по времени

Логи вращаются ежедневно или по другому временному интервалу.

**Конфигурация Logback**:

```xml
<configuration>
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/tests.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/tests.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
            <totalSizeCap>1GB</totalSizeCap>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="info">
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

Логи создаются ежедневно с именем `tests.2025-10-03.log`.

### Архивирование

Logback автоматически архивирует старые логи в `.zip` или `.gz` при использовании `<cleanHistoryOnStart>true</cleanHistoryOnStart>`.

**Конфигурация с архивированием**:

```xml
<configuration>
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/tests.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>logs/tests.%d{yyyy-MM-dd}.%i.log.zip</fileNamePattern>
            <maxFileSize>10MB</maxFileSize>
            <maxHistory>30</maxHistory>
            <totalSizeCap>1GB</totalSizeCap>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="info">
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

Старые логи сжимаются в `.zip`, что экономит место.

## Уровень доступа к логам

Контроль доступа к логам важен для защиты чувствительных данных, таких как `userId` или параметры API.

### Локальные файлы

- **Ограничение доступа**: Настройте права доступа к папке `logs` (например, `chmod 640 logs/*` на Linux).
- **Маскирование данных**: Используйте фильтры в Logback для маскирования чувствительных данных.

**Пример фильтра Logback**:

```xml
<configuration>
    <turboFilter class="ch.qos.logback.classic.turbo.DynamicThresholdFilter">
        <Key>sensitiveData</Key>
        <DefaultThreshold>ERROR</DefaultThreshold>
        <OnMatch>FILTER</OnMatch>
    </turboFilter>
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/tests.log</file>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="info">
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

### Облако и ELK

- **AWS CloudWatch**: Настройте IAM-политики для ограничения доступа к логам.
- **ELK Stack**: Используйте Kibana Spaces и роли для контроля доступа.
- **Шифрование**: Включайте шифрование в канале передачи (TLS для Logstash).

### Пример теста с логированием (JUnit 5)

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import static org.assertj.core.api.Assertions.assertThat;

public class LoginTest {
    private static final Logger logger = LoggerFactory.getLogger(LoginTest.class);
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

Логи сохраняются в `logs/tests.log` с ротацией по размеру и времени.

### Пример теста с логированием (TestNG)

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

Allure отображает логи в отчётах. Пример:

```java
import io.qameta.allure.Allure;
import io.qameta.allure.Step;
import org.junit.jupiter.api.Test;

public class AllureTest {
    @Test
    void testWithLogs() {
        logStep("Запуск теста");
        Allure.addAttachment("Log File", "text/plain", "Log content from tests.log");
    }

    @Step("Шаг: {step}")
    private void logStep(String step) {
        Logger logger = LoggerFactory.getLogger(AllureTest.class);
        logger.info("Allure: {}", step);
    }
}
```

## Лучшие практики и отладка

### Хранилище

- Используйте локальные файлы для локальной разработки, облако или ELK для CI/CD.
- Настройте ротацию, чтобы избежать переполнения диска.

### Ротация

- Комбинируйте ротацию по размеру и времени для гибкости.
- Храните архивы не более 30 дней, если нет других требований.

### Доступ

- Ограничивайте доступ к логам через права файлов или IAM-политики.
- Маскируйте чувствительные данные перед логированием.

### Отладка

- Используйте IntelliJ IDEA для анализа логов через точки останова.
- Проверяйте файлы логов или Kibana для поиска ошибок.

### Распространённые ошибки

- **Отсутствие ротации**: Переполнение диска из-за больших логов.
- **Незащищённые логи**: Утечка чувствительных данных.
- **Сложность анализа**: Неправильная структура логов затрудняет поиск.

## Дополнительно

### Интеграция с Jenkins

Настройте Jenkins для архивирования логов:

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

Testcontainers полезен для тестирования хранилища логов, например, с Logstash.

```java
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testcontainers.containers.GenericContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

@Testcontainers
public class LogstashTest {
    private static final Logger logger = LoggerFactory.getLogger(LogstashTest.class);

    @Container
    private static final GenericContainer<?> logstash = new GenericContainer<>("logstash:8.15.2")
            .withExposedPorts(5000);

    @Test
    void testLogstash() {
        logger.info("Отправка лога в Logstash на {}:{}", logstash.getHost(), logstash.getFirstMappedPort());
        // Логи отправляются в Logstash
    }
}
```

## Источники

- [SLF4J User Manual](http://www.slf4j.org/manual.html)
- [Logback Documentation](https://logback.qos.ch/documentation.html)
- [Logstash Documentation](https://www.elastic.co/guide/en/logstash/current/index.html)
- [AWS CloudWatch Logs](https://docs.aws.amazon.com/cloudwatch/index.html)
- [Allure Framework](https://allurereport.org/docs/)

---
[**← Назад к оглавлению**](../README.md)