# Автоматизация и аудит логов

Эффективное логирование в автоматизированных тестах требует интеграции с CI/CD, статического анализа кода и аудита логов для отслеживания действий. В статье рассматриваются проверка логов в CI, использование линтеров и Checkstyle для статического анализа, а также аудит логов для фиксации "кто, когда, что сделал". Используется технологический стек: Java 17, Maven, SLF4J с Logback, JUnit 5, TestNG, Selenium WebDriver, RestAssured, Allure и Jenkins.

## Проверка логов в CI

В CI/CD (например, Jenkins) логи должны быть доступны для анализа, архивирования и интеграции с отчётами. Это включает настройку хранения логов и их проверку в рамках пайплайна.

### Настройка Jenkins для работы с логами

Настройте Jenkins для запуска тестов, архивирования логов и генерации Allure-отчётов. Пример `Jenkinsfile`:

```groovy
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh 'mvn clean test'
            }
        }
        stage('Archive Logs') {
            steps {
                archiveArtifacts artifacts: 'logs/*.log', allowEmptyArchive: true
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
}
```

### Проверка логов в CI

Для проверки логов используйте скрипты или плагины Jenkins, такие как Log Parser Plugin, чтобы анализировать содержимое логов.

**Пример проверки логов** (скрипт в `Jenkinsfile`):

```groovy
stage('Check Logs') {
    steps {
        sh '''
            if grep -i "ERROR" logs/tests.log; then
                echo "Найдены ошибки в логах"
                exit 1
            else
                echo "Ошибки в логах отсутствуют"
            fi
        '''
    }
}
```

Логи хранятся в папке `logs/` и архивируются для последующего анализа.

### Конфигурация Logback

Настройте Logback для записи логов в файл с ротацией:

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
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} [%X{userId}] - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="info">
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

## Статический анализ: линтеры и Checkstyle

Статический анализ кода помогает обеспечить качество логирования, проверяя стиль, читаемость и наличие потенциальных ошибок.

### Checkstyle

Checkstyle проверяет код на соответствие стандартам. Например, можно настроить правило, требующее логирования через SLF4J с использованием параметризации (`logger.info("Сообщение: {}", var)` вместо `logger.info("Сообщение: " + var)`).

**Зависимость Maven для Checkstyle**:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-checkstyle-plugin</artifactId>
    <version>3.4.0</version>
    <configuration>
        <configLocation>checkstyle.xml</configLocation>
        <failOnViolation>true</failOnViolation>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

**Пример `checkstyle.xml`**:

```xml
<?xml version="1.0"?>
<!DOCTYPE module PUBLIC "-//Checkstyle//DTD Checkstyle Configuration 1.3//EN"
    "https://checkstyle.org/dtds/configuration_1_3.dtd">
<module name="Checker">
    <module name="TreeWalker">
        <module name="IllegalImport">
            <property name="illegalPkgs" value="java.util.logging"/>
        </module>
        <module name="MethodParamPad"/>
        <module name="WhitespaceAfter"/>
    </module>
</module>
```

- Запрещает использование `java.util.logging` в пользу SLF4J.
- Проверяет форматирование кода, включая отступы в вызовах методов логирования.

### Интеграция Checkstyle в CI

Добавьте выполнение Checkstyle в `Jenkinsfile`:

```groovy
stage('Static Analysis') {
    steps {
        sh 'mvn checkstyle:check'
    }
}
```

Если код не соответствует стандартам, сборка завершится с ошибкой.

## Аудит логов: кто, когда, что сделал

Аудит логов требует фиксации контекста: кто выполнил действие (например, `userId`), когда (временная метка) и что было сделано (действие). Используйте MDC (Mapped Diagnostic Context) для добавления контекста.

### Настройка MDC

Пример класса с использованием MDC:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.slf4j.MDC;

public class AuditLogger {
    private static final Logger logger = LoggerFactory.getLogger(AuditLogger.class);

    public static void logAuditAction(String userId, String action, String details) {
        MDC.put("userId", userId);
        logger.info("Аудит: {} - {}", action, details);
        MDC.clear();
    }
}
```

Обновите `logback.xml` для включения MDC:

```xml
<configuration>
    <appender name="AUDIT_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/audit.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>logs/audit.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>10MB</maxFileSize>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} [userId=%X{userId}] - %msg%n</pattern>
        </encoder>
    </appender>
    <logger name="AuditLogger" level="info" additivity="false">
        <appender-ref ref="AUDIT_FILE"/>
    </logger>
    <root level="info">
        <appender-ref ref="AUDIT_FILE"/>
    </root>
