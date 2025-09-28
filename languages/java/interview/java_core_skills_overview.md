# Обзор ключевых тем для Java-разработчика: от основ до архитектуры

Ниже представлен структурированный список тем, которые охватывают фундаментальные и продвинутые знания Java, необходимые для уверенного старта в разработке и автоматизации. Эти области охватывают синтаксис, ООП, работу с памятью, многопоточность, сборку проектов, архитектурные принципы и инструменты.

---

## 🧠 Языки программирования и Java-платформа

- Популярные языки: Java, Python, JavaScript, Kotlin, C#, Go, Rust
- **JVM (Java Virtual Machine)** — виртуальная машина, исполняющая байт-код
- **JRE (Java Runtime Environment)** — окружение для запуска Java-программ
- **JDK (Java Development Kit)** — включает JRE + компилятор + инструменты разработки

---

## 🧱 Память и классы в Java

- **Области памяти**: Stack, Heap, Metaspace, Code, Static
- **Классы и объекты** — основа ООП
- **Class Loaders** — загрузчики классов: Bootstrap, Extension, Application
- **Объект класса Class** — отражение метаданных о классе (`Class<?> clazz = String.class`)

---

## 🧩 Абстракции и вложенные классы

- **Абстрактные классы** — частичная реализация, нельзя создать экземпляр
- **Интерфейсы** — контракт, поддерживают `default` и `static` методы
- **Изменяемые и неизменяемые объекты** — `String` — пример неизменяемого
- **Inner и Nested классы** — вложенные классы, доступ к внешнему классу
- **Локальные классы** — внутри метода
- **Анонимные классы** — без имени, реализуют интерфейс или наследуют класс

---

## 🔧 Класс Object и ООП

- `Object` — базовый класс всех объектов
- Принципы ООП: инкапсуляция, наследование, полиморфизм, абстракция
- Оболочки примитивов: `Integer`, `Double`, `Boolean` и др.
- Основное API: `String`, `Math`, `Arrays`, `Collections`, `Date`, `Optional`
- Сравнение значений: `==` vs `.equals()`, `compareTo()`

---

## 📚 Алгоритмы, Generics, Коллекции

- Алгоритмы: сортировка, поиск, рекурсия, жадные, динамическое программирование
- Generics: обобщённые типы (`List<T>`, `Map<K,V>`)
- Коллекции: `List`, `Set`, `Map`, `Queue`, `Deque`, `Collections`, `Streams`

---

## ⚙️ Функциональное программирование

- Лямбда-выражения: `(x) -> x * 2`
- Функциональные интерфейсы: `Predicate`, `Function`, `Consumer`, `Supplier`
- Stream API: `filter()`, `map()`, `collect()`, `reduce()`

---

## 🚨 Исключения, ввод-вывод, сериализация

- Обработка исключений: `try-catch-finally`, `throws`, `throw`
- Ввод-вывод: `Scanner`, `BufferedReader`, `File`, `InputStream`, `OutputStream`
- Сериализация: `Serializable`, `ObjectOutputStream`, `ObjectInputStream`

---

## 🔀 Многопоточность и java.util.concurrent

- Потоки: `Thread`, `Runnable`, `ExecutorService`
- Синхронизация: `synchronized`, `Lock`, `Semaphore`
- Пакет `java.util.concurrent`: `Future`, `Callable`, `CountDownLatch`, `ConcurrentHashMap`

---

## 🧰 Сборщики проектов и Git

- **Gradle** — декларативный, гибкий
- **Maven** — XML-конфигурация, стандартный жизненный цикл
- **Git** — система контроля версий: `clone`, `commit`, `push`, `pull`, `merge`, `branch`

---

## 🧠 Архитектура и паттерны

- Принципы **SOLID**:
    - Single Responsibility
    - Open/Closed
    - Liskov Substitution
    - Interface Segregation
    - Dependency Inversion
- Паттерны проектирования: Singleton, Factory, Strategy, Observer, Builder, Adapter

---

## 🌐 Web-разработка

- **Servlets** — базовая технология Java EE
- **Apache Tomcat** — контейнер сервлетов, сервер приложений
- Основы HTTP, REST, JSON, HTML, JSP

---

## 📌 Требования к кандидатам

- Базовые знания Java: синтаксис, ООП, коллекции, исключения
- Понимание работы с Git: ветки, коммиты, pull/push, разрешение конфликтов

---
## 🔗 Источники:

- [Руководство по Java от Oracle (официальный туториал)](https://docs.oracle.com/javase/tutorial/)
- [Обновления и новые возможности Java (Baeldung)](https://www.baeldung.com/java-new)
- [Обучение Java: статьи и задачи (GeeksforGeeks)](https://www.geeksforgeeks.org/java/)
- [Полный курс Java с примерами (JavaTpoint)](https://www.javatpoint.com/java-tutorial)
- [Паттерны проектирования и принципы SOLID (Refactoring Guru)](https://refactoring.guru/ru/design-patterns)
- [Руководство по началу работы с Maven](https://maven.apache.org/guides/getting-started/)
- [Официальный сайт Gradle](https://gradle.org/)
- [Официальная документация Git](https://git-scm.com/)
- [Сервер приложений Apache Tomcat](https://tomcat.apache.org/)
---
[**← Назад к оглавлению**](../README.md)
