
# Генерация Allure-отчёта в IntelliJ IDEA: пошаговая инструкция

Далее мы рассмотрим, как настроить и сгенерировать Allure-отчёты в IntelliJ IDEA, чтобы получать наглядные результаты автотестов прямо из IDE. Это упрощает анализ результатов и отладку.

## Что нужно для генерации отчёта

Для работы с Allure в IntelliJ IDEA потребуются:
1. Плагин **Allure Report** для IntelliJ IDEA.
2. **Allure CLI** для генерации отчётов.
3. Проект с настроенными зависимостями Allure в Maven.
4. Результаты тестов в директории `target/allure-results`.

## Шаг 1: Установка плагина Allure в IntelliJ IDEA

- Откройте `File → Settings → Plugins → Marketplace`.
- Найдите плагин **Allure Report** и установите его.
- Перезапустите IntelliJ IDEA для активации плагина.

## Шаг 2: Установка Allure CLI

- Скачайте Allure с [официальной страницы GitHub Releases](https://github.com/allure-framework/allure2/releases).
- Распакуйте архив и добавьте папку `bin` в переменную окружения `PATH`.
- Проверьте установку в терминале:
  ```bash
  allure --version
  ```

## Шаг 3: Настройка зависимостей в `pom.xml`

Добавьте зависимости для Allure в `pom.xml`. Пример включает поддержку JUnit 5 и TestNG (оставьте только нужный фреймворк, если используете один).

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>allure-intellij</artifactId>
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

- Убедитесь, что директория результатов настроена в `src/test/resources/allure.properties` (как описано ранее).

## Шаг 4: Запуск тестов и генерация отчёта

### Вариант 1: Через терминал IntelliJ

1. Запустите тесты:
   ```bash
   mvn clean test
   ```
2. Сгенерируйте отчёт:
   ```bash
   allure generate target/allure-results --clean -o target/allure-report
   ```
3. Откройте отчёт в браузере:
   ```bash
   allure open target/allure-report
   ```

### Вариант 2: Через плагин Allure в IntelliJ

- После выполнения тестов (`mvn clean test`) откройте вкладку **Allure** в нижней панели IntelliJ IDEA.
- Нажмите **Generate Report** или укажите путь к `target/allure-results`.
- Отчёт автоматически откроется в браузере.

## Что должно быть в `target/allure-results`

- JSON-файлы с результатами тестов, создаваемые Allure при запуске.
- Скриншоты и вложения (если используются аннотации `@Attachment`).
- Файлы логов, если настроено логирование через SLF4J+Logback.

### Пример теста с Allure

Пример UI-теста с JUnit 5 и Selenium (подробно описан ранее):

```java
import io.qameta.allure.Description;
import io.qameta.allure.Severity;
import io.qameta.allure.SeverityLevel;
import io.qameta.allure.Step;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import static org.assertj.core.api.Assertions.assertThat;

public class AllureSimpleTest {

    private WebDriver driver;

    @BeforeEach
    void setUp() {
        WebDriverManager.chromedriver().setup();
        driver = new ChromeDriver();
    }

    @Test
    @Description("Проверка загрузки страницы")
    @Severity(SeverityLevel.NORMAL)
    void testPageLoad() {
        // Arrange & Act
        loadPage("https://example.com");
        
        // Assert
        assertThat(driver.getCurrentUrl()).contains("example.com");
    }

    @Step("Загрузка страницы {url}")
    private void loadPage(String url) {
        driver.get(url);
    }

    @AfterEach
    void tearDown() {
        driver.quit();
    }
}
```

## Советы

- Используйте аннотации `@Step`, `@Attachment`, `@Severity`, `@Description` для информативных отчётов (подробно описаны ранее).
- Настройте `allure.properties` в `src/test/resources` для кастомизации директории результатов и других параметров, как указано в предыдущих статьях.
- Для UI-тестов с Selenium применяйте `WebDriverWait` и стабильные локаторы (`id`, `data-*`) для устойчивости.
- Для отладки используйте точки останова в IntelliJ IDEA и проверяйте содержимое `target/allure-results`.

## Источники
- [Allure IntelliJ Plugin](https://plugins.jetbrains.com/plugin/10407-allure)
- [Allure Command Line Documentation](https://docs.qameta.io/allure/#_commandline)
- [Allure Maven Integration](https://docs.qameta.io/allure/#_maven)
- [Allure Framework Documentation](https://docs.qameta.io/allure/)

---
[**← Назад к оглавлению**](README.md)
