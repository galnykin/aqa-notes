# Работа с чувствительными данными

Работа с чувствительными данными, такими как пароли и токены, требует особого внимания к безопасности. В автоматизации тестирования важно обеспечить их хранение в зашифрованном виде, безопасную передачу и исключение из логов и отчётов. В статье описаны подходы к этим задачам с использованием Java, Maven, Jenkins и других инструментов.

## Хранение чувствительных данных

Чувствительные данные, такие как пароли и токены, не должны храниться в коде или конфигурационных файлах в открытом виде. Вместо этого используются зашифрованные хранилища или секрет-хранилища.

### Использование секрет-хранилищ

Для хранения данных рекомендуется применять секрет-хранилища, такие как **HashiCorp Vault** или встроенные механизмы CI/CD-систем (например, Jenkins Credentials, GitHub Secrets). Vault предоставляет безопасное хранение и доступ к данным через API.

#### Пример интеграции с Vault

Зависимости в `pom.xml` для работы с Vault:

```xml
<dependencies>
    <dependency>
        <groupId>com.bettercloud</groupId>
        <artifactId>vault-java-driver</artifactId>
        <version>5.1.0</version>
    </dependency>
</dependencies>
```

Пример получения секрета из Vault:

```java
import com.bettercloud.vault.Vault;
import com.bettercloud.vault.VaultConfig;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class VaultClient {
    private static final Logger logger = LoggerFactory.getLogger(VaultClient.class);

    public String getSecret(String vaultUrl, String token, String secretPath) {
        try {
            VaultConfig config = new VaultConfig()
                    .address(vaultUrl)
                    .token(token)
                    .build();
            Vault vault = new Vault(config);
            String secret = vault.logical()
                    .read(secretPath)
                    .getData()
                    .get("password");
            logger.info("Successfully retrieved secret from Vault");
            return secret;
        } catch (Exception e) {
            logger.error("Failed to retrieve secret from Vault: {}", e.getMessage());
            throw new RuntimeException("Vault access error", e);
        }
    }
}
```

### Локальное хранение в зашифрованном виде

Для локального хранения можно использовать библиотеки шифрования, такие как Jasypt. Данные шифруются и хранятся в конфигурационных файлах.

#### Зависимости для Jasypt

```xml
<dependencies>
    <dependency>
        <groupId>org.jasypt</groupId>
        <artifactId>jasypt</artifactId>
        <version>1.9.3</version>
    </dependency>
</dependencies>
```

#### Пример шифрования и дешифрования

```java
import org.jasypt.util.text.BasicTextEncryptor;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class EncryptionUtil {
    private static final Logger logger = LoggerFactory.getLogger(EncryptionUtil.class);
    private final BasicTextEncryptor encryptor;

    public EncryptionUtil(String encryptionPassword) {
        encryptor = new BasicTextEncryptor();
        encryptor.setPassword(encryptionPassword);
    }

    public String encrypt(String data) {
        logger.info("Encrypting sensitive data");
        return encryptor.encrypt(data);
    }

    public String decrypt(String encryptedData) {
        logger.info("Decrypting sensitive data");
        return encryptor.decrypt(encryptedData);
    }
}
```

Использование в коде:

```java
public class SensitiveDataExample {
    private static final Logger logger = LoggerFactory.getLogger(SensitiveDataExample.class);

    public void useEncryptedData() {
        String encryptionPassword = System.getenv("ENCRYPTION_PASSWORD");
        EncryptionUtil encryptor = new EncryptionUtil(encryptionPassword);
        String password = "mySecretPassword";
        String encryptedPassword = encryptor.encrypt(password);
        logger.info("Encrypted password: {}", encryptedPassword);
        String decryptedPassword = encryptor.decrypt(encryptedPassword);
        logger.info("Decrypted password: [PROTECTED]");
    }
}
```

## Передача через переменные окружения или CI-секреты

Передача чувствительных данных через переменные окружения или секреты CI/CD-систем — стандартная практика для обеспечения безопасности.

### Переменные окружения

Переменные окружения задаются на уровне операционной системы или в CI/CD-системе. В Java они доступны через `System.getenv()`.

#### Пример использования переменных окружения

```java
public class EnvironmentConfig {
    private static final Logger logger = LoggerFactory.getLogger(EnvironmentConfig.class);

    public String getApiToken() {
        String token = System.getenv("API_TOKEN");
        if (token == null) {
            logger.error("API_TOKEN environment variable is not set");
            throw new IllegalStateException("API_TOKEN is missing");
        }
        logger.info("API token retrieved from environment");
        return token;
    }
}
```

### Настройка Jenkins Credentials

1. В Jenkins перейдите в **Manage Jenkins → Manage Plugins** и установите плагин **Credentials Plugin**.
2. В **Manage Jenkins → Manage Credentials** добавьте новый секрет (например, `API_TOKEN`).
3. В задаче Jenkins настройте доступ к секрету:

```groovy
pipeline {
    agent any
    environment {
        API_TOKEN = credentials('api-token-id')
    }
    stages {
        stage('Test') {
            steps {
                sh 'mvn clean test'
            }
        }
    }
}
```

В тестах используйте `System.getenv("API_TOKEN")` для получения значения.

### Настройка GitHub Actions

В GitHub Actions секреты задаются в настройках репозитория (**Settings → Secrets and variables → Actions**).

#### Пример workflow для GitHub Actions

