# Практика работы с API, динамическим контентом и форматами данных: инструменты и задачи

Автоматизатору важно понимать, как работает динамический контент на веб-страницах, как устроен API, какие форматы данных используются, и как документировать и тестировать интерфейсы. Ниже — обзор ключевых тем, связанных с JavaScript, HTML, API, XML/JSON, Swagger и curl.

---

## ⚙️ JS + HTML: `onclick` и динамический контент

HTML-элементы могут реагировать на действия пользователя через JavaScript:

```html
<button onclick="loadData()">Загрузить</button>
```

```javascript
function loadData() {
  fetch("/api/products")
    .then(response => response.json())
    .then(data => {
      document.getElementById("output").innerText = data.name;
    });
}
```

Такой подход используется для динамической загрузки данных без перезагрузки страницы (AJAX, fetch/XHR).

---

## 🌐 Интерфейс прикладного программирования (API)

API — это набор методов, через которые клиент (браузер, приложение) взаимодействует с сервером.

- **Методы**: `GET`, `POST`, `PUT`, `DELETE`
- **Форматы данных**: JSON, XML
- **Передача данных**: через заголовки, параметры, тело запроса

Пример JSON-запроса:
```json
{
  "username": "admin",
  "password": "1234"
}
```

---

## 🔄 XML и JSON: сериализация и передача данных

- **Сериализация** — преобразование объекта в строку (JSON/XML) для передачи.
- **Десериализация** — обратное преобразование строки в объект.

### JSON:
- Простой, читаемый, используется в REST API.
- Типы: число, строка, булеан, null, массив, объект.

### XML:
- Более формальный, используется в SOAP и старых системах.
- Требует схемы (XSD) для валидации.

---

## 🧩 Классы для парсинга: JSONClass, XMLClass, HTMLClass

В Java можно использовать:

- **Jackson / Gson** — для JSON:
  ```java
  ObjectMapper mapper = new ObjectMapper();
  Product product = mapper.readValue(jsonString, Product.class);
  ```

- **JAXB / DOM / SAX** — для XML:
  ```java
  JAXBContext context = JAXBContext.newInstance(Product.class);
  Product product = (Product) context.createUnmarshaller().unmarshal(new File("product.xml"));
  ```

- **Jsoup** — для HTML:
  ```java
  Document doc = Jsoup.connect("https://example.com").get();
  String title = doc.title();
  ```

---

## 📝 Домашнее задание: парсинг XML и HTML

1. Скачать XML-файл с продуктами.
2. Написать Java-класс `Product`.
3. Распарсить XML с помощью JAXB.
4. Скачать HTML-страницу с таблицей.
5. Извлечь данные с помощью Jsoup.

---

## 📄 Swagger: генерация документации по контроллерам

**Swagger** — инструмент для автоматической генерации документации API на основе аннотаций в коде.

### Пример аннотаций:
```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    @Operation(summary = "Получить продукт по ID")
    @GetMapping("/{id}")
    public Product getProduct(@PathVariable Long id) { ... }
}
```

Swagger UI отображает:
- список endpoints;
- методы;
- параметры;
- примеры запросов и ответов.

---

## 🧪 curl: отправка запросов из терминала

```bash
curl -X POST https://api.example.com/login \
-H "Content-Type: application/json" \
-d '{"username":"admin","password":"1234"}'
```

Полезен для быстрой проверки API, особенно в CI/CD и скриптах.

---

## 🔁 Аналоги Swagger

| Инструмент      | Назначение                        |
|-----------------|------------------------------------|
| **OpenAPI**     | Стандарт, на котором основан Swagger |
| **Postman**     | Ручное тестирование и документация |
| **Redoc**       | Генерация документации из OpenAPI |
| **Stoplight**   | Визуальное моделирование API       |
| **SpringDoc**   | Автоматическая интеграция Swagger с Spring Boot |

---

## 🔗 Источники:
- [Swagger Documentation](https://swagger.io/docs/)
- [Jsoup HTML Parser](https://jsoup.org/)
- [Jackson JSON Processor](https://github.com/FasterXML/jackson)
- [JAXB XML Binding](https://docs.oracle.com/javase/tutorial/jaxb/)
- [curl Manual](https://curl.se/docs/manual.html)
- [JsonPath.com](https://jsonpath.com/)

---
[**← Назад к оглавлению**](README.md)
