# Работа с чувствительными данными

## Хранение паролей и токенов

Чувствительные данные, такие как пароли, API-токены или ключи, должны храниться в зашифрованном виде или в специализированных секрет-хранилищах. Использование открытого текста в коде или конфигурационных файлах недопустимо, так как это создаёт риск утечки данных. 

Рекомендуемые подходы:
- **Секрет-хранилища**: Используйте инструменты вроде HashiCorp Vault, AWS Secrets Manager или Azure Key Vault для централизованного управления секретами.
- **Шифрование в файлах**: Если секреты хранятся в файлах, используйте библиотеки, такие как Jasypt, для шифрования данных. Пример:

```java
// Пример шифрования с Jasypt
import org.jasypt.util.text.BasicTextEncryptor;

public class SensitiveDataExample {
    public static void main(String[] args) {
        BasicTextEncryptor encryptor = new BasicTextEncryptor();
        encryptor.setPassword("encryptionKey"); // Ключ шифрования
        String encryptedPassword = encryptor.encrypt("mySecretPassword");
        System.out.println("Encrypted: " + encryptedPassword);
    }
}
```

- **Переменные окружения**: Храните чувствительные данные в переменных окружения, чтобы избежать их включения в репозиторий.

Пример настройки переменных окружения в JUnit 5:

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertNotNull;

public class EnvironmentVariableTest {
    @Test
    void testAccessSensitiveData() {
        // Arrange
        String apiToken = System.getenv("API_TOKEN");

        // Act
        boolean isTokenPresent = apiToken != null && !apiToken.isEmpty();

        // Assert
        assertNotNull(apiToken, "API token should be present in environment variables");
    }
}
```

Пример для TestNG:

```java
import org.testng.annotations.Test;
import static org.testng.Assert.assertNotNull;

public class EnvironmentVariableTestNG {
    @Test
    void testAccessSensitiveData() {
        // Given
        String apiToken = System.getenv("API_TOKEN");

        // When
        boolean isTokenPresent = apiToken != null && !apiToken.isEmpty();

        // Then
        assertNotNull(apiToken, "API token should be present in environment variables");
    }
}
```

## Передача данных через CI/CD

Для безопасной передачи чувствительных данных в CI/CD-системах (Jenkins, GitHub Actions, GitLab CI) используйте встроенные механизмы секретов:
- **Jenkins**: Храните секреты в Credentials Plugin. Пример использования в `Jenkinsfile`:

```groovy
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                withCredentials([string(credentialsId: 'api-token', variable: 'API_TOKEN')]) {
                    sh 'mvn test -Dapi.token=$API_TOKEN'
                }
            }
        }
    }
}
```

- **GitHub Actions**: Храните секреты в разделе Secrets. Пример `.yml`:

```yaml
name: Run Tests
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Run tests with secret
        env:
          API_TOKEN: ${{ secrets.API_TOKEN }}
        run: mvn test -Dapi.token=$API_TOKEN
```

Передача через переменные окружения исключает необходимость хардкода в скриптах и минимизирует риск утечки.

## Исключение из логов и отчётов

Чувствительные данные не должны попадать в логи и отчёты, такие как Allure. Используйте библиотеки логирования (например, SLF4J с Logback) для фильтрации или маскирования данных.

Пример настройки Logback для маскирования данных:

```xml
<!-- logback.xml -->
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <logger name="sensitive.data.logger" level="INFO">
        <appender-ref ref="CONSOLE"/>
    </logger>

    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
    </root>
</configuration>
```

Пример маскирования данных в коде:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class SensitiveDataLogger {
    private static final Logger logger = LoggerFactory.getLogger(SensitiveDataLogger.class);

    public void logSensitiveData(String token) {
        // Маскирование токена
        String maskedToken = token.replaceAll("(?<=.{4}).(?=.{4})", "*");
        logger.info("Processing token: {}", maskedToken);
    }
}
```

Для Allure используйте аннотацию `@Step` с параметром, исключающим чувствительные данные из отчёта:

```java
import io.qameta.allure.Step;

public class AllureSteps {
    @Step("Выполнение запроса с токеном")
    public void performRequest(String token) {
        // Логика без вывода token в отчёт
        String maskedToken = token.replaceAll("(?<=.{4}).(?=.{4})", "*");
        System.out.println("Request with token: " + maskedToken);
    }
}
```

## Маскирование данных в скриншотах

Для UI-тестов с Selenium WebDriver или Selenide маскируйте чувствительные данные на скриншотах. Например, используйте JavaScript для наложения маски на элементы.

Пример маскирования с Selenide:

```java
import com.codeborne.selenide.Selenide;
import com.codeborne.selenide.SelenideElement;
import static com.codeborne.selenide.Selenide.$;

public class MaskingExample {
    public void maskSensitiveField() {
        SelenideElement sensitiveField = $("#sensitive-input");
        if (sensitiveField.isDisplayed()) {
            Selenide.executeJavaScript(
                "arguments[0].setAttribute('value', '****')", sensitiveField
            );
            Selenide.screenshot("masked_screenshot");
        }
    }
}
```

Для устойчивости тестов:
- Используйте стабильные локаторы (CSS или XPath), избегая динамических ID.
- Проверяйте состояние элементов с помощью `isDisplayed()` и `isEnabled()`.
- Добавляйте явные ожидания с `Selenide.Wait()`.

Пример Page Object с маскировкой:

```java
import com.codeborne.selenide.SelenideElement;
import static com.codeborne.selenide.Condition.visible;
import static com.codeborne.selenide.Selenide.$;

public class LoginPage {
    private final SelenideElement usernameField = $("#username");
    private final SelenideElement passwordField = $("#password");

    public void enterCredentials(String username, String password) {
        usernameField.shouldBe(visible).setValue(username);
        passwordField.shouldBe(visible).setValue(password);
        // Маскирование значения в поле пароля
        Selenide.executeJavaScript("arguments[0].setAttribute('value', '****')", passwordField);
    }
}
```

Пример теста с JUnit 5:

```java
import org.junit.jupiter.api.Test;
import static com.codeborne.selenide.Selenide.open;

public class LoginTest {
    @Test
    void testLoginWithMaskedPassword() {
        // Arrange
        LoginPage loginPage = new LoginPage();
        open("https://example.com/login");

        // Act
        loginPage.enterCredentials("user", System.getenv("USER_PASSWORD"));

        // Assert
        // Проверка успешного входа
    }
}
```

Пример теста с TestNG:

```java
import org.testng.annotations.Test;
import static com.codeborne.selenide.Selenide.open;

public class LoginTestNG {
    @Test
    void testLoginWithMaskedPassword() {
        // Given
        LoginPage loginPage = new LoginPage();
        open("https://example.com/login");

        // When
        loginPage.enterCredentials("user", System.getenv("USER_PASSWORD"));

        // Then
        // Проверка успешного входа
    }
}
```

## Отладка

Для отладки в IntelliJ IDEA:
- Устанавливайте точки останова на строках с доступом к переменным окружения или секретам.
- Проверяйте значения переменных в окне Debug.
- Для UI-тестов используйте DevTools в браузере для проверки локаторов.
- Логируйте маскированные данные для анализа без риска утечки.

## Источники
- [Jasypt Documentation](http://www.jasypt.org/)
- [SLF4J Manual](http://www.slf4j.org/manual.html)
- [Selenide Documentation](https://selenide.org/documentation.html)

---
[**← Назад к оглавлению**](../README.md)