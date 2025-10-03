# Безопасность логирования

Безопасность логирования критически важна для защиты чувствительных данных, таких как пароли, токены и персональные данные (PII). В статье рассматриваются методы маскирования чувствительных данных, запрет на логирование PII и централизованная фильтрация чувствительных полей. Используется технологический стек: Java 17, Maven, SLF4J с Logback, JUnit 5, TestNG, Selenium WebDriver, RestAssured и Allure.

## Маскирование чувствительных данных

Чувствительные данные, такие как пароли, токены и API-ключи, не должны попадать в логи в открытом виде. Для этого применяются фильтры или маскирование.

### Реализация маскирования в Logback

Создайте кастомный фильтр для маскирования данных в Logback.

**Кастомный фильтр**:

```java
import ch.qos.logback.classic.spi.ILoggingEvent;
import ch.qos.logback.core.filter.Filter;
import ch.qos.logback.core.spi.FilterReply;

public class SensitiveDataFilter extends Filter<ILoggingEvent> {
    private static final String MASK = "****";

    @Override
    public FilterReply decide(ILoggingEvent event) {
        String message = event.getFormattedMessage();
        if (message.contains("password") || message.contains("token") || message.contains("apiKey")) {
            String maskedMessage = maskSensitiveData(message);
            // Создаём новый событие с замаскированным сообщением
            ILoggingEvent newEvent = new ch.qos.logback.classic.spi.LoggingEvent(
                event.getLoggerName(), 
                (ch.qos.logback.classic.Logger) event.getLoggerContextVO().getLoggerContext().getLogger(event.getLoggerName()),
                event.getLevel(),
                maskedMessage,
                event.getThrowableProxy(),
                event.getArgumentArray()
            );
            // Заменяем событие в контексте
            event.getCallerData(); // Сохраняем стек вызовов
            return FilterReply.NEUTRAL;
        }
        return FilterReply.NEUTRAL;
    }

    private String maskSensitiveData(String message) {
        return message.replaceAll("password=\\S+", "password=" + MASK)
                     .replaceAll("token=\\S+", "token=" + MASK)
                     .replaceAll("apiKey=\\S+", "apiKey=" + MASK);
    }
}
```

**Конфигурация Logback** (`logback.xml` в `src/main/resources`):

```xml
<configuration>
    <turboFilter class="SensitiveDataFilter"/>
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
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

**Пример использования**:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class SecureLogger {
    private static final Logger logger = LoggerFactory.getLogger(SecureLogger.class);

    public static void logSensitiveData(String password, String token) {
        logger.info("Входные данные: password={}, token={}", password, token);
    }
}
```

Лог будет выглядеть так:

```
2025-10-03 21:43:00 INFO  SecureLogger - Входные данные: password=****, token=****
```

## Запрет на логирование персональных данных (PII)

Персональные данные (PII), такие как имена, адреса, номера телефонов или email, не должны попадать в логи. Это соответствует требованиям GDPR, CCPA и другим стандартам защиты данных.

### Подход к запрету PII

- **Фильтрация на уровне кода**: Проверяйте данные перед логированием.
- **Использование аннотаций**: Применяйте аннотации для пометки чувствительных полей.
- **Конфигурация фильтров**: Настройте Logback для исключения PII.

**Пример проверки перед логированием**:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class PiiSafeLogger {
    private static final Logger logger = LoggerFactory.getLogger(PiiSafeLogger.class);

    public static void logSafeData(String username, String email) {
        // Проверяем, не является ли email PII
        if (email != null && email.matches(".*@.*\\..*")) {
            logger.info("Данные пользователя: username={}", username);
        } else {
            logger.info("Данные пользователя: username={}, email={}", username, email);
        }
    }
}
```

**Пример с аннотацией**:

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
@interface PII {
}

public class User {
    String username;
    @PII
    String email;

    public User(String username, String email) {
        this.username = username;
        this.email = email;
    }
}
```

**Логирование с учётом аннотаций**:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.lang.reflect.Field;

public class PiiSafeLogger {
    private static final Logger logger = LoggerFactory.getLogger(PiiSafeLogger.class);

    public static void logUserData(User user) {
        for (Field field : User.class.getDeclaredFields()) {
            if (!field.isAnnotationPresent(PII.class)) {
                try {
                    field.setAccessible(true);
                    logger.info("Поле: {} = {}", field.getName(), field.get(user));
                } catch (IllegalAccessException e) {
                    logger.error("Ошибка доступа к полю: {}", field.getName(), e);
                }
            }
        }
    }
}
```

Лог:

```
2025-10-03 21:43:00 INFO  PiiSafeLogger - Поле: username = testUser
```

Поле `email` не логируется из-за аннотации `@PII`.

## Централизованная фильтрация чувствительных полей

Централизованная фильтрация позволяет унифицировать маскирование и запрет PII через единый фильтр Logback.

### Конфигурация централизованного фильтра

**Расширенный фильтр**:

```java
import ch.qos.logback.classic.spi.ILoggingEvent;
import ch.qos.logback.core.filter.Filter;
import ch.qos.logback.core.spi.FilterReply;

