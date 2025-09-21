### Подробное руководство по лекции 1: Введение в тестирование

Данное руководство охватывает первую лекцию курса по тестированию программного обеспечения (ПО), с акцентом на цели, принципы и ключевые аспекты тестирования, включая различия между QA (Quality Assurance), QC (Quality Control) и тестированием, а также навыки, необходимые тестировщику. Материал ориентирован на применение в автоматизации тестирования (AQA) и связан с другими темами, такими как клиент-серверная модель, локаторы, SQL и автоматизация с использованием инструментов, например, Selenium. Рассматриваются также семь принципов тестирования по ISTQB, положительные и отрицательные тесты, а также преимущества ручного тестирования.

---

## 1. Введение в тестирование

**Цель лекции**: Познакомить с основами тестирования ПО, его значением, целями и ролями в процессе разработки. Лекция закладывает фундамент для понимания роли тестировщика, особенно в контексте AQA, где тестирование часто связано с автоматизацией UI, API или баз данных.

**План лекции**:
1. Зачем нужно тестирование и его цели.
2. Различия между тестированием, QA и QC.
3. Основные понятия: ошибка, дефект, отказ, тест-кейсы.
4. Направления QA: функциональное, нефункциональное, автоматизированное тестирование и др.
5. Навыки, необходимые тестировщику.
6. Семь принципов тестирования по ISTQB.
7. Ответы на ключевые вопросы о тестировании.

**Связь с AQA**: Тестирование — основа автоматизации. Например, автоматизированные тесты с Selenium проверяют соответствие фактического и ожидаемого поведения UI, а SQL-запросы валидируют данные в базе.

---

## 2. Зачем необходимо тестирование

Тестирование необходимо для обеспечения качества ПО и минимизации рисков. Основные причины:

1. **Ошибки дорого исправлять, особенно после релиза**:
    - Исправление дефекта после выпуска продукта может стоить в десятки раз дороже, чем на этапе разработки.
    - Пример: Ошибка в форме логина, пропущенная на этапе тестирования, может привести к потере пользователей.
    - В AQA: Автоматизированные тесты (Selenium) выявляют такие ошибки до релиза.

2. **Недопонимания даже при чётких требованиях**:
    - Разработчики и заказчики могут по-разному интерпретировать спецификации.
    - Пример: Требование "поле ввода принимает email" может быть реализовано без проверки формата.
    - В AQA: Тест-кейсы с TestNG или JUnit проверяют соответствие требованиям.

3. **Пользователи ожидают стабильности и удобства**:
    - Пользователи требуют интуитивного UI и безошибочной работы.
    - Пример: Ошибка в корзине e-commerce приложения отпугивает клиентов.
    - В AQA: UI-тесты (Selenium) проверяют элементы интерфейса.

4. **Тестировщик — последний барьер перед пользователем**:
    - Тестировщик выявляет дефекты до того, как они дойдут до конечного пользователя.
    - Пример: Проверка формы логина на некорректные данные.
    - В AQA: Негативные тесты автоматизируются для проверки граничных случаев.

**Определение тестирования**:
- Тестирование ПО — это процесс проверки соответствия между **реальным (actual)** и **ожидаемым (expected)** поведением программы.
- Пример в AQA:

  ```java
  import org.openqa.selenium.By;
  import org.openqa.selenium.WebDriver;
  import org.openqa.selenium.chrome.ChromeDriver;
  import org.junit.jupiter.api.Test;
  import static org.junit.jupiter.api.Assertions.assertEquals;

  public class TestDefinition {
      @Test
      void testWelcomeMessage() {
          WebDriver driver = new ChromeDriver();
          driver.get("https://example.com");
          String expected = "Welcome, John";
          String actual = driver.findElement(By.id("message")).getText();
          assertEquals(expected, actual, "Сообщение не соответствует ожидаемому");
          driver.quit();
      }
  }
  ```

**Связь с другими темами**:
- **Клиент-серверная модель**: Тестирование проверяет взаимодействие клиента (UI) и сервера (API, БД).
- **Локаторы**: Тестирование UI требует стабильных локаторов (например, `By.ID`) для проверки элементов.

---

## 3. Направления QA

QA охватывает множество специализаций, каждая из которых требует уникальных навыков:

