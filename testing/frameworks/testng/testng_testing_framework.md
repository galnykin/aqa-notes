# TestNG: фреймворк для модульного и интеграционного тестирования в Java

TestNG — это мощный и гибкий фреймворк, предназначенный для организации и выполнения тестов в Java. Он расширяет возможности JUnit, предоставляя более широкую систему аннотаций, поддержку параметризации, группировки, параллельного запуска и интеграции с CI/CD.

---

## Основные аннотации

| Аннотация         | Назначение                                  |
|-------------------|---------------------------------------------|
| `@Test`           | Обозначает метод как тест                   |
| `@BeforeMethod`   | Выполняется перед каждым `@Test`            |
| `@AfterMethod`    | После каждого `@Test`                       |
| `@BeforeClass`    | Один раз перед всеми методами класса        |
| `@AfterClass`     | Один раз после всех методов класса          |
| `@DataProvider`   | Подаёт параметры в тест                     |
| `@Parameters`     | Получает параметры из XML-файла             |

---

## Пример базового теста

```java
public class LoginTest {

    @BeforeMethod
    public void setup() {
        System.out.println("Открытие браузера");
    }

    @Test
    public void validLoginTest() {
        System.out.println("Проверка успешного входа");
    }

    @AfterMethod
    public void teardown() {
        System.out.println("Закрытие браузера");
    }
}
```

---

## Параметризация тестов

### Через `@DataProvider`:
```java
@DataProvider(name = "loginData")
public Object[][] loginData() {
    return new Object[][] {
        {"admin", "1234"},
        {"user", "pass"}
    };
}

@Test(dataProvider = "loginData")
public void loginTest(String username, String password) {
    // логика теста
}
```

### Через `@Parameters` и XML:
```xml
<parameter name="browser" value="chrome"/>
```

```java
@Parameters("browser")
@Test
public void openBrowser(String browser) {
    System.out.println("Открытие браузера: " + browser);
}
```

---

## Группировка и фильтрация

```java
@Test(groups = {"smoke"})
public void smokeTest() { }

@Test(groups = {"regression"})
public void regressionTest() { }
```

В `testng.xml` можно указать, какие группы запускать:
```xml
<groups>
  <run>
    <include name="smoke"/>
  </run>
</groups>
```

---

## Параллельный запуск

TestNG поддерживает параллельное выполнение:
```xml
<suite name="ParallelSuite" parallel="methods" thread-count="4">
  <test name="ParallelTest">
    <classes>
      <class name="tests.LoginTest"/>
    </classes>
  </test>
</suite>
```

---

## Интеграция с инструментами

- **Maven / Gradle** — запуск через `mvn test` или `gradle test`
- **Allure** — генерация отчётов
- **Jenkins / GitHub Actions** — запуск в CI/CD
- **Selenium / RestAssured** — для UI и API тестов

---

## Источники:

- [Официальная документация TestNG](https://testng.org/doc/)
- [TestNG на Baeldung](https://www.baeldung.com/testng)
- [Интеграция с Maven](https://maven.apache.org/surefire/maven-surefire-plugin/examples/testng.html)

---
[**← Назад к оглавлению**](../README.md)
