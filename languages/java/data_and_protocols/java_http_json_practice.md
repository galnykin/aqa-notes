# Java-контроллеры, HTTP-заголовки и работа с JSON-ответами: практика для автоматизаторов

В веб-приложениях на Java контроллеры — это точки входа для обработки HTTP-запросов. Они принимают данные от клиента, обрабатывают их и формируют ответ. Понимание структуры запроса и ответа, типов заголовков и форматов данных — ключевой навык для автоматизатора, особенно при работе с API и UI.

---

## 🧭 Контроллеры в Java: получение запросов и формирование ответов

В Spring Boot контроллеры создаются с помощью аннотаций `@RestController` и `@RequestMapping`. Они могут принимать параметры, заголовки, тело запроса и возвращать данные в разных форматах.

### Пример контроллера:
```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    @GetMapping("/{id}")
    public ResponseEntity<Product> getProduct(@PathVariable Long id) {
        Product product = productService.findById(id);
        return ResponseEntity.ok(product);
    }

    @PostMapping
    public ResponseEntity<Product> createProduct(@RequestBody Product product) {
        Product saved = productService.save(product);
        return ResponseEntity.status(HttpStatus.CREATED).body(saved);
    }
}
```

### Возврат разных типов:
- `ResponseEntity<T>` — позволяет управлять статусом, заголовками и телом.
- `String`, `Map`, `List`, `DTO` — любые объекты, сериализуемые в JSON/XML.

---

## 📦 HTTP-заголовки: минимальный набор

Заголовки — это метаинформация, передаваемая вместе с запросом или ответом.

### Минимальные заголовки запроса:
| Заголовок     | Назначение                                |
|---------------|--------------------------------------------|
| `Host`        | Имя сервера, к которому направлен запрос   |
| `Accept`      | Типы данных, которые клиент ожидает        |
| `User-Agent`  | Информация о клиенте (браузер, версия)     |
| `Content-Type`| Тип данных в теле запроса (`application/json`) |
| `Authorization`| Токен или данные авторизации              |

### Заголовки ответа:
- `Content-Type: application/json`
- `Cache-Control`, `Set-Cookie`, `Access-Control-Allow-Origin`

В Java можно получить заголовки через `@RequestHeader`:
```java
@GetMapping
public ResponseEntity<String> getHeader(@RequestHeader("User-Agent") String userAgent) {
    return ResponseEntity.ok("Ваш клиент: " + userAgent);
}
```

---

## 🧾 Content-Type и типы ответа

Ответ от сервера может быть:
- **JSON** — структурированные данные (наиболее распространённый формат для API)
- **HTML** — страница или фрагмент HTML
- **XML** — используется в SOAP или старых системах
- **Text** — простой текст

### Пример JSON:
```json
{
  "id": 1,
  "name": "Pizza",
  "price": 9.99,
  "available": true,
  "tags": ["cheese", "vegetarian"],
  "details": null
}
```

Типы данных в JSON:
- число (`9.99`)
- строка (`"Pizza"`)
- булеан (`true`)
- `null`
- массив (`["cheese", "vegetarian"]`)
- объект/класс (`{ "id": 1, "name": "Pizza" }`)

---

## 🌐 Подходящий сайт с хорошими JSON-ответами

Для практики можно использовать:

- [https://jsonplaceholder.typicode.com](https://jsonplaceholder.typicode.com) — фейковый API для тестов
- [https://reqres.in](https://reqres.in) — API с примерами авторизации, пользователей
- [https://api.publicapis.org](https://api.publicapis.org) — список публичных API
- [https://pokeapi.co](https://pokeapi.co) — API по Pokémon, с вложенными JSON

---

## 🛠️ DevTools: вкладка Sources и Snippets

В браузере Chrome DevTools:
- **Sources** → можно писать и сохранять JavaScript-сниппеты.
- **Snippets** → вкладка для создания повторно используемых скриптов.
- Пример сниппета:
```javascript
console.log("Текущий URL:", window.location.href);
```

Полезно для анализа поведения страницы, отладки UI и API.

---

## 🧹 Форматирование JSON

Для удобного чтения и анализа JSON можно использовать онлайн-инструменты:

- [https://jsonformatter.org](https://jsonformatter.org)
- [https://jsoneditoronline.org](https://jsoneditoronline.org)
- [https://jsonpath.com](https://jsonpath.com) — для извлечения данных из JSON по выражениям

---

## 🔗 Источники:
- [Spring Boot REST Docs](https://spring.io/guides/gs/rest-service/)
- [MDN HTTP Headers Reference](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)
- [JSONPlaceholder](https://jsonplaceholder.typicode.com/)
- [JsonPath.com](https://jsonpath.com/)
- [Chrome DevTools Snippets](https://developer.chrome.com/docs/devtools/snippets/)

---
[**← Назад к оглавлению**](../README.md)
