### Подробное руководство по часто используемым функциям JUnit 5 (JUnit Jupiter)

JUnit 5 — это современная платформа для модульного тестирования в Java, состоящая из трёх основных модулей: **JUnit Platform**, **JUnit Jupiter** и **JUnit Vintage**. JUnit Jupiter — это API для написания тестов и расширения функциональности, который включает аннотации, утверждения (assertions), методы жизненного цикла и другие инструменты для создания гибких и читаемых тестов. В этом руководстве подробно описаны наиболее часто используемые функции и аннотации JUnit 5 Jupiter, их применение в контексте Java, примеры кода, интеграция с IntelliJ IDEA, а также связь с темами из ваших предыдущих запросов (например, эквивалентность, примитивные типы, ООП).

---

#### 1. Основы JUnit 5 Jupiter

**JUnit Jupiter** предоставляет API для написания тестов с использованием аннотаций, таких как `@Test`, `@BeforeEach`, `@AfterEach`, и мощных утверждений через класс `org.junit.jupiter.api.Assertions`. Основные преимущества:
- **Модульность**: Поддержка новых возможностей, таких как параметризованные тесты и динамические тесты.
- **Читаемость**: Улучшенные сообщения об ошибках и поддержка лямбда-выражений.
- **Расширяемость**: Механизм расширений (Extensions) для кастомизации тестов.
- **Интеграция**: Работает с Maven, Gradle, IntelliJ IDEA и CI/CD-системами.

##### Зависимости Maven для JUnit 5 Jupiter
```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.11.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.11.2</version>
    <scope>test</scope>
</dependency>
```
- **Примечание**: На октябрь 2025 года последняя версия — 5.11.2.

---

#### 2. Часто используемые аннотации JUnit 5 Jupiter

##### 2.1. `@Test`
- **Описание**: Помечает метод как тестовый. Не требует указания возвращаемого типа или исключений (в отличие от JUnit 4).
- **Когда использовать**: Для определения тестового сценария.
- **Пример**:
  ```java
  import org.junit.jupiter.api.Test;
  import static org.junit.jupiter.api.Assertions.assertEquals;

  class CalculatorTest {
      @Test
      void testAddition() {
          Calculator calc = new Calculator();
          assertEquals(5, calc.add(2, 3), "2 + 3 должно равняться 5");
      }
  }
  ```
- **IntelliJ IDEA**: Щёлкните правой кнопкой на метод → **Run 'testAddition()'**.

##### 2.2. `@BeforeEach`
- **Описание**: Выполняет метод перед каждым тестом в классе. Используется для инициализации общих ресурсов.
- **Когда использовать**: Для создания объектов, очистки данных или настройки окружения.
- **Пример**:
  ```java
  import org.junit.jupiter.api.BeforeEach;
  import org.junit.jupiter.api.Test;

  class CalculatorTest {
      private Calculator calc;

      @BeforeEach
      void setUp() {
          calc = new Calculator(); // Инициализация перед каждым тестом
      }

      @Test
      void testAddition() {
          assertEquals(5, calc.add(2, 3));
      }
  }
  ```

##### 2.3. `@AfterEach`
- **Описание**: Выполняет метод после каждого теста. Используется для очистки ресурсов.
- **Когда использовать**: Для закрытия файлов, освобождения памяти или сброса состояния.
- **Пример**:
  ```java
  @AfterEach
  void tearDown() {
      calc = null; // Очистка после каждого теста
  }
  ```

##### 2.4. `@BeforeAll`
- **Описание**: Выполняет статический метод один раз перед всеми тестами в классе. Требует `static`.
- **Когда использовать**: Для настройки глобальных ресурсов (например, подключение к БД).
- **Пример**:
  ```java
  import org.junit.jupiter.api.BeforeAll;

  class DatabaseTest {
      @BeforeAll
      static void setUpDatabase() {
          System.out.println("Инициализация базы данных");
      }
  }
  ```

##### 2.5. `@AfterAll`
- **Описание**: Выполняет статический метод один раз после всех тестов. Требует `static`.
- **Когда использовать**: Для закрытия глобальных ресурсов.
- **Пример**:
  ```java
  @AfterAll
  static void tearDownDatabase() {
      System.out.println("Закрытие базы данных");
  }
  ```