</configuration>
```

Лог будет выглядеть так:

```
2025-10-03 20:41:00 INFO  AuditLogger [userId=testUser] - Аудит: Вход в систему - Пользователь user
```

### Пример теста с аудитом (JUnit 5)

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.MDC;

import static org.assertj.core.api.Assertions.assertThat;

public class LoginTest {
    private WebDriver driver;

    @BeforeEach
    void setUp() {
        MDC.put("userId", "testUser");
        AuditLogger.logAuditAction("testUser", "Инициализация теста", "Запуск браузера");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        MDC.clear();
    }

    @Test
    void testLogin() {
        MDC.put("userId", "testUser");
        AuditLogger.logAuditAction("testUser", "Вход в систему", "Пользователь user");
        driver.get("https://example.com/login");
        assertThat(driver.getCurrentUrl()).contains("login");
        MDC.clear();
    }

    @AfterEach
    void tearDown() {
        MDC.put("userId", "testUser");
        AuditLogger.logAuditAction("testUser", "Завершение теста", "Закрытие браузера");
        if (driver != null) {
            driver.quit();
        }
        MDC.clear();
    }
}
```

### Пример теста с аудитом (TestNG)

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.slf4j.MDC;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;

import static org.testng.Assert.assertTrue;

public class LoginTestNG {
    private WebDriver driver;

    @BeforeMethod
    void setUp() {
        MDC.put("userId", "testUser");
        AuditLogger.logAuditAction("testUser", "Инициализация теста", "Запуск браузера");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
        MDC.clear();
    }

    @Test
    void testLogin() {
        MDC.put("userId", "testUser");
        AuditLogger.logAuditAction("testUser", "Вход в систему", "Пользователь user");
        driver.get("https://example.com/login");
        assertTrue(driver.getCurrentUrl().contains("login"));
        MDC.clear();
    }

    @AfterMethod
    void tearDown() {
        MDC.put("userId", "testUser");
        AuditLogger.logAuditAction("testUser", "Завершение теста", "Закрытие браузера");
        if (driver != null) {
            driver.quit();
        }
        MDC.clear();
    }
}
```

## Интеграция с Allure

Allure помогает визуализировать логи аудита в отчётах.

**Пример с Allure**:

```java
import io.qameta.allure.Allure;
import io.qameta.allure.Step;
import org.junit.jupiter.api.Test;
import org.slf4j.MDC;

public class AllureAuditTest {
    @Test
    void testWithAudit() {
        MDC.put("userId", "testUser");
        auditStep("Вход в систему", "Пользователь user");
        MDC.clear();
    }

    @Step("Аудит: {action}, детали: {details}")
    private void auditStep(String action, String details) {
        AuditLogger.logAuditAction(MDC.get("userId"), action, details);
        Allure.addAttachment("Аудит", "userId: " + MDC.get("userId") + ", действие: " + action);
    }
}
```

**Зависимости Maven для Allure**:

```xml
<dependency>
    <groupId>io.qameta.allure</groupId>
    <artifactId>allure-junit5</artifactId>
    <version>2.29.0</version>
</dependency>
<dependency>
    <groupId>io.qameta.allure</groupId>
    <artifactId>allure-testng</artifactId>
    <version>2.29.0</version>
</dependency>
```

## Лучшие практики и отладка

### Проверка логов в CI

- Архивируйте логи в Jenkins для последующего анализа.
- Используйте скрипты для поиска ошибок (`grep -i "ERROR" logs/tests.log`).

### Статический анализ

- Настройте Checkstyle для проверки использования SLF4J и формата логов.
- Проверяйте наличие MDC в аудит-логах.

### Аудит логов

- Всегда включайте `userId` и временные метки в аудит-логи.
- Используйте отдельный файл (`audit.log`) для логов аудита.

### Отладка

- В IntelliJ IDEA настройте точки останова в методах логирования для проверки MDC.
- Проверяйте Allure-отчёты для анализа логов аудита.

### Распространённые ошибки

- **Отсутствие MDC**: Логи без контекста (`userId`) усложняют аудит.
- **Неправильная конфигурация Checkstyle**: Пропуск ошибок формата логов.
- **Игнорирование логов в CI**: Неархивированные логи теряются после сборки.

## Дополнительно

### Использование Testcontainers

Testcontainers полезен для проверки интеграции с системами логирования, например, ELK.

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
    void testLogstashAudit() {
        logger.info("Отправка лога в Logstash на {}:{}", logstash.getHost(), logstash.getFirstMappedPort());
        AuditLogger.logAuditAction("testUser", "Тест Logstash", "Отправка лога");
    }
}
```

## Источники

- [SLF4J User Manual](http://www.slf4j.org/manual.html)
- [Logback Documentation](https://logback.qos.ch/documentation.html)
- [Checkstyle Documentation](https://checkstyle.sourceforge.io/)
- [Allure Framework](https://allurereport.org/docs/)
- [Testcontainers Documentation](https://testcontainers.org/)
- [Jenkins Pipeline Documentation](https://www.jenkins.io/doc/book/pipeline/)

---
[**← Назад к оглавлению**](../README.md)