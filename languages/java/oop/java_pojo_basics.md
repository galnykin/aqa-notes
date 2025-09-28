# POJO (Plain Old Java Object) 

**POJO** — это аббревиатура от *Plain Old Java Object*, что означает "простой старый Java-объект". Термин используется для обозначения классов, которые не зависят от каких-либо специфических фреймворков, аннотаций, интерфейсов или наследования. Это просто обычные Java-классы, содержащие поля, геттеры/сеттеры и, возможно, конструкторы.

---

## 📦 Признаки POJO

- Не наследует специфические классы (например, `HttpServlet`, `Entity`)
- Не реализует специальные интерфейсы (например, `Serializable`, `Runnable`) — хотя это допустимо
- Не содержит аннотаций фреймворков (`@Entity`, `@Component`, `@Service`)
- Содержит только поля, конструкторы, геттеры/сеттеры, `toString()`, `equals()`, `hashCode()`

---

## 📋 Пример POJO-класса

```java
public class Product {
    private String name;
    private double price;

    public Product() {}

    public Product(String name, double price) {
        this.name = name;
        this.price = price;
    }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public double getPrice() { return price; }
    public void setPrice(double price) { this.price = price; }
}
```

---

## 🧩 Где используется POJO

- **DTO (Data Transfer Object)** — передача данных между слоями
- **Модели JSON/XML** — при сериализации/десериализации
- **Конфигурационные классы** — например, в Spring Boot
- **Тестовые данные** — генерация объектов для автотестов

---

## 🧠 Зачем нужны POJO

- Упрощают структуру кода
- Легко сериализуются в JSON/XML
- Не зависят от фреймворков — легко тестировать
- Используются как основа для ORM (Hibernate, JPA)

---

## 🔗 Источники:

- [Java POJO Tutorial — Baeldung](https://www.baeldung.com/java-pojo-class)
- [POJO Explained — GeeksforGeeks](https://www.geeksforgeeks.org/pojo-in-java/)
- [JavaBeans vs POJO — Stack Overflow](https://stackoverflow.com/questions/2453846/what-is-the-difference-between-a-pojo-and-a-javabean)

---
[**← Назад к оглавлению**](../README.md)