##### 2.6. `@Disabled`
- **Описание**: Отключает тест или класс тестов. Указывает причину отключения.
- **Когда использовать**: Для временного исключения тестов, которые ещё не готовы.
- **Пример**:
  ```java
  @Disabled("Тест ещё не реализован")
  @Test
  void testNotReady() {
      // Тест не будет выполнен
  }
  ```
- **IntelliJ IDEA**: Отключенные тесты отображаются серым цветом в Test Runner.

##### 2.7. `@DisplayName`
- **Описание**: Задаёт пользовательское имя для теста или класса, отображаемое в отчётах.
- **Когда использовать**: Для улучшения читаемости результатов тестов.
- **Пример**:
  ```java
  @Test
  @DisplayName("Проверка сложения двух чисел")
  void testAddition() {
      assertEquals(5, new Calculator().add(2, 3));
  }
  ```

##### 2.8. `@ParameterizedTest`
- **Описание**: Позволяет запускать один тест с разными входными данными.
- **Источники данных**:
    - `@ValueSource`: Простые значения (int, String и т.д.).
    - `@CsvSource`: Данные в формате CSV.
    - `@MethodSource`: Пользовательский метод-источник.
- **Когда использовать**: Для тестирования метода с разными входными параметрами.
- **Пример**:
  ```java
  import org.junit.jupiter.params.ParameterizedTest;
  import org.junit.jupiter.params.provider.ValueSource;

  class StringTest {
      @ParameterizedTest
      @ValueSource(strings = {"hello", "world", ""})
      void testNotEmpty(String input) {
          assertFalse(input.isEmpty(), "Строка не должна быть пустой");
      }
  }
  ```

##### 2.9. `@RepeatedTest`
- **Описание**: Повторяет тест указанное количество раз.
- **Когда использовать**: Для проверки стабильности кода.
- **Пример**:
  ```java
  import org.junit.jupiter.api.RepeatedTest;

  class RandomTest {
      @RepeatedTest(5)
      void testRandomNumber() {
          int random = (int) (Math.random() * 10);
          assertTrue(random >= 0 && random < 10);
      }
  }
  ```

##### 2.10. `@Nested`
- **Описание**: Позволяет группировать тесты в вложенные классы для логической организации.
- **Когда использовать**: Для разделения тестов по функциональности.
- **Пример**:
  ```java
  import org.junit.jupiter.api.Nested;

  class CalculatorTest {
      @Nested
      class AdditionTests {
          @Test
          void testPositiveNumbers() {
              assertEquals(5, new Calculator().add(2, 3));
          }
      }
  }
  ```

##### 2.11. `@Tag`
- **Описание**: Присваивает тег тесту или классу для фильтрации при запуске.
- **Когда использовать**: Для запуска определённых групп тестов (например, `smoke`, `regression`).
- **Пример**:
  ```java
  @Test
  @Tag("smoke")
  void testCriticalFunction() {
      assertTrue(true);
  }
  ```
- **Запуск тегов в Maven**:
  ```bash
  mvn test -Dgroups="smoke"
  ```

##### 2.12. `@TestFactory`
- **Описание**: Создаёт динамические тесты во время выполнения.
- **Когда использовать**: Для генерации тестов на основе данных.
- **Пример**:
  ```java
  import org.junit.jupiter.api.DynamicTest;
  import org.junit.jupiter.api.TestFactory;

  import java.util.stream.Stream;

  class DynamicTestExample {
      @TestFactory
      Stream<DynamicTest> testNumbers() {
          return Stream.of(1, 2, 3)
                       .map(n -> DynamicTest.dynamicTest("Test " + n, () -> assertTrue(n > 0)));
      }
  }
  ```

---

#### 3. Часто используемые утверждения (Assertions)

Класс `org.junit.jupiter.api.Assertions` предоставляет методы для проверки ожидаемых результатов.

##### 3.1. `assertEquals(expected, actual, [message])`
- **Описание**: Проверяет, что ожидаемое значение равно фактическому.
- **Пример**:
  ```java
  assertEquals(4, new Calculator().add(2, 2), "2 + 2 должно быть 4");
  ```

##### 3.2. `assertNotEquals(unexpected, actual, [message])`
- **Описание**: Проверяет, что значения не равны.
- **Пример**:
  ```java
  assertNotEquals(5, new Calculator().add(2, 2));
  ```

##### 3.3. `assertTrue(condition, [message])`
- **Описание**: Проверяет, что условие истинно.
- **Пример**:
  ```java
  assertTrue(5 > 0, "5 должно быть больше 0");
  ```

