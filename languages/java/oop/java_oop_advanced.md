# Java: Объектно-ориентированное программирование — расширенные концепции

Продолжая изучение ООП в Java, здесь рассматриваются более продвинутые темы, которые активно применяются в разработке и автоматизации тестирования. Наследование, интерфейсы, коллекции, исключения и принципы SOLID — это фундаментальные элементы, позволяющие строить гибкую, расширяемую и поддерживаемую архитектуру.

---

## 🧬 ООП. Наследование

Наследование позволяет одному классу (подклассу) перенимать свойства и методы другого класса (суперкласса). Это способствует повторному использованию кода и созданию иерархий.

Пример:
```java
public class Animal {
    public void speak() {
        System.out.println("Some sound");
    }
}

public class Cat extends Animal {
    public void purr() {
        System.out.println("Purr");
    }
}
```

Объект `Cat` наследует метод `speak()` от `Animal` и добавляет собственный `purr()`.

---

## 🔁 ООП. Переопределение методов (`override`)

Переопределение позволяет подклассу изменить реализацию метода суперкласса.

Пример:
```java
public class Dog extends Animal {
    @Override
    public void speak() {
        System.out.println("Woof");
    }
}
```

Аннотация `@Override` указывает, что метод переопределяется. Это важно для корректной работы полиморфизма.

---

## 📚 ООП. Коллекции

Коллекции — это структуры данных, используемые для хранения и обработки групп объектов.

### `List` — упорядоченный список:
- `ArrayList` — массивная реализация, быстрая по индексу
- `LinkedList` — связный список, эффективен при вставках/удалениях

```java
List<String> names = new ArrayList<>();
names.add("Alice");
names.add("Bob");
```

### `Set` — множество, не допускает дубликатов:
```java
Set<String> uniqueNames = new HashSet<>();
uniqueNames.add("Alice");
uniqueNames.add("Alice"); // не добавится второй раз
```

### `Map` — отображение ключ-значение:
```java
Map<String, Integer> scores = new HashMap<>();
scores.put("Alice", 90);
scores.put("Bob", 85);
```

Коллекции активно используются в тестах для хранения параметров, результатов, конфигураций.

---

## 🔌 ООП. Интерфейсы

Интерфейс — это контракт, определяющий набор методов, которые должен реализовать класс.

Пример:
```java
public interface Clickable {
    void click();
}

public class Button implements Clickable {
    @Override
    public void click() {
        System.out.println("Button clicked");
    }
}
```

Интерфейсы позволяют создавать гибкие архитектуры и легко подменять реализации в тестах (например, через мок-объекты).

---

## 🧠 ООП. Функциональный интерфейс, `lambda`, `stream`

### Функциональный интерфейс — интерфейс с одним абстрактным методом:
```java
@FunctionalInterface
public interface Converter {
    int convert(String value);
}
```

### `lambda` — краткая форма реализации:
```java
Converter toInt = s -> Integer.parseInt(s);
System.out.println(toInt.convert("123")); // 123
```

### `stream` — API для обработки коллекций:
```java
List<String> names = List.of("Alice", "Bob", "Charlie");

names.stream()
     .filter(n -> n.startsWith("B"))
     .map(String::toUpperCase)
     .forEach(System.out::println); // BOB
```

`stream` и `lambda` упрощают обработку данных и часто используются в тестах для фильтрации, агрегации и анализа.

---

## ⚠️ ООП. Исключения

Исключения — механизм обработки ошибок во время выполнения.

### Обработка:
```java
try {
    int result = 10 / 0;
} catch (ArithmeticException e) {
    System.out.println("Ошибка: деление на ноль");
} finally {
    System.out.println("Завершение");
}
```

### Создание собственных исключений:
```java
public class InvalidTestDataException extends RuntimeException {
    public InvalidTestDataException(String message) {
        super(message);
    }
}
```

Исключения позволяют контролировать поведение тестов и корректно реагировать на ошибки.

---

## 🧱 Принципы проектирования SOLID

SOLID — набор принципов, обеспечивающих надёжную архитектуру:

- **S — Single Responsibility Principle (Принцип единственной ответственности)**  
  Каждый класс должен выполнять одну задачу.

- **O — Open/Closed Principle (Принцип открытости/закрытости)**  
  Классы должны быть открыты для расширения, но закрыты для изменения.

- **L — Liskov Substitution Principle (Принцип подстановки Барбары Лисков)**  
  Подклассы должны заменять базовые классы без нарушения логики.

- **I — Interface Segregation Principle (Принцип разделения интерфейсов)**  
  Лучше несколько специализированных интерфейсов, чем один общий.

- **D — Dependency Inversion Principle (Принцип инверсии зависимостей)**  
  Зависимости должны быть от абстракций, а не от конкретных реализаций.

Применение SOLID делает код тестируемым, расширяемым и устойчивым к изменениям.

---

## 🔗 Источники:
- [Java Collections Framework](https://docs.oracle.com/javase/8/docs/technotes/guides/collections/overview.html)
- [Java Interfaces](https://docs.oracle.com/javase/tutorial/java/IandI/createinterface.html)
- [Java Streams and Lambdas](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)
- [SOLID Principles Explained](https://www.baeldung.com/solid-principles)

---
[**← Назад к оглавлению**](../README.md)