1. **Функциональное тестирование**:
   - **Описание**: Проверяет функциональность приложения и UI, например, корректность работы форм.
   - **Роли**: Software Tester, UI/UX Tester, Software Test Analyst.
   - **Пример**: Проверка формы логина с корректными данными.
   - **Применение в AQA**: Автоматизация UI-тестов с Selenium.

     ```java
     import org.openqa.selenium.By;
     import org.openqa.selenium.WebDriver;
     import org.openqa.selenium.chrome.ChromeDriver;
     import org.junit.jupiter.api.Test;

     public class FunctionalTest {
         @Test
         void testLoginForm() {
             WebDriver driver = new ChromeDriver();
             driver.get("https://example.com/login");
             driver.findElement(By.id("username")).sendKeys("John");
             driver.findElement(By.id("password")).sendKeys("pass123");
             driver.findElement(By.id("submit")).click();
             driver.quit();
         }
     }
     ```

2. **Автоматизированное тестирование**:
    - Автоматизация тестов для повышения эффективности.
    - Роли: Automation Test Engineer, SDET (Software Development Engineer in Test).
    - Пример: Написание тестов с TestNG и Selenium.
    - В AQA: Основной фокус — автоматизация UI, API, БД.

3. **Нефункциональное тестирование**:
    - **Load Testing**: Проверка производительности при нагрузке (Load Testing Engineer).
    - **Performance Testing**: Измерение скорости работы (Performance Tester).
    - **Security Testing**: Проверка уязвимостей (Security Tester).
    - Пример: Тестирование времени загрузки страницы.
    - В AQA: Использование JMeter для нагрузочного тестирования.

4. **Тестирование баз данных**:
    - Проверка целостности и качества данных.
    - Роли: Database QA Engineer, Data Quality Analyst.
    - Пример: SQL-запрос для проверки данных:
      ```sql
      SELECT * FROM users WHERE username = 'John';
      ```
    - В AQA: Интеграция SQL в тесты для валидации данных.

5. **Мобильное тестирование**:
    - Проверка приложений на iOS/Android.
    - Роли: Mobile QA Engineer.
    - Пример: Тестирование через Appium.
    - В AQA: Автоматизация мобильных UI-тестов.

6. **Тестирование API**:
   - **Описание**: Проверяет API-эндпоинты на корректность ответов, статус-кодов и данных.
   - **Роли**: API Tester, API Automation Engineer.
   - **Пример**: Тестирование GET-запроса к REST API.
   - **Применение в AQA**: Автоматизация с RestAssured (Java) или axios (JavaScript).
   - **Пример на Java (RestAssured)**:
     ```java
     import io.restassured.RestAssured;
     import io.restassured.response.Response;
     import org.junit.jupiter.api.Test;
     import static io.restassured.RestAssured.given;
     import static org.junit.jupiter.api.Assertions.assertEquals;

     public class ApiTest {
         @Test
         void testGetUsers() {
             RestAssured.baseURI = "https://api.example.com";
             Response response = given()
                     .get("/users");
             assertEquals(200, response.getStatusCode(), "Ожидался статус 200");
         }
     }
     ```

7. **Тестирование игр**:
    - Проверка игровых приложений.
    - Роли: GameDev Tester.
    - Пример: Проверка механик игры.
    - В AQA: Автоматизация тестирования UI игры.

8. **Тестирование AI и ML**:
    - Проверка моделей машинного обучения.
    - Роли: AI Tester, ML Testing Engineer.
    - Пример: Проверка точности предсказаний модели.
    - В AQA: Автоматизация валидации данных для ML.

**Связь с другими темами**:
- **SQL**: Тестирование БД требует SQL для проверки данных.
- **Клиент-серверная модель**: API-тестирование связано с RESTful взаимодействием.

---

## 4. Цели тестирования (по ISTQB)

По стандартам ISTQB (International Software Testing Qualifications Board), тестирование имеет следующие цели:

1. **Предотвращение появления дефектов**:
    - Выявление проблем на ранних этапах (например, код-ревью, тестирование требований).
    - Пример: Проверка спецификации формы логина.