##### 3.4. `assertFalse(condition, [message])`
- **Описание**: Проверяет, что условие ложно.
- **Пример**:
  ```java
  assertFalse(5 < 0);
  ```

##### 3.5. `assertNull(object, [message])` / `assertNotNull(object, [message])`
- **Описание**: Проверяет, что объект равен/не равен `null`.
- **Пример**:
  ```java
  assertNull(null);
  assertNotNull(new Object());
  ```

##### 3.6. `assertThrows(expectedType, executable, [message])`
- **Описание**: Проверяет, что код выбрасывает ожидаемое исключение.
- **Пример**:
  ```java
  assertThrows(ArithmeticException.class, () -> {
      int result = 10 / 0;
  }, "Деление на ноль должно выбросить исключение");
  ```

##### 3.7. `assertTimeout(duration, executable)`
- **Описание**: Проверяет, что код выполняется в пределах заданного времени.
- **Пример**:
  ```java
  import java.time.Duration;

  assertTimeout(Duration.ofSeconds(1), () -> {
      Thread.sleep(500); // Успешно, так как < 1 секунды
  });
  ```

##### 3.8. `assertAll(executables)`
- **Описание**: Выполняет несколько проверок, собирая все ошибки перед завершением.
- **Когда использовать**: Для проверки нескольких условий в одном тесте.
- **Пример**:
  ```java
  assertAll("Проверка нескольких условий",
      () -> assertEquals(4, 2 + 2),
      () -> assertTrue(5 > 0),
      () -> assertNotNull(new Object())
  );
  ```

---

#### 4. Полный пример тестового класса

```java
import org.junit.jupiter.api.*;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.CsvSource;

import static org.junit.jupiter.api.Assertions.*;

class CalculatorTest {
    private Calculator calc;

    @BeforeEach
    void setUp() {
        calc = new Calculator();
    }

    @Test
    @DisplayName("Проверка сложения")
    void testAddition() {
        assertEquals(5, calc.add(2, 3), "2 + 3 должно быть 5");
    }

    @ParameterizedTest
    @CsvSource({"1, 2, 3", "0, 0, 0", "-1, 1, 0"})
    void testAdditionParameterized(int a, int b, int expected) {
        assertEquals(expected, calc.add(a, b));
    }

    @Test
    @Tag("smoke")
    void testDivisionByZero() {
        assertThrows(ArithmeticException.class, () -> calc.divide(10, 0));
    }

    @Nested
    class SubtractionTests {
        @Test
        void testSubtraction() {
            assertEquals(2, calc.subtract(5, 3));
        }
    }

    @AfterEach
    void tearDown() {
        calc = null;
    }
}

class Calculator {
    int add(int a, int b) {
        return a + b;
    }

    int subtract(int a, int b) {
        return a - b;
    }

    int divide(int a, int b) {
        return a / b;
    }
}
```

---

#### 5. Интеграция с IntelliJ IDEA

- **Настройка проекта**:
    - Добавьте зависимости JUnit 5 в `pom.xml` или `build.gradle`.
    - IntelliJ IDEA автоматически распознаёт JUnit 5.
- **Создание теста**:
    - Щёлкните ПКМ на классе → **Generate** (Alt+Insert) → **Test** → Выберите JUnit 5.
- **Запуск тестов**:
    - Щёлкните ПКМ на методе/классе → **Run 'testName()'** или **Run 'ClassName'**.
    - Используйте **Run → Run with Coverage** для анализа покрытия кода.
- **Отладка**:
    - Установите точку останова (Ctrl+F8).
    - Запустите тест в режиме отладки (**Run → Debug**, Shift+F9).
    - Используйте **Evaluate Expression** (Alt+F8) для проверки значений.
- **Анализ результатов**:
    - Вкладка **Run** показывает результаты тестов (зелёный — успех, красный — провал).
    - Щёлкните на ошибке для перехода к строке кода.
- **Рефакторинг**:
    - Используйте **Refactor → Rename** (Shift+F6) для переименования тестов.
    - Автоматически генерируйте `@Test` методы через **Code → Generate**.

---

#### 6. Связь с предыдущими темами

- **Эквивалентность**:
    - JUnit 5 использует `assertEquals` для проверки эквивалентности:
      ```java
      assertEquals("Hello", "Hello", "Строки должны быть равны");
      ```
    - Связь с XPath/Selenium: Проверка текста элемента:
      ```java
      String text = driver.findElement(By.xpath("//div")).getText();
      assertEquals("Expected", text);
      ```

