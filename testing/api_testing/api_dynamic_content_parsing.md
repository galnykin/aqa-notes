# Практика работы с API, динамическим контентом и форматами данных: инструменты и задачи

Автоматизатору важно понимать динамический контент на веб-страницах, работу API, форматы данных и инструменты для тестирования. Ниже — расширенный обзор с практическими примерами на Java, подходами к тестированию и интеграцией с REST Assured, TestNG и Swagger.

---

## ⚙️ JS + HTML: динамический контент и обработка событий

HTML-элементы могут реагировать на действия пользователя через JavaScript:

```html
<button onclick="loadData()">Загрузить</button>
<div id="output"></div>
```

```javascript
function loadData() {
  fetch("/api/products")
    .then(response => response.json())
    .then(data => {
      document.getElementById("output").innerText = data.name;
    })
    .catch(err => console.error(err));
}
```

* Используется **AJAX / fetch / XHR** для обновления данных без перезагрузки страницы
* Позволяет динамически изменять DOM в зависимости от ответа сервера

**Пример автоматизации UI-теста через Selenium:**

```java
WebElement button = driver.findElement(By.id("loadButton"));
button.click();
WebElement output = new WebDriverWait(driver, Duration.ofSeconds(5))
    .until(ExpectedConditions.visibilityOfElementLocated(By.id("output")));
assertEquals("Pizza", output.getText());
```

---

## 🌐 API: взаимодействие клиента и сервера

* Методы: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`
* Форматы данных: JSON, XML, Form-data
* Передача данных: заголовки, query-параметры, path-параметры, тело запроса

**Пример JSON-запроса для авторизации:**

```json
{
  "username": "admin",
  "password": "1234"
}
```

**Пример автоматизации с REST Assured:**

```java
given()
    .contentType(ContentType.JSON)
    .body("{\"username\":\"admin\",\"password\":\"1234\"}")
.when()
    .post("/login")
.then()
    .statusCode(200)
    .body("token", notNullValue());
```

---

## 🔄 XML и JSON: сериализация и десериализация

* **Сериализация** — объект → строка (JSON/XML) для передачи
* **Десериализация** — строка → объект

**JSON (REST API)**:

* Простой, читаемый, поддерживает массивы и объекты

```java
ObjectMapper mapper = new ObjectMapper();
Product product = mapper.readValue(jsonString, Product.class);
```

**XML (SOAP / старые API)**:

```java
JAXBContext context = JAXBContext.newInstance(Product.class);
Product product = (Product) context.createUnmarshaller().unmarshal(new File("product.xml"));
```

**HTML (JSoup)**:

```java
Document doc = Jsoup.connect("https://example.com").get();
String title = doc.title();
Elements rows = doc.select("table#products tr");
```

---

**Пример динамического запроса с REST Assured:**

```java
given()
    .queryParam("category", "pizza")
    .header("Accept", "application/json")
.when()
    .get("/products")
.then()
    .statusCode(200)
    .body("products.size()", greaterThan(0));
```

---

## 📄 Swagger и OpenAPI: документация и тестирование

Swagger позволяет автоматически документировать API на основе аннотаций Spring Boot.

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    @Operation(summary = "Получить продукт по ID")
    @GetMapping("/{id}")
    public Product getProduct(@PathVariable Long id) { ... }
}
```

**Swagger UI отображает**:

* список endpoints, методы, параметры
* примеры запросов и ответов
* возможность тестирования прямо из браузера

**Аналоги и дополнения**:

* OpenAPI — стандарт документации
* Postman — ручное тестирование и коллекции
* Redoc, Stoplight — генерация красивой документации
* SpringDoc — интеграция Swagger в Spring Boot

---

## 🧪 curl: проверка API через терминал

```bash
curl -X POST https://api.example.com/login \
-H "Content-Type: application/json" \
-d '{"username":"admin","password":"1234"}'
```

* Удобно для CI/CD, скриптов, быстрого тестирования без UI
* Можно проверять коды состояния, заголовки и тело ответа

---

## 🔗 Источники

* [Swagger Documentation](https://swagger.io/docs/)
* [Jsoup HTML Parser](https://jsoup.org/)
* [Jackson JSON Processor](https://github.com/FasterXML/jackson)
* [JAXB XML Binding](https://docs.oracle.com/javase/tutorial/jaxb/)
* [curl Manual](https://curl.se/docs/manual.html)
* [JsonPath](https://jsonpath.com/)
* [REST Assured Documentation](https://rest-assured.io/)

---

[**← Назад к оглавлению**](README.md)