2. **Верификация требований (Verification)**:
   - **Описание**: Проверяет, что продукт соответствует спецификациям (например, поле ввода принимает только валидный email).
   - **Применение в AQA**: Автоматизация проверок через UI или API.
   - **Пример на Java (Selenium)**:
     ```java
     import org.openqa.selenium.By;
     import org.openqa.selenium.WebDriver;
     import org.openqa.selenium.chrome.ChromeDriver;
     import org.junit.jupiter.api.Test;
     import static org.junit.jupiter.api.Assertions.assertTrue;

     public class VerificationTest {
         @Test
         void testInvalidEmail() {
             WebDriver driver = new ChromeDriver();
             driver.get("https://example.com/register");
             driver.findElement(By.id("email")).sendKeys("invalid");
             driver.findElement(By.id("submit")).click();
             assertTrue(driver.findElement(By.id("error")).isDisplayed(), "Ожидалась ошибка валидации");
             driver.quit();
         }
     }
     ```

3. **Валидация ожиданий заказчиков (Validation)**:
    - Проверка, что продукт соответствует ожиданиям пользователей.
    - Пример: Проверка удобства UI.
    - В AQA: UI-тесты для проверки UX.

4. **Создание уверенности в продукте**:
    - Убедить команду и заказчиков в качестве.
    - Пример: Положительные тесты подтверждают работоспособность.

5. **Снижение рисков**:
    - Выявление дефектов до релиза.
    - Пример: Негативные тесты для граничных случаев.

6. **Предоставление информации для принятия решений**:
    - Отчёты о тестировании для стейкхолдеров.
    - Пример: Отчёт о покрытии тестами в Jira.

7. **Соответствие формальным требованиям**:
    - Проверка на соответствие стандартам (GDPR, ISO).
    - Пример: Тестирование безопасности данных.

**Связь с другими темами**:
- **Тестирование**: Цели реализуются через тест-кейсы и assertions.
- **ООП**: Page Object Model в AQA структурирует тесты для верификации.

---

## 5. QA, QC и тестирование

- **QA (Quality Assurance)**:
    - Процесс обеспечения качества на всех этапах разработки.
    - Включает: код-ревью, улучшение процессов, обучение.
    - Пример: Внедрение CI/CD для автоматического тестирования.

- **QC (Quality Control)**:
    - Проверка качества готового продукта.
    - Включает: тестирование, выявление дефектов.
    - Пример: Выполнение тест-кейсов перед релизом.

- **Testing (Тестирование)**:
    - Конкретный процесс проверки ПО на соответствие требованиям.
    - Пример: Ручное или автоматическое тестирование UI.

**Различия**:
- QA — проактивный процесс (предотвращение ошибок).
- QC и тестирование — реактивные процессы (поиск и исправление дефектов).

**Связь с AQA**: Автоматизация (Selenium, TestNG) — часть QC, но способствует QA через раннее выявление ошибок.

---

## 6. Основные понятия

1. **Ошибка (Error)**:
    - Ошибка человека (например, опечатка в коде).
    - Пример: Программист написал `if (x = 5)` вместо `if (x == 5)`.
    - В AQA: Ошибки выявляются через тестирование кода.

2. **Дефект (Defect/Bug)**:
   - **Описание**: Несоответствие продукта спецификациям или ожиданиям.
   - **Пример**: Кнопка логина неактивна при пустом пароле.
   - **Применение в AQA**: Логирование дефектов в баг-трекере (например, Jira).
   - **Пример на Java (Selenium)**:
     ```java
     import org.openqa.selenium.By;
     import org.openqa.selenium.WebDriver;
     import org.openqa.selenium.chrome.ChromeDriver;
     import org.junit.jupiter.api.Test;

     public class DefectTest {
         @Test
         void testLoginButton() {
             WebDriver driver = new ChromeDriver();
             driver.get("https://example.com/login");
             if (!driver.findElement(By.id("submit")).isEnabled()) {
                 System.out.println("Bug: Кнопка неактивна");
             }
             driver.quit();
         }
     }
     ```

3. **Отказ (Failure)**:
    - Дефект, обнаруженный пользователем после релиза.
    - Пример: Пользователь видит ошибку 500 вместо страницы.
    - В AQA: Цель — предотвратить отказы через тестирование.

