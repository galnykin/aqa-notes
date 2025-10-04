# Параметризация тестов

Параметризация тестов позволяет запускать один и тот же тест с разными входными данными, что увеличивает покрытие и упрощает поддержку тестового кода. В Java для автоматизации тестирования часто используют аннотацию `@DataProvider` в TestNG или `@ParameterizedTest` в JUnit 5. В статье рассматривается использование `@DataProvider` для передачи данных, включая поддержку внешних источников (CSV, JSON, Excel), с акцентом на лучшие практики и устойчивость тестов.

## Основы параметризации с `@DataProvider`

`@DataProvider` в TestNG позволяет создавать наборы данных для тестов. Метод, помеченный этой аннотацией, возвращает двумерный массив `Object[][]`, где каждый вложенный массив представляет набор параметров для одного прогона теста. Название провайдера данных должно быть осмысленным и отражать его назначение.

### Пример кода (TestNG)

```java
import org.testng.annotations.DataProvider;
import org.testng.annotations.Test;

public class LoginTest {

    @Test(dataProvider = "loginData")
    public void testLogin(String username, String password, boolean expectedSuccess) {
        // Arrange
        // Настройка теста, например, инициализация WebDriver
        // Act
        boolean loginResult = performLogin(username, password); // Метод логина
        // Assert
        Assert.assertEquals(loginResult, expectedSuccess, "Login result does not match expected");
    }

    @DataProvider(name = "loginData")
    public Object[][] provideLoginData() {
        return new Object[][] {
                {"validUser", "validPass", true},
                {"invalidUser", "invalidPass", false},
                {"validUser", "", false}
        };
    }

    private boolean performLogin(String username, String password) {
        // Логика логина, например, с использованием Selenium
        return username.equals("validUser") && password.equals("validPass");
    }
}
```

### Лучшие практики
- **Осмысленные имена**: Называйте `@DataProvider` так, чтобы было понятно, какие данные он предоставляет (например, `loginData`, `userCredentials`).
- **Разделение данных и логики**: Храните данные отдельно от тестового кода, используя внешние источники (CSV, JSON, Excel) для сложных наборов данных.
- **Покрытие граничных случаев**: Включайте в данные позитивные, негативные и граничные сценарии.

## Использование внешних источников данных

Для сложных тестов данные лучше хранить во внешних файлах (CSV, JSON, Excel), чтобы упростить их обновление и управление. Рассмотрим примеры чтения данных из этих источников.

### Чтение из CSV

Для работы с CSV используйте библиотеку `OpenCSV`. Добавьте зависимость в `pom.xml`:

```xml
<dependency>
    <groupId>com.opencsv</groupId>
    <artifactId>opencsv</artifactId>
    <version>5.9</version>
</dependency>
```

#### Пример кода (TestNG с CSV)

```java
import com.opencsv.CSVReader;
import com.opencsv.exceptions.CsvValidationException;
import org.testng.annotations.DataProvider;
import org.testng.annotations.Test;

import java.io.FileReader;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

public class LoginTestWithCsv {

    @Test(dataProvider = "csvLoginData")
    public void testLogin(String username, String password, String expectedSuccess) {
        // Arrange
        // Настройка теста
        // Act
        boolean loginResult = performLogin(username, password);
        // Assert
        Assert.assertEquals(loginResult, Boolean.parseBoolean(expectedSuccess), "Login result does not match expected");
    }

    @DataProvider(name = "csvLoginData")
    public Object[][] provideCsvLoginData() {
        List<Object[]> data = new ArrayList<>();
        try (CSVReader reader = new CSVReader(new FileReader("src/test/resources/login_data.csv"))) {
            String[] line;
            reader.readNext(); // Пропускаем заголовок
            while ((line = reader.readNext()) != null) {
                data.add(new Object[]{line[0], line[1], line[2]});
            }
        } catch (IOException | CsvValidationException e) {
            throw new RuntimeException("Error reading CSV file", e);
        }
        return data.toArray(new Object[0][]);
    }

    private boolean performLogin(String username, String password) {
        // Логика логина
        return username.equals("validUser") && password.equals("validPass");
    }
}
```

Пример содержимого `login_data.csv`:

```csv
username,password,expectedSuccess
validUser,validPass,true
invalidUser,invalidPass,false
validUser,,false
```

