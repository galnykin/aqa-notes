# Архитектура тестового автоматизированного фреймворка: слои, шаблоны, инструменты

Построение тестового фреймворка — это не просто набор тестов, а полноценная архитектура, включающая слои, шаблоны проектирования, инструменты логирования и репортинга. Ниже представлена структурированная модель, применимая для UI- и API-автоматизации на Java.

---

## 🧱 Пакеты и слои фреймворка

Фреймворк делится на логические слои, каждый из которых выполняет отдельную роль:

### 1. **Test Layer** (`src/test/java`)
- Содержит классы с аннотацией `@Test`
- Группировка по функциональности: `LoginTest`, `CartTest`, `SearchTest`
- Использует шаги, страницы и данные

### 2. **Page Layer** (`src/main/java/.../pages`)
- Реализация Page Object
- Описание элементов и действий
- Может использовать `PageFactory` для инициализации

### 3. **Steps Layer** (`.../steps`)
- Абстракция над действиями
- Комплексные шаги: `fillLoginFormStep()`, `searchForProduct()`
- Повышает читаемость и переиспользуемость

### 4. **Domain Layer** (`.../domain`)
- Модели данных: `User`, `Product`, `Order`
- Используется для передачи параметров между слоями

### 5. **Utils Layer** (`.../utils`)
- Вспомогательные классы: генераторы, конвертеры, валидаторы
- Работа с датами, строками, JSON, файлами

### 6. **Core Layer** (`.../core`)
- Базовые компоненты: `WebDriverSingleton`, `BaseTest`, `BasePage`
- Настройка окружения, ожидания, конфигурации

### 7. **Data Layer** (`.../data`)
- Источники данных: CSV, JSON, Excel
- Реализация `DataProvider` для параметризации тестов

---

## 🧩 Шаблоны проектирования

### PageFactory
- Автоматическая инициализация `WebElement` через аннотацию `@FindBy`
- Упрощает работу с элементами
```java
@FindBy(id = "login") WebElement loginField;
PageFactory.initElements(driver, this);
```

### Singleton
- Управление WebDriver через единый доступ
```java
public class WebDriverSingleton {
    private static WebDriver driver;
    public static WebDriver getDriver() { ... }
    public static void quit() { ... }
}
```

### Chain of Responsibility
- Цепочка вызовов в Page Object
```java
public HomePage clickAcceptCookies() {
    click(ACCEPT_BUTTON);
    return this;
}
```

### Steps
- Вынос действий в отдельный класс
```java
public class LoginSteps {
    public void loginAs(User user) { ... }
}
```

### Domain Model
- Описание сущностей
```java
public class Product {
    private String name;
    private double price;
}
```

### DataProvider
- Параметризация тестов
```java
@ParameterizedTest
@CsvSource({"admin,1234", "user,pass"})
void testLogin(String username, String password) { ... }
```

---

## 📋 Логирование: Log4j

**Log4j** — библиотека для ведения логов:
- Уровни: `INFO`, `DEBUG`, `ERROR`, `WARN`
- Конфигурация через `log4j2.xml` или `log4j.properties`
- Вывод в консоль, файл, HTML

Пример:
```java
private static final Logger logger = LogManager.getLogger(LoginTest.class);

@Test
void testLogin() {
    logger.info("Начало теста авторизации");
    loginPage.login("admin", "1234");
    logger.debug("Проверка отображения приветствия");
    assertTrue(homePage.isWelcomeVisible());
}
```

Логирование помогает отслеживать шаги, ошибки и поведение тестов.

---

## 📊 Репортинг: Allure

**Allure** — инструмент для генерации отчётов:
- Поддержка JUnit, TestNG, Selenide
- Аннотации: `@Step`, `@Attachment`, `@Description`
- Отчёт включает шаги, скриншоты, логи, параметры

Пример:
```java
@Step("Авторизация пользователя: {username}")
public void login(String username, String password) { ... }

@Attachment(value = "Скриншот", type = "image/png")
public byte[] takeScreenshot() { ... }
```

Allure позволяет визуализировать результаты тестов, выявлять ошибки и делиться отчётами с командой.

---

## 🔗 Источники:
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [Log4j Documentation](https://logging.apache.org/log4j/2.x/manual/)
- [Allure Framework](https://docs.qameta.io/allure/)
- [Refactoring Guru — Design Patterns](https://refactoring.guru/ru/design-patterns)

---
[**← Назад к оглавлению**](../../../README.md)