4. **Положительный тест (Positive Test)**:
   - **Описание**: Проверка на корректных данных (например, валидный email и пароль).
   - **Пример**: Успешный логин с корректными данными.
   - **Применение в AQA**: Автоматизация позитивных сценариев.
   - **Пример на Java (Selenium)**:
     ```java
     import org.openqa.selenium.By;
     import org.openqa.selenium.WebDriver;
     import org.openqa.selenium.chrome.ChromeDriver;
     import org.junit.jupiter.api.Test;
     import static org.junit.jupiter.api.Assertions.assertEquals;

     public class PositiveTest {
         @Test
         void testPositiveLogin() {
             WebDriver driver = new ChromeDriver();
             driver.get("https://example.com/login");
             driver.findElement(By.id("email")).sendKeys("test@example.com");
             driver.findElement(By.id("password")).sendKeys("pass123");
             driver.findElement(By.id("submit")).click();
             assertEquals("Welcome", driver.findElement(By.id("message")).getText(), "Ожидалось приветственное сообщение");
             driver.quit();
         }
     }
     ```

5. **Отрицательный тест (Negative Test)**:
   - **Описание**: Проверка на некорректных данных или действиях (например, неверный пароль).
   - **Пример**: Появление ошибки при неверном пароле.
   - **Применение в AQA**: Автоматизация негативных сценариев.
   - **Пример на Java (Selenium)**:
     ```java
     import org.openqa.selenium.By;
     import org.openqa.selenium.WebDriver;
     import org.openqa.selenium.chrome.ChromeDriver;
     import org.junit.jupiter.api.Test;
     import static org.junit.jupiter.api.Assertions.assertTrue;

     public class NegativeTest {
         @Test
         void testNegativeLogin() {
             WebDriver driver = new ChromeDriver();
             driver.get("https://example.com/login");
             driver.findElement(By.id("password")).sendKeys("wrong");
             driver.findElement(By.id("submit")).click();
             assertTrue(driver.findElement(By.id("error")).isDisplayed(), "Ожидалась ошибка");
             driver.quit();
         }
     }
     ```

6. **Тест-кейс (Test Case)**:
   - **Описание**: Документированный тест с входными данными, условиями и ожидаемым результатом.
   - **Пример**: "Ввести email: test@example.com, пароль: pass123, ожидать: Welcome".
   - **Применение в AQA**: Автоматизация тест-кейсов.
   - **Пример на Java (JUnit 5)**:
     ```java
     import org.openqa.selenium.By;
     import org.openqa.selenium.WebDriver;
     import org.openqa.selenium.chrome.ChromeDriver;
     import org.junit.jupiter.api.Test;
     import static org.junit.jupiter.api.Assertions.assertEquals;

     public class TestCaseExample {
         @Test
         void testLoginSuccess() {
             WebDriver driver = new ChromeDriver();
             driver.get("https://example.com/login");
             driver.findElement(By.id("email")).sendKeys("test@example.com");
             driver.findElement(By.id("password")).sendKeys("pass123");
             driver.findElement(By.id("submit")).click();
             assertEquals("Welcome", driver.findElement(By.id("message")).getText(), "Ожидалось приветственное сообщение");
             driver.quit();
         }
     }
     ```

7. **Критерий прохождения (Pass Criteria)**:
    - Правило, определяющее успешность теста.
    - Пример: "Сообщение 'Welcome' отображается".
    - В AQA: Реализуется через assertions.

**Связь с другими темами**:
- **SQL**: Тест-кейсы для БД используют SELECT для проверки данных.
- **Локаторы**: Тест-кейсы для UI зависят от стабильных локаторов.

---

## 7. Что должен знать manual tester

### 7.1. Теория тестирования
- Основы тестирования: цели, принципы (ISTQB), виды тестов.
- Типы тестирования: функциональное, нефункциональное, регрессионное.
- Жизненный цикл тестирования: планирование, написание тест-кейсов, выполнение, отчёт.

### 7.2. Компьютерная грамотность
- **Hardware**: Понимание устройств для тестирования (ПК, мобильные).
- **Операционные системы**: Работа в Windows, macOS, Linux.
- **Клиент-серверная модель**: Понимание, как UI взаимодействует с сервером.
- **Логи и скриншоты**: Сбор логов (например, в DevTools) и скриншотов для баг-репортов.
- **Internet, HTTP/HTTPS**: Знание протоколов для API-тестирования.
- **HTML**: Понимание структуры DOM для поиска локаторов.
- **Командная строка**: Использование для запуска тестов.
- **Web browsers & DevTools**: Проверка локаторов в Chrome DevTools (Ctrl+F для XPath).
- **Excel**: Создание тест-кейсов и анализ данных.

