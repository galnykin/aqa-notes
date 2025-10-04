# Интеграция Allure с Jenkins: автоматизация отчётов в CI/CD

Далее мы рассмотрим, как интегрировать Allure Framework с Jenkins для автоматической генерации и публикации отчётов в CI/CD. Это позволяет командам получать наглядные результаты тестов после каждого билда, упрощая анализ и отладку.

## Шаги интеграции

### 1. Установка Allure Plugin в Jenkins

- Перейдите в `Manage Jenkins → Manage Plugins`.
- Найдите и установите **Allure Jenkins Plugin** через раздел `Available Plugins`.
- После установки плагин появится в настройках проектов.

### 2. Настройка Maven

Добавьте зависимости и плагин Allure в `pom.xml`. Зависимости включают поддержку JUnit 5 и TestNG (если используются оба фреймворка). AspectJ необходим для работы Allure.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>allure-jenkins</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <allure.version>2.25.0</allure.version>
        <aspectj.version>1.9.21</aspectj.version>
    </properties>

    <dependencies>
        <!-- Allure JUnit 5 -->
        <dependency>
            <groupId>io.qameta.allure</groupId>
            <artifactId>allure-junit5</artifactId>
            <version>${allure.version}</version>
            <scope>test</scope>
        </dependency>
        <!-- Allure TestNG -->
        <dependency>
            <groupId>io.qameta.allure</groupId>
            <artifactId>allure-testng</artifactId>
            <version>${allure.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.2.3</version>
                <configuration>
                    <argLine>
                        -javaagent:${settings.localRepository}/org/aspectj/aspectjweaver/${aspectj.version}/aspectjweaver-${aspectj.version}.jar
                    </argLine>
                </configuration>
                <dependencies>
                    <dependency>
                        <groupId>org.aspectj</groupId>
                        <artifactId>aspectjweaver</artifactId>
                        <version>${aspectj.version}</version>
                    </dependency>
                </dependencies>
            </plugin>
            <plugin>
                <groupId>io.qameta.allure</groupId>
                <artifactId>allure-maven</artifactId>
                <version>2.12.0</version>
                <configuration>
                    <reportVersion>${allure.version}</reportVersion>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### 3. Настройка allure.properties

Создайте файл `src/test/resources/allure.properties` для указания директории результатов:

```properties
allure.results.directory=target/allure-results
allure.attachments.screenshots=true
allure.logging.level=INFO
```

### 4. Настройка Jenkins Pipeline

Добавьте шаг для генерации и публикации Allure-отчёта в `Jenkinsfile`:

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Run Tests') {
            steps {
                sh 'mvn clean test'
            }
        }
    }
    post {
        always {
            allure includeProperties: false, jdk: '', results: [[path: 'target/allure-results']]
        }
    }
}
```

- `allure` шаг использует Allure Jenkins Plugin для генерации отчёта.
- `results: [[path: 'target/allure-results']]` указывает директорию с результатами тестов.

### 5. Просмотр отчёта

- После успешного билда в Jenkins откройте вкладку **Allure Report** в интерфейсе проекта.
- Отчёт отображает шаги, статусы, скриншоты и вложения, если они настроены в тестах.

### Пример теста с Allure

Пример UI-теста с использованием Selenium, JUnit 5 и аннотаций Allure (подробно описан в предыдущих статьях):

```java
import io.qameta.allure.Description;
import io.qameta.allure.Severity;
import io.qameta.allure.SeverityLevel;
import io.qameta.allure.Step;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import static org.assertj.core.api.Assertions.assertThat;

public class SimpleAllureTest {

    @Test
    @Description("Проверка загрузки страницы")
    @Severity(SeverityLevel.NORMAL)
    void testPageLoad() {
        // Arrange
        WebDriverManager.chromedriver().setup();
        WebDriver driver = new ChromeDriver();
        
        // Act
        loadPage(driver, "https://example.com");
        
        // Assert
        assertThat(driver.getCurrentUrl()).contains("example.com");
        driver.quit();
    }

    @Step("Загрузка страницы {url}")
    private void loadPage(WebDriver driver, String url) {
        driver.get(url);
    }
}
```

## Советы по интеграции

- **Разделение отчётов**: Используйте аннотации `@Epic`, `@Feature` или теги для разделения smoke и regression тестов в отчётах.
- **Хранение отчётов**: Настройте архивирование директории `target/allure-results` как артефакт в Jenkins (`archiveArtifacts artifacts: 'target/allure-results/**'`).
- **Flaky тесты**: Добавьте механизм повторов (например, `@Retry` в TestNG) и включайте логи/скриншоты для failed-тестов с помощью `@Attachment`.
- **Отладка**: Используйте IntelliJ IDEA для проверки локаторов и логов (SLF4J+Logback), если тесты падают.

## Источники
- [Allure Jenkins Plugin](https://plugins.jenkins.io/allure-jenkins-plugin)
- [Allure Maven Documentation](https://docs.qameta.io/allure/#_maven)
- [Allure Framework Documentation](https://docs.qameta.io/allure/)

---
[**← Назад к оглавлению**](README.md)
