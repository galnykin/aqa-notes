# Тестирование клиент-серверного взаимодействия и API: инструменты, структура, практика

Клиент-серверное взаимодействие — основа работы современных приложений. Клиент (браузер, мобильное приложение) отправляет запросы, сервер обрабатывает их и возвращает ответы. API-тестирование позволяет проверить корректность этих обменов, структуру данных, обработку ошибок и бизнес-логику без участия UI.

---

## 🌐 Клиент и сервер: как устроено взаимодействие

- **Клиент** — браузер, мобильное приложение, десктопное ПО.
- **Сервер** — может быть визуально представлен в окне браузера, на экране мобильного устройства или работать "за кадром" через JavaScript, API, базы данных.

Запросы от клиента к серверу проходят по протоколу **HTTPS**, включают заголовки, тело и метод, и возвращают **response** с кодом состояния, данными и метаданными.

---

## 🔧 API-тестирование: структура запроса

Каждый HTTP-запрос состоит из нескольких частей:

### 1. **URL** — адрес сервиса или endpoint
```http
https://api.example.com/user/login
```

### 2. **Метод запроса** — соответствует операции CRUD:
| Метод | Назначение       | Идемпотентность |
|-------|------------------|-----------------|
| GET   | Чтение данных    | ✅              |
| POST  | Создание         | ❌              |
| PUT   | Полное обновление| ✅              |
| PATCH | Частичное обновление | ❌          |
| DELETE| Удаление         | ✅              |

### 3. **Заголовки (Headers)** — метаинформация:
```http
Content-Type: application/json
Authorization: Bearer <token>
```
В Java это может быть:
```java
Map<String, String> headers = new HashMap<>();
headers.put("Content-Type", "application/json");
```

### 4. **Тело запроса (Body)** — данные, особенно для POST/PUT:
```json
{
  "username": "admin",
  "password": "1234"
}
```

---

## 🧪 Инструменты для API-тестирования

### DevTools (в браузере)
- Вкладка **Network** → фильтры `fetch/xhr`
- Отображение запросов, заголовков, тела, ответа
- Удобно для анализа поведения UI и API

### Postman
- Отправка запросов вручную
- Поддержка коллекций, переменных, авторизации
- Шаблоны именования:
  - `POST /auth/login` → `LoginUser`
  - `GET /products/{id}` → `GetProductById`
- Удобен для исследования и ручного тестирования

### cURL
- Командная строка для отправки запросов:
```bash
curl -X POST https://api.example.com/login \
-H "Content-Type: application/json" \
-d '{"username":"admin","password":"1234"}'
```

---

## 📥 Ответ от сервера (Response)

Состоит из:
- **Код состояния**: `200 OK`, `401 Unauthorized`, `404 Not Found`, `500 Internal Server Error`
- **Заголовки**: `Content-Type`, `Set-Cookie`, `Cache-Control`
- **Тело**: JSON, XML, HTML, текст

Пример:
```json
{
  "token": "abc123",
  "user": {
    "id": 1,
    "name": "Admin"
  }
}
```

---

## 🧪 RestAssured: автоматизация API-тестов на Java

Библиотека для написания API-тестов в стиле fluent:

```java
given()
    .header("Content-Type", "application/json")
    .body("{\"username\":\"admin\",\"password\":\"1234\"}")
.when()
    .post("/login")
.then()
    .statusCode(200)
    .body("token", notNullValue());
```

Поддерживает:
- авторизацию;
- сериализацию/десериализацию;
- валидацию JSON;
- интеграцию с JUnit/TestNG.

---

## 📝 Практика: форма логина в Postman

**Домашнее задание**:
1. Открыть Postman.
2. Создать `POST`-запрос к `/login` endpoint.
3. Ввести `raw` JSON в теле:
```json
{
  "username": "admin",
  "password": "1234"
}
```
4. Отправить запрос и изучить:
  - статус ответа;
  - тело ответа;
  - заголовки;
  - поведение при неправильных данных.

Это поможет понять, как работает серверная логика и какие данные передаются между клиентом и сервером.

---

## 🔗 Источники:
- [Postman Learning Center](https://learning.postman.com/)
- [RestAssured Documentation](https://rest-assured.io/)
- [MDN HTTP Overview](https://developer.mozilla.org/en-US/docs/Web/HTTP)
- [DevTools Network Panel](https://developer.chrome.com/docs/devtools/network/)
- [cURL Manual](https://curl.se/docs/manual.html)

---
[**← Назад к оглавлению**](../../README.md)