```yaml
name: Run Tests
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
      - name: Run tests
        env:
          API_TOKEN: ${{ secrets.API_TOKEN }}
        run: mvn clean test
```

Секреты передаются в переменные окружения и доступны в тестах.

## Исключение чувствительных данных из логов и отчётов

Чувствительные данные не должны попадать в логи или отчёты Allure. Для этого применяются следующие практики:

1. **Маскировка данных в логах**:
   - Используйте заглушки, такие как `[PROTECTED]`, вместо реальных значений.
   - Пример в Page Object:

```java
import io.qameta.allure.Step;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LoginPage {
    private static final Logger logger = LoggerFactory.getLogger(LoginPage.class);
    private final WebDriver driver;

    @FindBy(id = "password")
    private WebElement passwordField;

    public LoginPage(WebDriver driver) {
        this.driver = driver;
    }

    @Step("Entering password")
    public void enterPassword(String password) {
        logger.info("Entering password: [PROTECTED]");
        passwordField.sendKeys(password);
    }
}
```

2. **Фильтрация в Allure**:
   - Используйте аннотацию `@Step` с безопасными описаниями, избегая передачи чувствительных данных.
   - Настройте Allure для исключения параметров, если они содержат чувствительные данные, через кастомный `AllureListener`.

3. **Логирование в RestAssured**:
   - Настройте фильтр для маскировки данных в логах API-запросов:

```java
import io.restassured.RestAssured;
import io.restassured.filter.Filter;
import io.restassured.filter.FilterContext;
import io.restassured.response.Response;
import io.restassured.specification.FilterableRequestSpecification;
import io.restassured.specification.FilterableResponseSpecification;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class SensitiveDataFilter implements Filter {
    private static final Logger logger = LoggerFactory.getLogger(SensitiveDataFilter.class);

    @Override
    public Response filter(FilterableRequestSpecification requestSpec, FilterableResponseSpecification responseSpec, FilterContext ctx) {
        logger.info("Sending request to: {}", requestSpec.getURI());
        requestSpec.removeHeader("Authorization"); // Удаляем токен из логов
        Response response = ctx.next(requestSpec, responseSpec);
        logger.info("Response received: [PROTECTED]");
        return response;
    }
}
```

Использование фильтра:

```java
RestAssured.filters(new SensitiveDataFilter());
RestAssured.given()
    .header("Authorization", "Bearer " + System.getenv("API_TOKEN"))
    .get("/api/endpoint")
    .then()
    .statusCode(200);
```

## Отладка и устойчивость

1. **Отладка в IntelliJ IDEA**:
   - Установите точки останова в местах использования чувствительных данных.
   - Проверяйте значения переменных окружения через `System.getenv()` в дебаггере.
   - Используйте логи для подтверждения корректности получения данных.

2. **Устойчивость тестов**:
   - Используйте `WebDriverWait` для ожидания элементов, чтобы избежать ошибок из-за задержек.
   - Пример:

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.time.Duration;

public class WaitUtil {
    private static final Logger logger = LoggerFactory.getLogger(WaitUtil.class);

    public void waitForElement(WebDriver driver, WebElement element) {
        logger.info("Waiting for element to be visible");
        new WebDriverWait(driver, Duration.ofSeconds(10))
                .until(d -> element.isDisplayed());
        logger.info("Element is visible");
    }
}
```

3. **Стабильные локаторы**:
   - Используйте `id` или `data-*` атрибуты для надёжного поиска элементов.
   - Проверяйте элементы через `isDisplayed()` и `isEnabled()` перед действиями.

## Распространённые ошибки

1. **Хранение в коде**: Пароли или токены в коде или конфигурационных файлах без шифрования.
2. **Логирование данных**: Случайное логирование чувствительных данных в консоль или отчёты.
3. **Незащищённый доступ**: Отсутствие проверки прав доступа к секрет-хранилищу.
4. **Неправильные переменные окружения**: Ошибки в именовании или отсутствии переменных в CI/CD.

## Дополнительно

- **Testcontainers для Vault**:
  - Используйте Testcontainers для локального тестирования интеграции с Vault:

```xml
<dependencies>
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>testcontainers</artifactId>
        <version>1.20.2</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>vault</artifactId>
        <version>1.20.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

```java
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testcontainers.vault.VaultContainer;

public class VaultTest {
    private static final Logger logger = LoggerFactory.getLogger(VaultTest.class);

    @Test
    void testVaultIntegration() {
        VaultContainer<?> vaultContainer = new VaultContainer<>("vault:1.13.0")
                .withVaultToken("test-token")
                .withSecretInVault("secret/test", "password=testpass");
        vaultContainer.start();
        logger.info("Vault container started at: {}", vaultContainer.getHttpHostAddress());
        // Логика получения секрета
        vaultContainer.close();
    }
}
```

- **OWASP Dependency-Check**:
  - Добавьте проверку зависимостей для безопасности:

```xml
<plugin>
    <groupId>org.owasp</groupId>
    <artifactId>dependency-check-maven</artifactId>
    <version>10.0.4</version>
    <executions>
        <execution>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

## Источники
- [HashiCorp Vault](https://www.vaultproject.io/)
- [Jasypt Documentation](http://www.jasypt.org/)
- [Jenkins Credentials Plugin](https://plugins.jenkins.io/credentials/)
- [GitHub Actions Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

---
[**← Назад к оглавлению**](../README.md)