### Чтение из JSON

Для работы с JSON используйте библиотеку `Jackson`. Добавьте зависимость в `pom.xml`:

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.17.2</version>
</dependency>
```

#### Пример кода (TestNG с JSON)

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import org.testng.annotations.DataProvider;
import org.testng.annotations.Test;

import java.io.File;
import java.io.IOException;
import java.util.Arrays;

public class LoginTestWithJson {

    static class LoginData {
        public String username;
        public String password;
        public boolean expectedSuccess;
    }

    @Test(dataProvider = "jsonLoginData")
    public void testLogin(String username, String password, boolean expectedSuccess) {
        // Arrange
        // Настройка теста
        // Act
        boolean loginResult = performLogin(username, password);
        // Assert
        Assert.assertEquals(loginResult, expectedSuccess, "Login result does not match expected");
    }

    @DataProvider(name = "jsonLoginData")
    public Object[][] provideJsonLoginData() {
        ObjectMapper mapper = new ObjectMapper();
        try {
            LoginData[] data = mapper.readValue(new File("src/test/resources/login_data.json"), LoginData[].class);
            return Arrays.stream(data)
                    .map(d -> new Object[]{d.username, d.password, d.expectedSuccess})
                    .toArray(Object[][]::new);
        } catch (IOException e) {
            throw new RuntimeException("Error reading JSON file", e);
        }
    }

    private boolean performLogin(String username, String password) {
        // Логика логина
        return username.equals("validUser") && password.equals("validPass");
    }
}
```

Пример содержимого `login_data.json`:

```json
[
    {"username": "validUser", "password": "validPass", "expectedSuccess": true},
    {"username": "invalidUser", "password": "invalidPass", "expectedSuccess": false},
    {"username": "validUser", "password": "", "expectedSuccess": false}
]
```

### Чтение из Excel

Для работы с Excel используйте `Apache POI`. Добавьте зависимости в `pom.xml`:

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>5.3.0</version>
</dependency>
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.3.0</version>
</dependency>
```

#### Пример кода (TestNG с Excel)

```java
import org.apache.poi.ss.usermodel.*;
import org.testng.annotations.DataProvider;
import org.testng.annotations.Test;

import java.io.FileInputStream;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

public class LoginTestWithExcel {

    @Test(dataProvider = "excelLoginData")
    public void testLogin(String username, String password, boolean expectedSuccess) {
        // Arrange
        // Настройка теста
        // Act
        boolean loginResult = performLogin(username, password);
        // Assert
        Assert.assertEquals(loginResult, expectedSuccess, "Login result does not match expected");
    }

    @DataProvider(name = "excelLoginData")
    public Object[][] provideExcelLoginData() {
        List<Object[]> data = new ArrayList<>();
        try (FileInputStream fis = new FileInputStream("src/test/resources/login_data.xlsx")) {
            Workbook workbook = WorkbookFactory.create(fis);
            Sheet sheet = workbook.getSheetAt(0);
            for (Row row : sheet) {
                if (row.getRowNum() == 0) continue; // Пропускаем заголовок
                String username = row.getCell(0).getStringCellValue();
                String password = row.getCell(1).getStringCellValue();
                boolean expectedSuccess = row.getCell(2).getBooleanCellValue();
                data.add(new Object[]{username, password, expectedSuccess});
            }
        } catch (IOException e) {
            throw new RuntimeException("Error reading Excel file", e);
        }
        return data.toArray(new Object[0][]);
    }

    private boolean performLogin(String username, String password) {
        // Логика логина
        return username.equals("validUser") && password.equals("validPass");
    }
}
```

Пример структуры `login_data.xlsx`:

| username   | password    | expectedSuccess |
|------------|-------------|-----------------|
| validUser  | validPass   | TRUE            |
| invalidUser| invalidPass | FALSE           |
| validUser  |             | FALSE           |

## Интеграция с CI/CD

Для запуска параметризованных тестов в CI/CD (например, Jenkins или GitHub Actions) настройте выполнение Maven-команды `mvn test`. Убедитесь, что внешние файлы данных (CSV, JSON, Excel) находятся в репозитории и доступны для тестов.

### Пример настройки GitHub Actions

```yaml
name: Run Tests
on:
  push:
    branches:
      - main
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
      - name: Run tests
        run: mvn clean test
