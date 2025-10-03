
# Интеграция Allure с GitHub Actions: отчёты прямо в workflow

Далее мы рассмотрим настройку Allure Framework в GitHub Actions для автоматической генерации и публикации отчётов в CI/CD. Это особенно полезно для open-source проектов и команд, использующих GitHub для автоматизации.

## Шаги настройки

### 1. Настройка Maven

Добавьте зависимости для Allure в `pom.xml`. Включены модули для JUnit 5 и TestNG, если оба фреймворка используются в проекте.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>allure-github-actions</artifactId>
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

### 2. Настройка allure.properties

Создайте файл `src/test/resources/allure.properties` для указания директории результатов:

```properties
allure.results.directory=target/allure-results
allure.attachments.screenshots=true
allure.logging.level=INFO
```

### 3. Настройка workflow в GitHub Actions

Создайте файл `.github/workflows/tests.yml`:

```yaml
name: Run Tests with Allure
on:
  push:
    branches:
      - main
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
      - name: Run tests
        run: mvn clean test
      - name: Generate Allure Report
        run: mvn allure:report
      - name: Upload Allure Report
        uses: actions/upload-artifact@v3
        with:
          name: allure-report
          path: target/site/allure-maven-plugin
      - name: Publish to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: target/site/allure-maven-plugin
```

- `mvn allure:report` генерирует HTML-отчёт.
- `actions/upload-artifact` сохраняет отчёт как артефакт.
- `actions-gh-pages` публикует отчёт на GitHub Pages.

### 4. Публикация отчёта

- После выполнения workflow отчёт доступен в разделе **Artifacts** на странице Actions.
- Если настроен GitHub Pages, отчёт публикуется по URL вида `https://<username>.github.io/<repository>/`.
- Альтернативно, отчёты можно загружать в S3, FTP или внешний сервер с помощью дополнительных шагов.

### Пример теста с Allure

Пример API-теста с RestAssured и TestNG (подробно описан в предыдущих статьях):

```java
import io.qameta.allure.Description;
import io.qameta.allure.Severity;
import io.qameta.allure.SeverityLevel;
import io.restassured.RestAssured;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static org.assertj.core.api.Assertions.assertThat;

public class AllureApiTest {

    @Test
    @Description("Проверка API эндпоинта")
    @Severity(SeverityLevel.NORMAL)
    public void testApiEndpoint() {
        // Arrange
        RestAssured.baseURI = "https://api.example.com";
        
        // Act
        int statusCode = given()
            .when()
            .get("/items")
            .then()
            .extract()
            .statusCode();
        
        // Assert
        assertThat(statusCode).isEqualTo(200);
    }
}
```

## Советы по интеграции

- **Артефакты**: Сохраняйте `target/allure-results` как артефакт для последующего анализа.
- **История отчётов**: Используйте `allure-history` для отслеживания стабильности тестов. Скопируйте `history` из предыдущих билдов в `target/allure-results/history` перед генерацией отчёта.
- **Уведомления**: Настройте отправку результатов в Telegram или Slack с помощью actions, например, `slack-send`.
- **Flaky тесты**: Используйте механизм повторов (например, TestNG `@Retry`) и прикрепляйте скриншоты/логи для упрощения анализа.

## Источники
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Allure Framework Documentation](https://docs.qameta.io/allure/)
- [Allure Maven Documentation](https://docs.qameta.io/allure/#_maven)
- [Allure Command Line Guide](https://docs.qameta.io/allure/#_commandline)

---
[**← Назад к оглавлению**](README.md)
