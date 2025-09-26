# Основы объектно-ориентированного программирования на Java

Объектно-ориентированное программирование (ООП) — это парадигма, лежащая в основе языка Java. Она позволяет структурировать код в виде объектов, объединяющих данные и поведение. ООП делает программы более читаемыми, масштабируемыми и удобными для тестирования.

---

## 🧩 Инкапсуляция и полиморфизм

**Инкапсуляция** — это сокрытие внутренней реализации объекта и предоставление доступа только через публичные методы. Это позволяет защитить данные от некорректного использования.

Пример:
```java
public class User {
    private String name; // скрытое поле

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

**Полиморфизм** — способность объектов вести себя по-разному в зависимости от контекста. Это достигается через наследование и переопределение методов.

Пример:
```java
public class Animal {
    public void speak() {
        System.out.println("Animal sound");
    }
}

public class Dog extends Animal {
    @Override
    public void speak() {
        System.out.println("Woof");
    }
}
```

---

## 🏗️ Определение класса

Класс — это шаблон для создания объектов. Он описывает свойства (поля) и поведение (методы).

Пример:
```java
public class Car {
    String model;
    int year;

    void drive() {
        System.out.println("Машина едет");
    }
}
```

Классы могут быть публичными (`public`) или иметь пакетную видимость (`package-private`).

---

## 🧱 Создание объектов

Объекты создаются с помощью ключевого слова `new`, которое вызывает конструктор класса.

Пример:
```java
Car myCar = new Car();
myCar.model = "Toyota";
myCar.drive();
```

Объект — это экземпляр класса, обладающий собственным состоянием.

---

## 🛠️ Определение методов

Метод — это блок кода, выполняющий определённую задачу. Он имеет сигнатуру, включающую:

- Имя метода
- Параметры
- Возвращаемый тип

Пример:
```java
public int sum(int a, int b) {
    return a + b;
}
```

### Перегрузка методов (`overload`)
Один и тот же метод может иметь разные параметры:

```java
public void print(String text) {
    System.out.println(text);
}

public void print(int number) {
    System.out.println(number);
}
```

---

## 🔐 Модификаторы доступа

Модификаторы определяют, кто может обращаться к элементам класса:

- `public` — доступен всем
- `private` — доступен только внутри класса
- `package-private` (по умолчанию) — доступен внутри пакета

Пример:
```java
public class Account {
    private double balance; // доступ только внутри класса
}
```

---

## ⚙️ Ключевое слово `static`

`static` означает, что элемент принадлежит классу, а не конкретному объекту.

Пример:
```java
public class MathUtils {
    public static int square(int x) {
        return x * x;
    }
}
```

Вызов:
```java
int result = MathUtils.square(5);
```

`static` часто используется в утилитарных классах и при написании тестов.

---

## 🧰 Утилитарные классы

Утилитарные (`util`) классы содержат статические методы, не требующие создания объекта.

Пример:
```java
public class StringUtils {
    public static boolean isEmpty(String s) {
        return s == null || s.isEmpty();
    }
}
```

Такие классы часто используются в тестах для обработки строк, чисел, дат и др.

---

## 🧬 Поля и переменные класса

**Поля** — переменные, определённые внутри класса. Они могут быть:

- **Экземплярными** — принадлежат объекту
- **Статическими** — принадлежат классу

Пример:
```java
public class Counter {
    static int totalCount = 0; // общее количество
    int instanceCount = 0;     // количество для объекта
}
```

---

## 🔒 Ключевое слово `final`

`final` используется для объявления неизменяемых переменных, методов и классов.

### Примеры:
- Переменная:
  ```java
  final int MAX_USERS = 100;
  ```
- Метод:
  ```java
  public final void log() { ... }
  ```
- Класс:
  ```java
  public final class Constants { ... }
  ```

`final` полезен для защиты от переопределения и случайных изменений.

---

## 🧱 Конструкторы

Конструктор — это специальный метод, вызываемый при создании объекта. Он может принимать параметры.

Пример:
```java
public class User {
    String name;

    public User(String name) {
        this.name = name;
    }
}
```

Можно создавать несколько конструкторов (перегрузка).

---

## 📐 Принципы проектирования: DRY, KISS, YAGNI

Эти принципы помогают писать чистый и поддерживаемый код:

- **DRY (Don't Repeat Yourself)** — избегать дублирования
- **KISS (Keep It Simple, Stupid)** — простота важнее сложности
- **YAGNI (You Aren't Gonna Need It)** — не реализовывать то, что не нужно сейчас

Применение этих принципов особенно важно при написании тестов, создании архитектуры и работе в команде.

---

## 🔗 Источники:
- [Java Language Specification](https://docs.oracle.com/javase/specs/)
- [Java Classes and Objects](https://docs.oracle.com/javase/tutorial/java/javaOO/classes.html)
- [Effective Java by Joshua Bloch](https://www.oreilly.com/library/view/effective-java/9780134686097/)
- [Clean Code by Robert C. Martin](https://www.oreilly.com/library/view/clean-code/9780136083238/)

---
[**← Назад к оглавлению**](../README.md)