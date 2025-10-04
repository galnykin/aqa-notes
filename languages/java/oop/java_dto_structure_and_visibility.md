# DTO и структура классов: когда допустимы public поля

**DTO (Data Transfer Object)** — это объект, предназначенный исключительно для передачи данных между слоями приложения, например, между контроллером и сервисом, или между клиентом и сервером. В отличие от бизнес-логики, DTO не содержит поведения — только структуру. Это позволяет упростить сериализацию, десериализацию и повысить читаемость кода.

---

## 📦 Что такое DTO

DTO — это разновидность POJO, которая:
- содержит только поля и геттеры/сеттеры;
- не содержит логики;
- может использоваться для сериализации в JSON/XML;
- часто применяется в REST API, при работе с формами, запросами, ответами.

---

## ✅ Когда допустимы public поля

Если класс используется **только как контейнер данных**, допустимо оставить поля `public`, особенно если:
- нет необходимости в инкапсуляции;
- класс не содержит логики;
- сериализация/десериализация работает напрямую с полями;
- используется в тестах, временных структурах, JSON-моделях.

### Пример DTO с public полями:
```java
public class UserDTO {
    public String username;
    public String email;
    public int age;
}
```

Это допустимо, особенно в простых API-моделях, где важна скорость и компактность.

---

## 🔒 Когда лучше использовать private поля и геттеры/сеттеры

- Если класс будет расширяться или использоваться в бизнес-логике
- Если требуется контроль над доступом к данным
- Если используется фреймворк, который требует JavaBeans-стиль (`private` + `get/set`)
- Если важно соблюдение принципов ООП

### Пример DTO с инкапсуляцией:
```java
public class ProductDTO {
    private String name;
    private double price;

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public double getPrice() { return price; }
    public void setPrice(double price) { this.price = price; }
}
```

---

## 🧰 Использование DTO в проектах

- **Spring Boot**: для передачи данных между контроллерами и сервисами
- **Jackson / Gson**: для сериализации JSON
- **MapStruct / ModelMapper**: для преобразования между сущностями и DTO
- **Swagger / OpenAPI**: для генерации документации по DTO

---

## 🔗 Источники:

- [DTO Explained — Baeldung](https://www.baeldung.com/java-dto-pattern)
- [POJO vs DTO — GeeksforGeeks](https://www.geeksforgeeks.org/difference-between-pojo-and-dto-in-java/)
- [JavaBeans Specification](https://www.oracle.com/java/technologies/javabeans.html)
- [Jackson JSON Processor](https://github.com/FasterXML/jackson)

---
[**← Назад к оглавлению**](../README.md)
