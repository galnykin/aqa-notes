# JavaBeans-стиль: зачем нужен и где используется

**JavaBeans-стиль** — это соглашение о структуре классов в Java, при котором поля объявляются как `private`, а доступ к ним осуществляется через публичные геттеры и сеттеры. Такой подход обеспечивает инкапсуляцию и совместимость с множеством Java-фреймворков, библиотек и инструментов.

---

## 📦 Структура JavaBean

Класс в стиле JavaBean должен:
- иметь публичный конструктор без аргументов;
- содержать `private` поля;
- предоставлять `public` геттеры и сеттеры;
- быть сериализуемым (опционально).

### Пример:
```java
public class User {
    private String name;
    private int age;

    public User() {}

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }
}
```

---

## 🧩 Зачем нужен JavaBeans-стиль

### 1. **Совместимость с фреймворками**
Многие фреймворки используют рефлексию и требуют геттеров/сеттеров для работы с объектами:

- **Spring Framework** — для бин-инъекций, сериализации, валидации
- **Jackson / Gson** — для JSON-сериализации/десериализации
- **Hibernate / JPA** — для ORM и работы с базой данных
- **JSF / JSP / EL** — для отображения данных в UI

### 2. **Инкапсуляция**
Позволяет контролировать доступ к полям, добавлять валидацию, логику, защиту от изменения.

### 3. **Универсальность**
JavaBeans легко использовать в IDE, инструментах генерации кода, XML-конфигурациях, шаблонизаторах.

---

## ⚠️ Когда JavaBeans-стиль обязателен

- При использовании **Spring Boot** с `@ConfigurationProperties`
- При работе с **JPA Entity** классами
- При сериализации с **Jackson**, если нет аннотаций `@JsonProperty`
- При использовании **JSF**, где EL (`#{user.name}`) требует геттеров

---

## 🔗 Источники:

- [JavaBeans Specification — Oracle](https://www.oracle.com/java/technologies/javabeans.html)
- [Spring Boot Configuration Properties](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html)
- [Jackson Serialization Guide](https://github.com/FasterXML/jackson)
- [Hibernate Entity Mapping](https://hibernate.org/orm/documentation/)

---
[**← Назад к оглавлению**](../README.md)