- **Примитивные типы**:
    - Тесты JUnit 5 часто работают с примитивными типами (`int`, `boolean`):
      ```java
      @Test
      void testBoolean() {
          assertTrue(true, "Значение должно быть истинным");
      }
      ```

- **Операторы**:
    - Логические операторы используются в условиях:
      ```java
      assertTrue(5 > 0 && 3 < 10, "Условие должно быть истинным");
      ```

- **ООП**:
    - Тестовые классы используют принципы ООП:
        - **Инкапсуляция**: Поля и методы тестов инкапсулированы в классах.
        - **Наследование**: Можно создавать базовые тестовые классы.
        - **Полиморфизм**: Используется в расширениях и параметризованных тестах.
    - Пример:
      ```java
      abstract class BaseTest {
          @BeforeEach
          void setUp() {
              System.out.println("Базовая настройка");
          }
      }
  
      class DerivedTest extends BaseTest {
          @Test
          void test() {
              assertTrue(true);
          }
      }
      ```

- **Коллекции**:
    - Проверка коллекций с помощью `assertIterableEquals`:
      ```java
      import java.util.List;
  
      assertIterableEquals(List.of(1, 2), List.of(1, 2));
      ```

- **Selenium и XPath**:
    - JUnit 5 часто используется для тестирования веб-приложений:
      ```java
      import org.openqa.selenium.By;
      import org.openqa.selenium.WebDriver;
      import org.openqa.selenium.chrome.ChromeDriver;
  
      class LoginTest {
          WebDriver driver;
  
          @BeforeEach
          void setUp() {
              System.setProperty("webdriver.chrome.driver", "path/to/chromedriver");
              driver = new ChromeDriver();
          }
  
          @Test
          void testLogin() {
              driver.get("https://example.com/login");
              driver.findElement(By.xpath("//input[@id='username']")).sendKeys("testuser");
              driver.findElement(By.xpath("//button[text()='Submit']")).click();
              assertEquals("Welcome", driver.findElement(By.xpath("//div")).getText());
          }
  
          @AfterEach
          void tearDown() {
              driver.quit();
          }
      }
      ```

---

#### 7. Рекомендации

- **Используйте `@DisplayName`**: Улучшает читаемость отчётов.
- **Применяйте параметризованные тесты**: Сокращают дублирование кода.
- **Добавляйте сообщения в assertions**: Упрощают диагностику ошибок.
- **Организуйте тесты с `@Nested`**: Логически группируйте тесты.
- **Избегайте сложной логики в тестах**:
  ```java
  // Плохо
  @Test
  void complexTest() {
      if (condition) { /* ... */ } else { /* ... */ }
  }

  // Хорошо
  @Test
  void testConditionTrue() { /* ... */ }
  @Test
  void testConditionFalse() { /* ... */ }
  ```
- **Используйте ожидания в Selenium**:
  ```java
  import org.openqa.selenium.support.ui.WebDriverWait;
  import org.openqa.selenium.support.ui.ExpectedConditions;

  WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
  wait.until(ExpectedConditions.visibilityOfElementLocated(By.xpath("//div")));
  ```
- **Интеграция с CI/CD**: Настройте запуск тестов в Jenkins с помощью Maven/Gradle.
- **Покрытие кода**:
    - Используйте JaCoCo в IntelliJ IDEA (**Run → Run with Coverage**).
    - Стремитесь к покрытию >80% для критического кода.

---

#### 8. Преимущества и недостатки JUnit 5 Jupiter

##### Преимущества:
- Современный API с поддержкой Java 8+ (лямбда-выражения, Stream API).
- Гибкость благодаря параметризованным и динамическим тестам.
- Подробные сообщения об ошибках.
- Расширяемость через механизм расширений.
- Интеграция с популярными инструментами (Selenium, Mockito).

##### Недостатки:
- Требует перехода с JUnit 4 (изменения в аннотациях, импортах).
- Более сложная настройка для начинающих по сравнению с JUnit 4.
- Ограниченная поддержка устаревших инструментов, использующих JUnit 4.

---

#### 9. Источники
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [Baeldung: JUnit 5 Tutorial](https://www.baeldung.com/junit-5)
- [DZone: Getting Started with JUnit 5](https://dzone.com/articles/getting-started-with-junit-5)
- [Vogella: JUnit 5 Tutorial](https://www.vogella.com/tutorials/JUnit/article.html)

---
