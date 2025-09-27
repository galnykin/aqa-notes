# Архитектура тестового автоматизированного фреймворка: слои, шаблоны, инструменты

Построение тестового фреймворка — это не просто набор тестов, а полноценная архитектура, включающая слои, шаблоны проектирования, инструменты логирования и репортинга. Ниже представлена структурированная модель, применимая для UI- и API-автоматизации на Java.

-----

## 🧱 Пакеты и слои фреймворка

Фреймворк делится на логические слои, каждый из которых выполняет отдельную роль, обеспечивая **низкую связанность** и **высокую переиспользуемость** кода.

### 1\. **Test Layer** (`src/test/java`)

Это самый верхний уровень, отвечающий за **бизнес-логику проверок**.

- Содержит классы с аннотациями `@Test` (JUnit 5 / TestNG).
- Тесты должны быть декларативными (описывать **ЧТО** проверяется, а не **КАК**).
- Они оперируют шагами (Steps) и моделями данных (DTO), но **никогда не обращаются напрямую** к `WebDriver` или элементам страниц.
- Содержат **ассерты** (предпочтительно **AssertJ**).

### 2\. **Steps / Business Layer** (`.../steps`)

Слой бизнес-абстракций. Он превращает действия на страницах в осмысленные пользовательские сценарии.

- Объединяет несколько действий с разных Page Objects в один шаг. Например, шаг `loginAs(user)` может включать открытие страницы логина, ввод данных и нажатие кнопки.
- Повышает читаемость тестов и позволяет переиспользовать целые сценарии.

### 3\. **API Layer** (`.../api`)

Этот слой существует параллельно с Page Layer и отвечает за взаимодействие с API.

- Содержит API-клиенты (например, `UserApiClient`, `CartApiClient`), построенные с помощью **Rest Assured**.
- Инкапсулирует логику отправки запросов (endpoints, headers, body) и парсинга ответов.
- Используется в API-тестах или для подготовки состояния в UI-тестах (например, создать пользователя через API перед проверкой логина в UI).

### 4\. **Page Layer** (`.../pages`)

Реализация шаблона **Page Object**.

- Каждый класс соответствует странице или значимому компоненту интерфейса (например, `HeaderComponent`).
- Содержит **локаторы элементов** (в виде `private final By`) и **методы для взаимодействия** с ними.
- Методы должны возвращать либо `void`, либо другой Page Object для реализации **Fluent Interface**.

### 5\. **DTO / Domain Layer** (`.../dto`)

Слой моделей данных.

- Описывает бизнес-сущности приложения: `User`, `Product`, `Order`.
- Обычно это POJO (Plain Old Java Object) классы. Для их создания идеально подходит **Lombok** (`@Data`, `@Builder`).
- Используется для передачи данных между слоями (например, передать объект `User` в `loginAs(user)`).

### 6\. **Core Layer** (`.../core`)

Инфраструктурное ядро фреймворка.

- Управление `WebDriver` (например, **WebDriverFactory** с **ThreadLocal** для параллельного запуска).
- Базовые классы (`BaseTest`, `BasePage`), содержащие общую логику `setUp` и `tearDown`.
- Конфигурация фреймворка (чтение `.properties` файлов).

### 7\. **Utils & Data Layers** (`.../utils`, `.../data`)

- **Utils**: Вспомогательные классы-утилиты (генераторы случайных данных, работа с датами, файлами).
- **Data**: Классы для поставки тестовых данных (например, чтение из CSV/JSON файлов для параметризованных тестов).

-----

## 🧩 Шаблоны проектирования

### Page Object с Fluent Interface

Классический шаблон, где методы возвращают экземпляр следующей страницы, позволяя выстраивать цепочки вызовов.

```java
// Вместо:
// loginPage.enterUsername("user");
// loginPage.enterPassword("pass");
// loginPage.clickSubmit();

// Используем Fluent Interface:
homePage = loginPage.loginAs("user", "pass");
```

