# Практика написания тестов и современные возможности Java

В этом материале собраны ключевые элементы, связанные с написанием тестов на Java, включая аннотации, структуру тестов, порядок выполнения, а также современные синтаксические возможности языка. Эти знания важны для построения надёжной системы автоматизации и повышения читаемости и устойчивости тестов.

---

## 🧪 Аннотация `@Test` и описание тестов

Аннотация `@Test` используется для обозначения метода как тестового. Она применяется в JUnit 4 и JUnit 5.

```java
@Test
void shouldAddTwoNumbers() {
    // тестовая логика
}
```

Для улучшения читаемости можно использовать `@DisplayName`, чтобы задать понятное описание теста:

```java
@DisplayName("Сложение двух положительных чисел")
@Test
void addPositiveNumbers() {
    assertEquals(4, calculator.add(2, 2));
}
```

---

## 🧾 Структура теста: подготовка, действие, результат

Хорошо структурированный тест состоит из трёх частей:

1. **Подготовка** — создание объектов, настройка данных.
2. **Действие** — вызов метода, который тестируется.
3. **Результат** — проверка ожидаемого результата.

Эта структура известна как **AAA** — Arrange, Act, Assert:

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

Также используется шаблон **given-when-then**, особенно в BDD (Behavior-Driven Development):

```java
@Test
void givenTwoNumbers_whenAdded_thenReturnsSum() {
    Calculator calc = new Calculator();
    int result = calc.add(2, 3);
    assertEquals(5, result);
}
```

---

## 🔄 Жизненный цикл тестов: `@BeforeEach`, `@AfterEach`, `@BeforeAll`, `@AfterAll`

JUnit предоставляет аннотации для управления подготовкой и очисткой перед и после тестов.

### `@BeforeEach`
Вызывается перед каждым тестом. Используется для инициализации:

```java
@BeforeEach
void createCalculator() {
    calculator = new Calculator();
}
```

Вместо `setUp()` или `beforeMethod()` лучше использовать контекстное имя, отражающее суть подготовки.

### `@AfterEach`
Вызывается после каждого теста. Используется для очистки:

```java
@AfterEach
void cleanUp() {
    calculator = null;
}
```

### `@BeforeAll` и `@AfterAll`
Вызываются один раз до и после всех тестов. Методы должны быть `static`.

```java
@BeforeAll
static void initSuite() {
    System.out.println("Начало тестов");
}

@AfterAll
static void tearDownSuite() {
    System.out.println("Завершение тестов");
}
```

---

## 🔢 Управление порядком выполнения тестов

По умолчанию тесты выполняются в произвольном порядке. Для явного управления используется `@TestMethodOrder`.

```java
@TestMethodOrder(MethodOrderer.MethodName.class)
class OrderedTests {

    @Order(1)
    @Test
    void testA() { ... }

    @Order(2)
    @Test
    void testB() { ... }
}
```

Можно использовать другие стратегии: `MethodOrderer.OrderAnnotation`, `MethodOrderer.Random`, `MethodOrderer.Alphanumeric`.

---

## 🧠 Функциональные интерфейсы и лямбда-выражения

Функциональные интерфейсы — это интерфейсы с единственным абстрактным методом. Они лежат в основе лямбда-выражений, пришедших из JavaScript.

Пример на JavaScript:
```javascript
const result = (a, b) => a + b;
console.log(result(4, 4)); // 8

const result2 = (a, b) => { return a - b };
console.log(result2(4, 4)); // 0
```

Аналог на Java:
```java
@FunctionalInterface
interface Operation {
    int apply(int a, int b);
}

Operation sum = (a, b) -> a + b;
System.out.println(sum.apply(4, 4)); // 8
```

Функциональные интерфейсы активно используются в `Stream API`, `Comparator`, `Runnable`, `Predicate`.

---

## 🔄 Современный `switch` — выражение

Начиная с Java 14, появился улучшенный `switch`, называемый **switch expression**. Он возвращает значение и поддерживает стрелочный синтаксис.

Пример:
```java
String result = switch (day) {
    case "MON" -> "Понедельник";
    case "TUE" -> "Вторник";
    default -> "Неизвестно";
};
```

Также можно использовать блоки:
```java
int length = switch (word) {
    case "Java" -> {
        int len = word.length();
        yield len;
    }
    default -> 0;
};
```

---

## 🧪 Дополнительные аннотации и утверждения

### `@CsvSource`
Используется для параметризации тестов:
```java
@ParameterizedTest
@CsvSource({
    "2, 3, 5",
    "10, 5, 15"
})
void testAdd(int a, int b, int expected) {
    assertEquals(expected, calculator.add(a, b));
}
```

### `@Disabled`
Отключает тест:
```java
@Disabled("Временно отключён")
@Test
void testFeatureInProgress() {
    // не будет выполнен
}
```

### Утверждения:
```java
assertTrue(condition);   // проверка, что условие истинно
assertFalse(condition);  // проверка, что условие ложно
```

---

## 🔗 Источники:
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [Java Functional Interfaces](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)
- [Java 14 Switch Expressions](https://openjdk.org/jeps/361)
- [Test Method Order in JUnit](https://junit.org/junit5/docs/current/user-guide/#writing-tests-test-order)

---
[**← Назад к оглавлению**](../../../README.md)