### 7.3. Инструменты тестировщика
- **Баг-трекеры и системы управления тест-кейсами**: Jira, Trello, Zephyr.
- **Инструменты коммуникации**: Slack, Microsoft Teams, Confluence.
- **API-тестирование**: Postman, SoapUI, Swagger.
- **Базы данных**: SQL (MySQL Workbench) для проверки данных.
- **Автоматизация**: Selenium, Appium.

### 7.4. Soft skills
- **Общение**: Взаимодействие с разработчиками и менеджерами.
- **Самообучение**: Изучение новых инструментов (например, Selenium).
- **Командность**: Работа в Agile/Scrum-командах.
- **Проактивность**: Инициатива в поиске дефектов.

### 7.5. Навыки трудоустройства
- Резюме и портфолио: Указывайте проекты с Selenium, SQL.
- Telegram-каналы: Например, **CatchTheOffer** для поиска вакансий.
- Практика: Выполняйте тестовые задания (например, автоматизация логина).

**Связь с AQA**:
- Автоматизация требует знаний ручного тестирования для создания эффективных тест-кейсов.
- Пример: Тестировщик пишет тест-кейс в Jira, затем автоматизирует его с Selenium.

---

## 8. Семь принципов тестирования (по ISTQB)

1. **Тестирование показывает наличие дефектов, но не их отсутствие**:
    - Тестирование снижает риски, но не гарантирует 100% качества.
    - Пример: Даже если тест пройден, могут быть скрытые баги.

2. **Исчерпывающее тестирование невозможно**:
    - Нельзя протестировать все комбинации данных и сценариев.
    - Пример: В AQA приоритет на критические сценарии (логин, оплата).

3. **Раннее тестирование экономит время и деньги**:
    - Выявление дефектов на этапе требований дешевле.
    - Пример: Проверка спецификаций перед разработкой.

4. **Скопление дефектов**:
    - Большинство дефектов сосредоточено в определённых модулях.
    - Пример: Тестирование формы логина выявляет 80% багов UI.

5. **Парадокс пестицида**:
    - Повторение одних и тех же тестов снижает их эффективность.
    - Пример: Добавляйте негативные тесты для новых сценариев.

6. **Тестирование зависит от контекста**:
    - Тесты для e-commerce отличаются от тестов для медицинских систем.
    - Пример: В AQA для мобильных приложений используется Appium.

7. **Ошибочное отсутствие ошибок**:
    - Отсутствие найденных дефектов не означает качественный продукт.
    - Пример: Негативные тесты обязательны для проверки граничных случаев.

**Связь с AQA**:
   - **Принципы в AQA**: Верификация (соответствие требованиям) и валидация (правильность работы) автоматизируются.
   - **Пример на Java (Selenium + RestAssured)**:
     ```java
     import io.restassured.RestAssured;
     import org.openqa.selenium.By;
     import org.openqa.selenium.WebDriver;
     import org.openqa.selenium.chrome.ChromeDriver;
     import org.junit.jupiter.api.Test;
     import static io.restassured.RestAssured.given;
     import static org.junit.jupiter.api.Assertions.assertTrue;
     import static org.junit.jupiter.api.Assertions.assertEquals;
   
     public class AqaIntegrationTest {
         @Test
         void testNegativeLoginAndApi() {
             // Негативный тест UI
             WebDriver driver = new ChromeDriver();
             driver.get("https://example.com/login");
             driver.findElement(By.id("password")).sendKeys("wrong");
             driver.findElement(By.id("submit")).click();
             assertTrue(driver.findElement(By.id("error")).isDisplayed(), "Ожидалась ошибка");
             driver.quit();
   
             // Верификация через API
             Response response = given()
                     .get("https://api.example.com/users/1");
             assertEquals(200, response.getStatusCode(), "Ожидался статус 200");
         }
     }
     ```

---

## 9. Ответы на ключевые вопросы

1. **Что такое тестирование программного обеспечения?**
    - Проверка соответствия между реальным и ожидаемым поведением ПО.
    - Пример: Проверка, что кнопка "Войти" работает корректно.