public class CentralizedSensitiveDataFilter extends Filter<ILoggingEvent> {
    private static final String MASK = "****";
    private static final String[] SENSITIVE_FIELDS = {"password", "token", "apiKey", "email", "phone"};

    @Override
    public FilterReply decide(ILoggingEvent event) {
        String message = event.getFormattedMessage();
        for (String field : SENSITIVE_FIELDS) {
            if (message.toLowerCase().contains(field)) {
                String maskedMessage = message.replaceAll(field + "=\\S+", field + "=" + MASK);
                // Создаём новое событие с замаскированным сообщением
                ILoggingEvent newEvent = new ch.qos.logback.classic.spi.LoggingEvent(
                    event.getLoggerName(),
                    (ch.qos.logback.classic.Logger) event.getLoggerContextVO().getLoggerContext().getLogger(event.getLoggerName()),
                    event.getLevel(),
                    maskedMessage,
                    event.getThrowableProxy(),
                    event.getArgumentArray()
                );
                return FilterReply.NEUTRAL;
            }
        }
        return FilterReply.NEUTRAL;
    }
}
```

**Обновление `logback.xml`**:

```xml
<configuration>
    <turboFilter class="CentralizedSensitiveDataFilter"/>
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
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

## Пример теста с безопасным логированием (JUnit 5)

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

public class SecureLoginTest {
    private static final Logger logger = LoggerFactory.getLogger(SecureLoginTest.class);
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
        SecureLogger.logSensitiveData("secret", "xyz123");
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

Лог:

```
2025-10-03 21:43:00 INFO  SecureLogger - Входные данные: password=****, token=****
```

## Пример теста с безопасным логированием (TestNG)

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

public class SecureLoginTestNG {
    private static final Logger logger = LoggerFactory.getLogger(SecureLoginTestNG.class);
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
        SecureLogger.logSensitiveData("secret", "xyz123");
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

Allure позволяет прикреплять безопасные логи к отчётам.

**Пример с Allure**:

```java
import io.qameta.allure.Allure;
import io.qameta.allure.Step;
import org.junit.jupiter.api.Test;

public class SecureAllureTest {
    @Test
    void testWithSecureLogs() {
        logSecureStep("secret", "xyz123");
    }

    @Step("Логирование безопасных данных")
    private void logSecureStep(String password, String token) {
        SecureLogger.logSensitiveData(password, token);
        Allure.addAttachment("Безопасные данные", "password=****, token=****");
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

### Маскирование данных

- Используйте регулярные выражения для маскирования полей (`password`, `token`, `email`).
- Настройте централизованный фильтр для единообразной обработки.

### Запрет PII

- Проверяйте данные перед логированием, используя аннотации или условия.
- Регулярно аудитируйте логи на наличие PII.

### Централизованная фильтрация

- Реализуйте один фильтр для всех чувствительных полей.
- Тестируйте фильтр на различных сценариях (например, JSON, строки).

### Отладка

- В IntelliJ IDEA используйте точки останова в фильтрах Logback для проверки маскирования.
- Проверяйте логи в файле `logs/tests.log` или Allure-отчётах.

### Распространённые ошибки

- **Неполное маскирование**: Пропуск чувствительных полей в фильтрах.
- **Логирование PII**: Случайное включение email или номеров телефонов.
- **Сложные фильтры**: Слишком сложные регулярные выражения снижают производительность.

## Дополнительно

### Интеграция с Jenkins

Настройте Jenkins для архивирования логов и проверки безопасности:

```groovy
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh 'mvn clean test'
            }
        }
        stage('Check Logs for PII') {
            steps {
                sh '''
                    if grep -i "password\\|email" logs/tests.log; then
                        echo "Обнаружены чувствительные данные в логах"
                        exit 1
                    else
                        echo "Чувствительные данные не найдены"
                    fi
                '''
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

Testcontainers полезен для проверки фильтрации логов в интеграционных тестах.

```java
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testcontainers.containers.GenericContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

@Testcontainers
public class SecureLogTest {
    private static final Logger logger = LoggerFactory.getLogger(SecureLogTest.class);

    @Container
    private static final GenericContainer<?> logstash = new GenericContainer<>("logstash:8.15.2")
            .withExposedPorts(5000);

    @Test
    void testSecureLogging() {
        logger.info("Тест безопасного логирования");
        SecureLogger.logSensitiveData("secret", "xyz123");
        logger.info("Отправка лога в Logstash на {}:{}", logstash.getHost(), logstash.getFirstMappedPort());
    }
}
```

## Источники

- [SLF4J User Manual](http://www.slf4j.org/manual.html)
- [Logback Documentation](https://logback.qos.ch/documentation.html)
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/webdriver/)
- [RestAssured Documentation](https://rest-assured.io/)
- [Allure Framework](https://allurereport.org/docs/)
- [Testcontainers Documentation](https://testcontainers.org/)

---
[**← Назад к оглавлению**](../README.md)