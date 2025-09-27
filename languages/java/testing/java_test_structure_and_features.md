# Практика написания тестов и современные возможности Java

В этой статье рассмотрены основные подходы к написанию тестов на Java с использованием JUnit 5 (а также отличия от JUnit 4). Понимание структуры тестов, аннотаций и жизненного цикла позволяет создавать более надёжную систему тестирования и облегчает сопровождение кода. Кроме того, будут рассмотрены современные возможности языка Java, которые упрощают тесты и делают их более выразительными.

---

## 🧪 Аннотация `@Test` и описание тестов

Аннотация `@Test` указывает, что метод является тестовым. В JUnit 5 тесты могут быть методами с любой областью видимости (кроме `private`), без необходимости быть `public`, как это требовалось в JUnit 4.

Простой пример:

```java
@Test
void shouldAddTwoNumbers() {
    int result = calculator.add(2, 3);
    assertEquals(5, result);
}
```

Чтобы сделать тесты более понятными при запуске, используют аннотацию `@DisplayName`:

```java
@DisplayName("Сложение двух положительных чисел")
@Test
void addPositiveNumbers() {
    assertEquals(4, calculator.add(2, 2));
}
```

💡 Рекомендация: всегда пишите тестовые методы так, чтобы их названия и описания чётко отражали **поведение системы**, а не внутренние реализации.

---

## 🧾 Структура теста: AAA и given-when-then

Хорошая структура делает тест читаемым и простым для понимания. Обычно используют два популярных подхода:

### AAA (Arrange-Act-Assert)

1. **Arrange** — подготовка данных и объектов
2. **Act** — вызов тестируемого метода
3. **Assert** — проверка результата

```java
@Test
void shouldReturnSum_whenTwoNumbersGiven() {
    // Arrange
    Calculator calc = new Calculator();

    // Act
    int result = calc.add(2, 3);

    // Assert
    assertEquals(5, result);
}
```

### Given-When-Then (BDD стиль)

Чаще используется в поведении-ориентированном тестировании (BDD):

* **Given** — задано условие
* **When** — выполняется действие
* **Then** — ожидается результат

```java
@Test
void givenTwoNumbers_whenAdded_thenReturnsSum() {
    Calculator calc = new Calculator();
    int result = calc.add(2, 3);
    assertEquals(5, result);
}
```

💡 Совет: выбирайте стиль единообразно внутри проекта, чтобы тесты выглядели консистентно.

---

## 🔄 Жизненный цикл тестов: `@BeforeEach`, `@AfterEach`, `@BeforeAll`, `@AfterAll`

JUnit предоставляет аннотации для настройки окружения тестов:

### `@BeforeEach`

Выполняется **перед каждым тестом**. Подходит для инициализации:

```java
@BeforeEach
void init() {
    calculator = new Calculator();
}
```

### `@AfterEach`

Выполняется **после каждого теста**. Используется для очистки:

```java
@AfterEach
void tearDown() {
    calculator = null;
}
```

### `@BeforeAll` и `@AfterAll`

Выполняются **один раз** до и после всех тестов. Методы должны быть `static` (или помечены `@TestInstance(Lifecycle.PER_CLASS)`):

```java
@BeforeAll
static void beforeAllTests() {
    System.out.println("Подготовка перед запуском тестов");
}

@AfterAll
static void afterAllTests() {
    System.out.println("Все тесты завершены");
}
```

---

## 🔢 Управление порядком выполнения тестов

По умолчанию JUnit выполняет тесты в произвольном порядке. Чтобы явно задать порядок, используют `@TestMethodOrder` и аннотацию `@Order`.

```java
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class OrderedTests {

    @Order(1)
    @Test
    void initData() { ... }

    @Order(2)
    @Test
    void processData() { ... }
}
```

⚠️ Но чаще всего рекомендуется **не полагаться на порядок тестов**, а делать их независимыми друг от друга.

---

## 🧪 Параметризация тестов

JUnit 5 позволяет запускать один тест с разными наборами входных данных.

### `@CsvSource`

```java
@ParameterizedTest
@CsvSource({
    "2, 3, 5",
    "10, 5, 15",
    "-2, 8, 6"
})
void testAdd(int a, int b, int expected) {
    assertEquals(expected, calculator.add(a, b));
}
```

### `@ValueSource`

Для одного аргумента:

```java
@ParameterizedTest
@ValueSource(strings = {"abc", "test", "JUnit"})
void testNotEmpty(String word) {
    assertFalse(word.isEmpty());
}
```

---

## 🛑 Отключение тестов

Чтобы временно выключить тест, используют `@Disabled`:

```java
@Disabled("Функционал ещё не реализован")
@Test
void testFeatureInProgress() {
    // не будет выполнен
}
```

---

## 📊 Утверждения (Assertions)

JUnit предоставляет богатый набор методов для проверки результатов:

```java
assertEquals(10, calculator.add(5, 5));
assertTrue(result > 0);
assertFalse(list.isEmpty());
assertNotNull(object);
assertThrows(IllegalArgumentException.class, () -> calculator.divide(1, 0));
```

💡 Для более гибкой проверки можно использовать **Hamcrest** или **AssertJ**.

---

## 🚀 Современные возможности Java в тестах

* **var** для упрощённого объявления переменных:

  ```java
  var result = calculator.add(2, 3);
  ```
* **Лямбда-выражения** и `assertThrows`:

  ```java
  assertThrows(ArithmeticException.class, () -> calc.divide(1, 0));
  ```
* **Stream API** для работы с коллекциями:

  ```java
  var numbers = List.of(1, 2, 3);
  assertEquals(6, numbers.stream().mapToInt(Integer::intValue).sum());
  ```

---

## 🔗 Источники

* [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
* [Test Method Order in JUnit](https://junit.org/junit5/docs/current/user-guide/#writing-tests-test-order)

---

[**← Назад к оглавлению**](../README.md)