**`PageFactory`** с аннотацией `@FindBy` — это одна из реализаций PO, но она считается устаревшей из-за проблем с отладкой, производительностью и гибкостью. Современный подход — использовать `private final By` локаторы.

### Factory + ThreadLocal (вместо Singleton)

**Singleton** — анти-паттерн в автоматизации, так как он не обеспечивает потокобезопасность при параллельном запуске тестов. Вместо него используется **Factory** для создания экземпляров `WebDriver` и **ThreadLocal** для хранения уникального экземпляра для каждого потока.

```java
public class WebDriverFactory {
    private static final ThreadLocal<WebDriver> driver = new ThreadLocal<>();

    public static WebDriver getDriver() { /* ... */ }
    public static void quitDriver() { /* ... */ }
}
```

### Builder

Идеален для создания сложных объектов данных (DTO) в тестах. **Lombok** упрощает его реализацию до одной аннотации.

```java
@Builder @Data
public class User {
    private String username;
    private String password;
}

// В тесте:
User testUser = User.builder().username("admin").password("secret123").build();
```

-----

## 📋 Логирование: SLF4J + Logback

Вместо привязки к конкретной реализации (как Log4j), современный подход — использовать фасад **SLF4J (Simple Logging Facade for Java)**. Это позволяет легко менять реализацию логирования (например, **Logback** или Log4j2) без изменения кода.

**Lombok** делает добавление логгера тривиальным.

**Пример в коде:**

```java
import lombok.extern.slf4j.Slf4j;

@Slf4j // Аннотация Lombok для создания логгера
public class LoginTest {

    @Test
    void testLogin() {
        log.info("Начало теста авторизации для пользователя 'admin'");
        try {
            // ... тестовые шаги ...
            log.debug("Проверка успешной авторизации");
            // ... ассерт ...
        } catch (Exception e) {
            log.error("Тест упал с ошибкой!", e);
        }
    }
}
```

Конфигурация логирования (`logback-test.xml`) помещается в `src/test/resources` и позволяет гибко настраивать формат вывода, уровни логирования и направление (консоль, файл).

-----

## 📊 Репортинг: Allure

**Allure** — это мощный инструмент, который превращает результаты тестов в подробный интерактивный отчёт.

**Ключевые аннотации:**

- `@Epic`, `@Feature`, `@Story`: для иерархической группировки тестов.
- `@Step`: для описания шагов внутри теста. Allure автоматически логирует шаги.
- `@Attachment`: для прикрепления к отчёту файлов (скриншоты, логи, JSON-ответы).
- `@Description`: для детального описания цели теста.

**Пример интеграции:**

```java
@Epic("User Management")
@Feature("Login")
public class LoginSteps {
    private LoginPage loginPage = new LoginPage();

    @Step("Авторизация пользователя '{user.username}'")
    public void loginAs(User user) {
        log.info("Открываем страницу логина");
        loginPage.open();
        
        log.info("Вводим креды и нажимаем 'Войти'");
        loginPage.enterUsername(user.getUsername())
                 .enterPassword(user.getPassword())
                 .clickSubmit();
        takeScreenshot(); // Пример прикрепления скриншота
    }

    @Attachment(value = "Page screenshot", type = "image/png")
    public byte[] takeScreenshot() {
        // Логика создания скриншота с помощью Selenium
        return ((TakesScreenshot) WebDriverFactory.getDriver()).getScreenshotAs(OutputType.BYTES);
    }
}
```

В отчёте Allure будет виден каждый шаг с вложенными скриншотами, что делает анализ упавших тестов быстрым и удобным.

-----

## 🔗 Источники:

- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [SLF4J and Logback Documentation](https://www.google.com/search?q=http://logback.qos.ch/documentation.html)
- [Allure Framework](https://docs.qameta.io/allure/)
- [Refactoring Guru — Design Patterns](https://refactoring.guru/ru/design-patterns)
- [Baeldung — Introduction to Lombok](https://www.baeldung.com/intro-to-project-lombok)

-----

[**← Назад к оглавлению**](../README.md)