```

### Пример настройки Jenkins

1. Создайте новый проект в Jenkins.
2. Настройте SCM (Git) для клонирования репозитория.
3. Добавьте шаг сборки: `Execute shell` с командой `mvn clean test`.
4. Настройте Allure для генерации отчётов:
   ```bash
   mvn allure:report
   ```

## Дополнительно

### Использование Allure для отчётов

Allure позволяет создавать информативные отчёты для параметризованных тестов. Добавьте зависимости в `pom.xml`:

```xml
<dependency>
    <groupId>io.qameta.allure</groupId>
    <artifactId>allure-testng</artifactId>
    <version>2.27.0</version>
</dependency>
```

Пример теста с Allure:

```java
import io.qameta.allure.Description;
import io.qameta.allure.Step;
import org.testng.annotations.DataProvider;
import org.testng.annotations.Test;

public class LoginTestWithAllure {

    @Test(dataProvider = "loginData")
    @Description("Тест логина с разными учетными данными")
    public void testLogin(String username, String password, boolean expectedSuccess) {
        loginStep(username, password);
        Assert.assertEquals(performLogin(username, password), expectedSuccess, "Login result does not match expected");
    }

    @Step("Выполнение логина с username: {0}, password: {1}")
    private void loginStep(String username, String password) {
        // Логика шага
    }

    @DataProvider(name = "loginData")
    public Object[][] provideLoginData() {
        return new Object[][] {
                {"validUser", "validPass", true},
                {"invalidUser", "invalidPass", false}
        };
    }

    private boolean performLogin(String username, String password) {
        return username.equals("validUser") && password.equals("validPass");
    }
}
```

### Использование Testcontainers

Для тестирования API или UI, зависящих от внешних сервисов, используйте Testcontainers для запуска сервисов в Docker-контейнерах.

Пример зависимости в `pom.xml`:

```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers</artifactId>
    <version>1.20.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>1.20.2</version>
    <scope>test</scope>
</dependency>
```

Пример использования Testcontainers для запуска браузера:

```java
import org.junit.jupiter.api.Test;
import org.openqa.selenium.remote.RemoteWebDriver;
import org.testcontainers.containers.BrowserWebDriverContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

@Testcontainers
public class BrowserTest {

    @Container
    public BrowserWebDriverContainer<?> chrome = new BrowserWebDriverContainer<>()
            .withCapabilities(new ChromeOptions());

    @Test
    void testBrowser() {
        RemoteWebDriver driver = (RemoteWebDriver) chrome.getWebDriver();
        driver.get("https://example.com");
        // Проверки
        driver.quit();
    }
}
```

### Устойчивость тестов

- **Ожидания**: Используйте `WebDriverWait` для UI-тестов, чтобы дождаться видимости элементов:
  ```java
  WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
  wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("loginButton")));
  ```
- **Стабильные локаторы**: Предпочитайте `id` или `data-*` атрибуты, избегайте длинных XPath.
- **Проверки**: Используйте `isDisplayed()` и `isEnabled()` для проверки состояния элементов.

### Отладка

Для отладки в IntelliJ IDEA:
1. Установите точки останова в тестовом коде или методе `@DataProvider`.
2. Запустите тесты в режиме Debug.
3. Проверяйте значения параметров и состояние элементов через Variables или Watches.

## Распространённые ошибки

- **Неправильная структура данных**: Убедитесь, что метод `@DataProvider` возвращает `Object[][]`, и количество параметров в тесте совпадает с данными.
- **Ошибки чтения файлов**: Проверяйте пути к файлам (CSV, JSON, Excel) и их наличие в репозитории.
- **Flaky тесты**: Используйте ожидания и стабильные локаторы, чтобы минимизировать нестабильность.

## Источники
- [TestNG Documentation](https://testng.org/doc/documentation-main.html#parameters-dataproviders)
- [OpenCSV Documentation](http://opencsv.sourceforge.net/)
- [Jackson Documentation](https://github.com/FasterXML/jackson)
- [Apache POI Documentation](https://poi.apache.org/)
- [Allure Framework](https://allurereport.org/docs/)
- [Testcontainers Documentation](https://testcontainers.org/)

---
[**← Назад к оглавлению**](../README.md)