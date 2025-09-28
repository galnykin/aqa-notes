# Хранить поле или вычислять динамически: архитектурный выбор в Java

При проектировании классов в Java разработчик часто сталкивается с вопросом: **стоит ли хранить значение в поле**, или **вычислять его при каждом обращении**. Это решение влияет на производительность, читаемость, гибкость и корректность работы приложения. В этой статье рассмотрим критерии выбора, примеры реализации и архитектурные рекомендации.

---

## 📦 Хранение значения в поле

### Когда это оправдано:

- **Вычисление ресурсоёмкое** — например, обращение к базе данных, файловой системе, сетевым ресурсам, сложные вычисления.
- **Значение не изменяется** после инициализации (или меняется редко).
- **Частый доступ** к значению — например, в UI, логике сравнения, сериализации.
- **Нужно сохранить значение** при сериализации (например, в JSON).
- **Используется в `equals()` или `hashCode()`** — важно, чтобы значение было стабильным.

### Пример:
```java
public class Product {
    private final double priceWithTax;

    public Product(double basePrice, double taxRate) {
        this.priceWithTax = basePrice * (1 + taxRate);
    }

    public double getPriceWithTax() {
        return priceWithTax;
    }
}
```

---

## 🔁 Вычисление значения динамически

### Когда это предпочтительно:

- **Значение зависит от других полей**, которые могут изменяться.
- **Вычисление дешёвое** — простая арифметика, конкатенация, доступ к кэшу.
- **Нужно получать актуальное значение** при каждом вызове.
- **Нет необходимости в сериализации** или кэшировании.

### Пример:
```java
public class Product {
    private double basePrice;
    private double taxRate;

    public double getPriceWithTax() {
        return basePrice * (1 + taxRate);
    }
}
```

---

## ⚖️ Компромисс: ленивое вычисление и кэш

Если значение вычисляется дорого, но может меняться — можно использовать **ленивую инициализацию с кэшем**, сбрасывая его при изменении зависимых данных.

### Пример:
```java
public class Product {
    private double basePrice;
    private double taxRate;
    private Double cachedPriceWithTax;

    public double getPriceWithTax() {
        if (cachedPriceWithTax == null) {
            cachedPriceWithTax = basePrice * (1 + taxRate);
        }
        return cachedPriceWithTax;
    }

    public void setBasePrice(double basePrice) {
        this.basePrice = basePrice;
        cachedPriceWithTax = null;
    }

    public void setTaxRate(double taxRate) {
        this.taxRate = taxRate;
        cachedPriceWithTax = null;
    }
}
```

---

## 🧠 Архитектурные рекомендации

| Критерий                             | Хранить поле | Вычислять динамически |
|--------------------------------------|--------------|------------------------|
| Вычисление ресурсоёмкое              | ✅            | ❌                    |
| Частый доступ                        | ✅            | ❌                    |
| Зависимость от изменяемых данных     | ❌            | ✅                    |
| Требуется сериализация               | ✅            | ❌                    |
| Простая логика                       | ❌            | ✅                    |
| Используется в `equals()` / `hashCode()` | ✅        | ❌                    |

---

## 🧰 Инструменты и практики

- **Lombok**: можно использовать `@Getter(lazy = true)` для ленивых вычислений
- **Jackson / Gson**: сериализуют поля, но не методы — важно учитывать при выборе
- **MapStruct / ModelMapper**: при маппинге DTO лучше использовать поля
- **Spring Boot**: `@Value` и `@ConfigurationProperties` работают с полями

---

## 🔗 Источники:

- **Effective Java — Joshua Bloch, Item 18: Favor composition over inheritance**
- [Lombok Lazy Getter](https://projectlombok.org/features/GetterLazy)
- [Jackson Serialization Guide](https://github.com/FasterXML/jackson)
- [JavaBeans Specification — Oracle](https://www.oracle.com/java/technologies/javabeans.html)

---
[**← Назад к оглавлению**](../README.md)
