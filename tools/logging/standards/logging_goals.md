# Цели логирования в проекте

Логирование преследует несколько ключевых целей, каждая из которых решает конкретные задачи в жизненном цикле проекта:

1. **Отладка и диагностика проблем**  
   Логи позволяют фиксировать последовательность событий, ведущих к ошибке или сбою. Это помогает разработчикам и тестировщикам быстро находить причину проблемы, будь то некорректная логика, сбой в API или ошибка в пользовательском интерфейсе. Например, логи могут показать, какой запрос API завершился с ошибкой и какие данные были отправлены.

2. **Мониторинг работы системы**  
   Логи предоставляют информацию о текущем состоянии приложения, включая производительность, использование ресурсов и активность пользователей. Это позволяет DevOps-инженерам отслеживать метрики, такие как время ответа API или частота использования функций, и выявлять узкие места.

3. **Обеспечение прозрачности действий**  
   Логирование фиксирует действия пользователей и системы, что важно для аудита и анализа. Например, в финансовых приложениях логи могут записывать, кто и когда выполнил транзакцию, чтобы обеспечить отслеживаемость.

4. **Обеспечение безопасности**  
   Логи помогают выявлять подозрительные действия, такие как несанкционированные попытки входа или аномальная активность. Это критично для обнаружения угроз и расследования инцидентов безопасности.

5. **Поддержка тестирования и автоматизации**  
   В автотестах логи фиксируют шаги выполнения, входные данные и результаты, упрощая отладку и анализ сбоев. Например, в UI-тестах можно логировать действия Selenium, а в API-тестах — запросы и ответы.

6. **Сбор данных для аналитики**  
   Логи могут использоваться для анализа поведения пользователей, выявления популярных функций или проблемных сценариев. Это помогает улучшать продукт, основываясь на реальных данных.

7. **Соответствие требованиям и стандартам**  
   Во многих отраслях (финансы, здравоохранение) логирование требуется для соблюдения нормативных актов, таких как GDPR или PCI DSS. Логи должны фиксировать определённые события и храниться в соответствии с политиками.

## Как логирование достигает этих целей

- **Структурированность**: Логи должны быть чёткими и содержать контекст (время, уровень, источник), чтобы упростить поиск и анализ.
- **Гибкость**: Настройка уровней логирования (например, DEBUG, INFO, ERROR) позволяет фокусироваться на нужных данных.
- **Хранение и ротация**: Логи должны храниться с учётом объёма и политики ротации, чтобы не перегружать систему.
- **Интеграция**: Логи можно интегрировать с инструментами мониторинга (например, ELK Stack) для автоматизированного анализа.

## Пример использования в автотестах

В автотестах логирование помогает фиксировать шаги и ошибки. Например, при тестировании UI с помощью Selenium можно записывать:

- Начало теста и инициализацию WebDriver.
- Ввод данных пользователем (например, логин и пароль).
- Ошибки, такие как таймауты или неожиданные исключения.

Для API-тестов логи могут включать:

- URL и параметры запроса.
- Статус-код и тело ответа.
- Исключения при сбоях.


### Отладка

Логи помогают выявить и устранить ошибки в тестах, указывая на проблемные участки кода или взаимодействия с системой.

- **Пример**: Логирование шагов теста и исключений в UI-тестах с Selenium WebDriver.
- **Что логировать**: Шаги теста, входные данные, фактические результаты, стек вызовов при ошибках.

**Пример кода** (JUnit 5):

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

public class DebugLogTest {
    private static final Logger logger = LoggerFactory.getLogger(DebugLogTest.class);
    private WebDriver driver;

    @BeforeEach
    void setUp() {
        logger.info("Инициализация теста для отладки");
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
    }

