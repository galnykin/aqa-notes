# Интерфейс прикладного программирования (API): структура, форматы, тестирование

API (Application Programming Interface) — это программный интерфейс, позволяющий приложениям обмениваться данными. В контексте веб-разработки и тестирования API — это способ взаимодействия клиента и сервера через HTTP-запросы. Ниже рассмотрены ключевые элементы запроса и ответа, форматы данных, категории кодов состояния и практическое тестирование с помощью Postman.

---

## 📡 Элементы запроса (Request)

Каждый HTTP-запрос состоит из нескольких частей:

### 1. **Адрес (Endpoint)**
- URL, указывающий на конкретный ресурс или сервис.
- Пример:  
  `https://api.example.com/users/123`

### 2. **Заголовки (Headers)**
- Метаинформация, передаваемая вместе с запросом.
- Примеры:
  - `Content-Type: application/json`
  - `Authorization: Bearer <token>`
  - `Accept-Language: en-US`

В Java заголовки часто оформляются как `Map<String, String>`.

### 3. **Параметры поиска (Query Parameters)**
- Передаются в URL после знака `?`.
- Пример:  
  `https://api.example.com/products?category=pizza&sort=price`

### 4. **Тело запроса (Body / Payload / Data)**
- Основное содержимое запроса, особенно для `POST`, `PUT`, `PATCH`.
- Форматы: JSON, XML, form-data, x-www-form-urlencoded.
- Пример JSON:
  ```json
  {
    "username": "admin",
    "password": "1234"
  }
  ```

---

## 📥 Элементы ответа (Response)

Ответ от сервера также состоит из нескольких частей:

### 1. **Код состояния (Status Code)**
- Числовой код, отражающий результат обработки запроса.

### 2. **Заголовки (Headers)**
- Метаинформация о типе контента, времени ответа, cookies и т. д.
- Примеры:
  - `Content-Type: application/json`
  - `Set-Cookie: sessionId=abc123`

### 3. **Тело ответа (Body)**
- Основные данные, возвращаемые сервером.
- Форматы: JSON, XML, HTML, текст.

Пример JSON-ответа:
```json
{
  "id": 123,
  "name": "Pizza Margherita",
  "price": 9.99
}
```

---

## 🔢 Категории HTTP-кодов состояния

| Категория | Диапазон | Назначение                         |
|-----------|----------|------------------------------------|
| 1xx       | 100–199  | Информационные                     |
| 2xx       | 200–299  | Успешные операции                  |
| 3xx       | 300–399  | Перенаправления                   |
| 4xx       | 400–499  | Ошибки клиента                     |
| 5xx       | 500–599  | Ошибки сервера                     |

### Примеры:
- `200 OK` — успешный запрос
- `201 Created` — ресурс создан
- `400 Bad Request` — ошибка в запросе
- `401 Unauthorized` — требуется авторизация
- `404 Not Found` — ресурс не найден
- `500 Internal Server Error` — ошибка на сервере

---

## 📄 Форматы тела запроса и ответа

### JSON (JavaScript Object Notation)
- Легковесный, читаемый формат
- Широко используется в REST API
- Пример:
  ```json
  {
    "product": "Pizza",
    "price": 12.5
  }
  ```

### XML (eXtensible Markup Language)
- Более формальный, используется в SOAP и некоторых старых API
- Пример:
  ```xml
  <product>
    <name>Pizza</name>
    <price>12.5</price>
  </product>
  ```

---

## 🧪 Функциональное тестирование API с помощью Postman

**Postman** — инструмент для ручного и автоматизированного тестирования API.

### Возможности:
- Отправка запросов всех типов: `GET`, `POST`, `PUT`, `DELETE`
- Настройка заголовков, параметров, тела
- Сохранение коллекций и сценариев
- Проверка ответов с помощью встроенных тестов (JavaScript)
- Интеграция с CI/CD

### Пример запроса:
- Метод: `POST`
- URL: `https://api.example.com/login`
- Headers:
  - `Content-Type: application/json`
- Body (raw JSON):
  ```json
  {
    "username": "admin",
    "password": "1234"
  }
  ```

### Пример теста в Postman:
```javascript
pm.test("Статус 200", function () {
    pm.response.to.have.status(200);
});

pm.test("Ответ содержит токен", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.token).to.not.be.undefined;
});
```

---

## 🔗 Источники:
- [MDN Web Docs — HTTP Overview](https://developer.mozilla.org/en-US/docs/Web/HTTP)
- [Postman Learning Center](https://learning.postman.com/)
- [REST API Concepts](https://restfulapi.net/)
- [Swagger Documentation](https://swagger.io/docs/)

---
[**← Назад к оглавлению**](../../README.md)
