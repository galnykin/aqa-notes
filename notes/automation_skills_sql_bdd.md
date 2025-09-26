# Дополнительные навыки автоматизатора: SQL и BDD с Cucumber

Автоматизатор — это не просто специалист по написанию тестов. Это инженер, который понимает архитектуру приложения, умеет работать с данными и владеет современными подходами к разработке. Ниже представлены два ключевых навыка, которые расширяют возможности автоматизатора: работа с базами данных через SQL и разработка тестов на основе поведения (BDD) с использованием фреймворка Cucumber.

---

## 🗃️ Основы языка SQL и работы с базами данных

SQL (Structured Query Language) — это язык для работы с реляционными базами данных. Он позволяет извлекать, добавлять, обновлять и удалять данные, а также управлять структурой таблиц.

### Основные команды SQL:

#### 1. **SELECT** — извлечение данных:
```sql
SELECT * FROM users WHERE role = 'admin';
```

#### 2. **INSERT** — добавление данных:
```sql
INSERT INTO products (name, price) VALUES ('Pizza', 9.99);
```

#### 3. **UPDATE** — обновление данных:
```sql
UPDATE users SET password = 'newpass' WHERE id = 1;
```

#### 4. **DELETE** — удаление данных:
```sql
DELETE FROM orders WHERE status = 'cancelled';
```

### Почему это важно для автоматизатора:
- Проверка данных после действий в UI
- Подготовка тестовых данных
- Очистка базы перед тестами
- Валидация бизнес-логики на уровне хранения

Инструменты: DBeaver, MySQL Workbench, pgAdmin, JDBC (для Java-интеграции)

---

## 🧠 BDD и фреймворк Cucumber

**BDD (Behavior-Driven Development)** — это подход к разработке, при котором требования описываются в виде сценариев поведения, понятных как бизнесу, так и разработчикам.

### Основные принципы BDD:
- Общий язык между бизнесом и командой
- Сценарии описываются в формате **Gherkin**
- Тесты читаются как документация

### Пример сценария на Gherkin:
```gherkin
Feature: Авторизация

  Scenario: Успешный вход
    Given пользователь находится на странице логина
    When он вводит корректные имя и пароль
    And нажимает кнопку "Войти"
    Then он должен увидеть приветствие "Добро пожаловать"
```

### Cucumber в Java:
- Сценарии хранятся в `.feature` файлах
- Каждому шагу соответствует Java-метод с аннотацией:
```java
@Given("пользователь находится на странице логина")
public void openLoginPage() {
    loginPage.open();
}
```

### Преимущества:
- Повышает читаемость тестов
- Упрощает коммуникацию с заказчиком
- Позволяет строить тесты на уровне бизнес-логики

Интеграция: JUnit/TestNG, Selenium, RestAssured, Allure

---

## 🔗 Источники:
- [SQL Tutorial — W3Schools](https://www.w3schools.com/sql/)
- [Cucumber Documentation](https://cucumber.io/docs/)
- [Gherkin Syntax Guide](https://cucumber.io/docs/gherkin/)
- [DBeaver Universal DB Tool](https://dbeaver.io/)
- [BDD Explained — ThoughtWorks](https://www.thoughtworks.com/insights/blog/brief-introduction-behavior-driven-development)

---
[**← Назад к оглавлению**](../README.md)