2. **Что означает верификация и валидация?**
   - **Изучайте теорию тестирования**: Ознакомьтесь с ISTQB Foundation Level Syllabus.
   - **Практикуйтесь**:
      - Пишите тест-кейсы для простых приложений (например, формы логина).
   - **Пример на Java (JUnit 5)**:
     ```java
     import org.openqa.selenium.By;
     import org.openqa.selenium.WebDriver;
     import org.openqa.selenium.chrome.ChromeDriver;
     import org.junit.jupiter.api.Test;
     import static org.junit.jupiter.api.Assertions.assertEquals;

     public class PracticeTest {
         @Test
         void testLoginPositive() {
             WebDriver driver = new ChromeDriver();
             driver.get("https://example.com/login");
             driver.findElement(By.id("username")).sendKeys("testuser");
             driver.findElement(By.id("password")).sendKeys("pass123");
             driver.findElement(By.id("submit")).click();
             assertEquals("Welcome", driver.findElement(By.id("message")).getText(), "Ожидалось приветственное сообщение");
             driver.quit();
         }
     }
     ```

3. **Что такое контроль качества (QC) и чем он отличается от QA?**
    - **QC**: Проверка готового продукта (тестирование, выявление дефектов).
    - **QA**: Процесс предотвращения дефектов на всех этапах.
    - Пример: QC — выполнение тест-кейса, QA — внедрение автоматизации.

4. **Преимущества ручного тестирования**:
    - **Гибкость**: Подходит для тестирования UX и сложных сценариев.
    - **Быстрое начало**: Не требует написания кода.
    - **Обнаружение визуальных багов**: Например, смещение кнопки в UI.
    - **Исследовательское тестирование**: Поиск багов без заранее заданных тест-кейсов.
    - Пример: Ручная проверка формы логина на разных устройствах.
    - В AQA: Ручное тестирование помогает создавать автоматизированные тест-кейсы.

---

## 10. Рекомендации

- **Изучайте теорию тестирования**:
    - Ознакомьтесь с ISTQB Foundation Level Syllabus.
- **Практикуйтесь**:
  - Пишите тест-кейсы для простых приложений (например, формы логина).
  - **Пример на Java (JUnit 5, Selenium)**:
    ```java
    import org.openqa.selenium.By;
    import org.openqa.selenium.WebDriver;
    import org.openqa.selenium.chrome.ChromeDriver;
    import org.junit.jupiter.api.Test;
    import static org.junit.jupiter.api.Assertions.assertEquals;

    public class PracticeTest {
        @Test
        void testLoginPositive() {
            WebDriver driver = new ChromeDriver();
            driver.get("https://example.com/login");
            driver.findElement(By.id("username")).sendKeys("testuser");
            driver.findElement(By.id("password")).sendKeys("pass123");
            driver.findElement(By.id("submit")).click();
            assertEquals("Welcome", driver.findElement(By.id("message")).getText(), "Ожидалось приветственное сообщение");
            driver.quit();
        }
    }
    ```
- **Используйте инструменты**:
    - Jira для баг-трекинга.
    - Postman для API-тестов.
    - MySQL Workbench для проверки БД.
- **Развивайте soft skills**:
    - Участвуйте в Slack-каналах (#qa_sql) и Telegram (#CatchTheOffer).
- **Автоматизация**:
    - Начните с Selenium и TestNG для UI-тестов.
- **CI/CD**:
    - Интегрируйте тесты в GitHub Actions:
      ```yaml
      name: Run Tests
      on: [push]
      jobs:
        test:
          runs-on: ubuntu-latest
          steps:
            - uses: actions/checkout@v3
            - name: Set up JDK
              uses: actions/setup-java@v3
              with:
                java-version: '17'
            - name: Install dependencies
              run: mvn install
            - name: Run tests
              run: mvn
      ```

---

## 11. Источники
- [ISTQB Foundation Level Syllabus](https://www.istqb.org/certifications/certified-tester-foundation-level)
- [Selenium Documentation](https://www.selenium.dev/documentation/)
- [Baeldung: JUnit 5 Tutorial](https://www.baeldung.com/junit-5)
- [Postman Documentation](https://learning.postman.com/docs/)
- [W3Schools SQL Tutorial](https://www.w3schools.com/sql/)

---

[**&#x2190; Назад к оглавлению**](README.md)