    @Test
    void testLoginDebug() {
        logger.debug("Открытие страницы логина");
        driver.get("https://example.com/login");
        try {
            logger.debug("Проверка URL: {}", driver.getCurrentUrl());
            assertThat(driver.getCurrentUrl()).contains("login");
        } catch (Exception e) {
            logger.error("Ошибка в тесте: {}", e.getMessage(), e);
            throw e;
        }
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
2025-10-03 22:10:00 INFO  DebugLogTest - Инициализация теста для отладки
2025-10-03 22:10:01 DEBUG DebugLogTest - Открытие страницы логина
2025-10-03 22:10:01 DEBUG DebugLogTest - Проверка URL: https://example.com/login
```

### Аудит

Логи для аудита фиксируют, кто, когда и что сделал, обеспечивая отслеживаемость действий.

- **Пример**: Логирование с использованием MDC для добавления `userId` и `testId`.
- **Что логировать**: Действия пользователя, контекст (например, `userId`), временные метки.

**Пример кода с MDC**:

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

**Конфигурация Logback** (`logback.xml`):

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
</configuration>
```

Лог:

```
2025-10-03 22:10:00 INFO  AuditLogger [userId=testUser] - Аудит: Вход в систему - Пользователь user
```

### Мониторинг

Логи используются для отслеживания состояния тестов в реальном времени, особенно в CI/CD.

- **Пример**: Логирование статуса API-запросов в тестах с RestAssured.
- **Что логировать**: Статус выполнения тестов, производительность, ошибки.

**Пример API-теста** (TestNG):

```java
import io.restassured.RestAssured;
import io.restassured.response.Response;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testng.annotations.Test;

import static io.restassured.RestAssured.given;
import static org.testng.Assert.assertEquals;

public class MonitorApiTest {
    private static final Logger logger = LoggerFactory.getLogger(MonitorApiTest.class);

    @Test
    void testApiMonitor() {
        logger.info("Мониторинг: Запуск API-теста");
        RestAssured.baseURI = "https://api.example.com";

        long startTime = System.currentTimeMillis();
        Response response = given()
                .when()
                .get("/users/1")
                .then()
                .extract().response();

        logger.info("Мониторинг: Статус ответа: {}, Время: {} мс", 
                    response.getStatusCode(), System.currentTimeMillis() - startTime);
        assertEquals(response.getStatusCode(), 200);
    }
}
```

Лог:

```
2025-10-03 22:10:00 INFO  MonitorApiTest - Мониторинг: Запуск API-теста
2025-10-03 22:10:01 INFO  MonitorApiTest - Мониторинг: Статус ответа: 200, Время: 150 мс
```

### Аналитика

Логи помогают анализировать поведение системы, выявлять закономерности и оптимизировать тесты.

- **Пример**: Анализ частоты ошибок в UI-тестах.
- **Что логировать**: Статистика выполнения тестов, метрики производительности, тренды ошибок.

**Пример аналитики в CI**:

```groovy
stage('Analyze Logs') {
    steps {
        sh '''
            echo "Анализ ошибок в логах"
            grep -i "ERROR" logs/tests.log | wc -l > error_count.txt
            echo "Найдено ошибок: $(cat error_count.txt)"
        '''
    }
}
```

## Кому предназначены логи

Логи создаются для разных аудиторий, каждая из которых использует их для своих задач.

### Разработчики

- **Цель**: Отладка кода и тестов, выявление причин ошибок.
- **Что нужно**: Подробные логи с шагами теста, исключениями и стеком вызовов.
- **Пример**: Логирование исключений в UI-тестах с Selenium.

### QA-инженеры

- **Цель**: Проверка корректности тестов, анализ результатов, воспроизведение ошибок.
- **Что нужно**: Логи шагов теста, входных данных и результатов проверок.
- **Пример**: Логирование с Allure для структурированных отчётов.

**Пример с Allure**:

```java
import io.qameta.allure.Allure;
import io.qameta.allure.Step;
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class QaLogTest {
    private static final Logger logger = LoggerFactory.getLogger(QaLogTest.class);

    @Test
    void testForQa() {
        logStep("Проверка страницы логина");
        Allure.addAttachment("Результат", "URL: https://example.com/login");
    }

    @Step("Шаг: {step}")
    private void logStep(String step) {
        logger.info("QA: {}", step);
    }
}
```

### DevOps

- **Цель**: Мониторинг тестов в CI/CD, управление инфраструктурой.
- **Что нужно**: Логи производительности, статусов тестов, интеграция с ELK или облаком.
- **Пример**: Логирование в Logstash для анализа в Kibana.

**Конфигурация Logback для Logstash**:

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

**Зависимость Maven**:

```xml
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>7.4</version>
</dependency>
```

### Специалисты по безопасности

- **Цель**: Аудит действий, проверка на утечку PII или чувствительных данных.
- **Что нужно**: Логи аудита с MDC, фильтрация паролей и токенов.
- **Пример**: Использование фильтра для маскирования данных.

**Пример фильтра**:

```java
import ch.qos.logback.classic.spi.ILoggingEvent;
import ch.qos.logback.core.filter.Filter;
import ch.qos.logback.core.spi.FilterReply;

public class SecurityFilter extends Filter<ILoggingEvent> {
    private static final String MASK = "****";

    @Override
    public FilterReply decide(ILoggingEvent event) {
        String message = event.getFormattedMessage();
        if (message.contains("password") || message.contains("token")) {
            String maskedMessage = message.replaceAll("password=\\S+", "password=" + MASK)
                                         .replaceAll("token=\\S+", "token=" + MASK);
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
        return FilterReply.NEUTRAL;
    }
}
```

**Обновление `logback.xml`**:

```xml
<configuration>
    <turboFilter class="SecurityFilter"/>
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

## Лучшие практики и отладка

### Отладка

- **Для разработчиков**: Используйте уровень `DEBUG` для подробных логов, включайте стек вызовов.
- **Инструменты**: В IntelliJ IDEA настройте точки останова в методах логирования для проверки сообщений.

### Аудит

- Включайте MDC с `userId` и `testId` для отслеживания действий.
- Храните аудит-логи в отдельном файле (`audit.log`).

### Мониторинг

- Интегрируйте логи с ELK Stack или облачными сервисами (например, AWS CloudWatch).
- Логируйте метрики производительности (время ответа API, длительность теста).

### Аналитика

- Используйте скрипты в CI для подсчёта ошибок или анализа трендов.
- Интегрируйте с Allure для визуализации результатов.

### Отладка

- Проверяйте логи в файлах (`logs/tests.log`, `logs/audit.log`) или в Kibana.
- Используйте фильтры Logback для проверки маскирования данных.

### Распространённые ошибки

- **Недостаточная детализация**: Пропуск шагов теста в логах усложняет отладку.
- **Отсутствие контекста**: Логи без MDC затрудняют аудит.
- **Перегрузка логов**: Избыточные логи мешают аналитике.

## Дополнительно

### Интеграция с Jenkins

Настройте Jenkins для архивирования логов и анализа:

```groovy
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh 'mvn clean test'
            }
        }
        stage('Analyze Logs') {
            steps {
                sh '''
                    echo "Подсчёт ошибок"
                    grep -i "ERROR" logs/tests.log | wc -l > error_count.txt
                    echo "Найдено ошибок: $(cat error_count.txt)"
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

Testcontainers полезен для проверки интеграции с системами мониторинга логов.

```java
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testcontainers.containers.GenericContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

@Testcontainers
public class MonitorLogTest {
    private static final Logger logger = LoggerFactory.getLogger(MonitorLogTest.class);

    @Container
    private static final GenericContainer<?> logstash = new GenericContainer<>("logstash:8.15.2")
            .withExposedPorts(5000);

    @Test
    void testLogMonitoring() {
        logger.info("Мониторинг: Отправка лога в Logstash на {}:{}", 
                    logstash.getHost(), logstash.getFirstMappedPort());
        AuditLogger.logAuditAction("testUser", "Тест мониторинга", "Logstash");
    }
}
```
## Преимущества эффективного логирования

- Ускоряет диагностику проблем, снижая время на отладку.
- Повышает прозрачность работы системы для всех участников проекта.
- Поддерживает соответствие требованиям аудита и безопасности.
- Упрощает анализ поведения системы и пользователей.

## Источники

- [SLF4J User Manual](http://www.slf4j.org/manual.html)
- [Logback Documentation](https://logback.qos.ch/documentation.html)
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/webdriver/)
- [RestAssured Documentation](https://rest-assured.io/)
- [Allure Framework](https://allurereport.org/docs/)
- [Testcontainers Documentation](https://testcontainers.org/)

---

[**← Назад к оглавлению**](../README.md)