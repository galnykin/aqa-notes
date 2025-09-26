# Типы данных и операторы Java

Ключевые элементы языка Java, которые лежат в основе любой автоматизации: типы данных, массивы, строки, операторы и циклы. Эти конструкции активно используются при написании тестов, обработке данных, проверке условий и генерации отчётов. Понимание этих основ критично для уверенного владения инструментами вроде JUnit, Selenium и Rest Assured.

---

## 🔢 Типы данных в Java

Java — строго типизированный язык, и каждая переменная должна иметь определённый тип. Типы делятся на:

### Примитивные типы:
- `int` — целое число: `int count = 5;`
- `double` — число с плавающей точкой: `double price = 19.99;`
- `boolean` — логическое значение: `boolean isValid = true;`
- `char` — одиночный символ: `char grade = 'A';`

### Ссылочные типы:
- `String` — строка символов
- Массивы (`int[]`, `String[]`)
- Классы и интерфейсы

---

## ➕ Операторы Java

Операторы позволяют выполнять действия над переменными и выражениями.

### Арифметические:
```java
int sum = a + b;
int diff = a - b;
int prod = a * b;
int div = a / b;
int mod = a % b;
```

### Сравнения:
```java
a == b   // равно
a != b   // не равно
a > b    // больше
a < b    // меньше
```

### Логические:
```java
a && b   // логическое И
a || b   // логическое ИЛИ
!a       // логическое НЕ
```

### Присваивания:
```java
x = 10;
x += 5;  // x = x + 5
x *= 2;  // x = x * 2
```

---

## 📦 Массивы в Java

Массив — это структура данных, хранящая фиксированное количество элементов одного типа.

### Создание массива:
```java
int[] numbers = new int[5];           // массив из 5 элементов
String[] names = {"Alice", "Bob"};    // инициализация с данными
```

### Перебор элементов:
```java
for (int i = 0; i < numbers.length; i++) {
    System.out.println(numbers[i]);
}
```

### Действия с элементами:
```java
numbers[0] = 10;
numbers[1] += 5;
```

### Сортировка:
```java
import java.util.Arrays;

int[] scores = {90, 70, 80};
Arrays.sort(scores);  // [70, 80, 90]
```

### Фильтрация:
```java
for (int score : scores) {
    if (score >= 80) {
        System.out.println("Прошёл: " + score);
    }
}
```

---

## 🧵 Символьные строки (`String`)

`String` — это класс, представляющий последовательность символов. Строки неизменяемы.

### Создание:
```java
String name = "Sergej";
String empty = new String();
```

### Операции:
```java
name.length();               // длина строки
name.toUpperCase();          // в верхний регистр
name.contains("r");          // содержит ли символ
name.equals("Sergej");       // сравнение
name.substring(0, 3);        // подстрока
name.replace("e", "3");      // замена символов
```

### Конкатенация:
```java
String fullName = "QA " + name;
```

---

## 🔁 Цикл `for-each`

Цикл `for-each` — удобный способ перебора коллекций и массивов:

```java
String[] browsers = {"Chrome", "Firefox", "Edge"};

for (String browser : browsers) {
    System.out.println("Тестируем в: " + browser);
}
```

Используется в тестах для перебора тестовых данных, конфигураций, параметров.

---

## ❓ Тернарный оператор

Тернарный оператор — компактная форма условного выражения:

```java
String result = (score >= 75) ? "Пройдено" : "Не пройдено";
```

Эквивалентно:
```java
if (score >= 75) {
    result = "Пройдено";
} else {
    result = "Не пройдено";
}
```

Полезен при написании коротких проверок в тестах:
```java
assertEquals((isAdmin ? "Admin" : "User"), actualRole);
```

---

## 🔗 Источники:
- [Java Language Specification](https://docs.oracle.com/javase/specs/)
- [Java API Documentation](https://docs.oracle.com/en/java/javase/17/docs/api/index.html)
- [Java Tutorials by Oracle](https://docs.oracle.com/javase/tutorial/)
- [Baeldung — Java Basics](https://www.baeldung.com/java-tutorial)

---
[**← Назад к оглавлению**](../README.md)