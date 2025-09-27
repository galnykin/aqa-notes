
# Тестирование клиент-серверного взаимодействия и API: инструменты, структура, практика

Клиент-серверное взаимодействие — основа работы современных приложений. Клиент (браузер, мобильное приложение) отправляет запросы, сервер обрабатывает их и возвращает ответы. API-тестирование позволяет проверить корректность этих обменов, структуру данных, обработку ошибок и бизнес-логику без участия UI.

---

## 🌐 Клиент и сервер: как устроено взаимодействие

* **Клиент** — браузер, мобильное приложение, десктопное ПО
* **Сервер** — обрабатывает запросы и возвращает ответы (JSON, XML, HTML)
* **Сеть** — транспорт данных по протоколу HTTP/HTTPS

Запросы от клиента к серверу проходят по протоколу **HTTPS**, включают заголовки, тело и метод, и возвращают **response** с кодом состояния, данными и метаданными.

---

## 🔧 API-тестирование: структура запроса

Каждый HTTP-запрос состоит из нескольких частей:

1. **URL (endpoint)** — адрес ресурса
2. **Метод (HTTP method)** — `GET`, `POST`, `PUT`, `PATCH`, `DELETE`
3. **Заголовки (headers)** — метаинформация о запросе
4. **Тело (body)** — данные, чаще всего в формате JSON

Пример запроса авторизации:

```http
POST https://api.example.com/login
Content-Type: application/json

{
  "username": "admin",
  "password": "1234"
}
```

---

## 🔀 Виды API

* **REST (Representational State Transfer)**
  Наиболее распространённый стиль, обмен в формате JSON, простота интеграции

* **SOAP (Simple Object Access Protocol)**
  Использует XML, строгие контракты, часто в банковских и legacy-системах

* **GraphQL**
  Позволяет клиенту выбирать, какие именно данные нужны, вместо фиксированных эндпоинтов

* **gRPC**
  Основан на Protocol Buffers, очень быстрый, часто применяется в микросервисах

---

## 🔐 Авторизация и безопасность API

* **Basic Auth** — логин и пароль в заголовке (устаревший и небезопасный вариант)
* **Bearer Token / JWT (JSON Web Token)** — токен в заголовке Authorization
* **OAuth 2.0** — используется в Google, Facebook, GitHub API
* **API Keys** — простой ключ доступа

Пример заголовка:

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

## 🧪 Инструменты для API-тестирования

### 🔹 DevTools (браузер)

* Просмотр запросов и ответов в Network-панели
* Удобно для анализа UI → API

### 🔹 Postman

* Отправка запросов вручную
* Коллекции, переменные окружений
* Автотесты на JavaScript
* Экспорт в **Newman** для запуска в CI/CD

### 🔹 cURL

* CLI-инструмент для тестирования API

```bash
curl -X POST https://api.example.com/login \
-H "Content-Type: application/json" \
-d '{"username":"admin","password":"1234"}'
```

### 🔹 Swagger / OpenAPI

* Автодокументация и тестирование API прямо в браузере

### 🔹 Insomnia

* Альтернатива Postman с простым интерфейсом и плагинами

---

## 📥 Ответ от сервера (Response)

Состоит из:

* **Код состояния**: `200 OK`, `401 Unauthorized`, `404 Not Found`, `500 Internal Server Error`
* **Заголовки**: `Content-Type`, `Set-Cookie`, `Cache-Control`
* **Тело**: JSON, XML, HTML, текст

Пример JSON-ответа:

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

Пример теста на авторизацию:

```java
import static io.restassured.RestAssured.*;
import static org.hamcrest.Matchers.*;

@Test
public void loginTest() {
    given()
        .header("Content-Type", "application/json")
        .body("{\"username\":\"admin\",\"password\":\"1234\"}")
    .when()
        .post("/login")
    .then()
        .statusCode(200)
        .body("token", notNullValue());
}
```

RestAssured поддерживает:

* JSONPath и XPath проверки
* сериализацию POJO в JSON
* интеграцию с JUnit/TestNG
* работу с авторизацией

---

## ❌ Негативное тестирование API

Важно проверять не только happy path, но и ошибки:

* **Неверные данные** → `400 Bad Request`
* **Отсутствие авторизации** → `401 Unauthorized`
* **Нет доступа к ресурсу** → `403 Forbidden`
* **Несуществующий ресурс** → `404 Not Found`
* **Падение сервера** → `500 Internal Server Error`

---

## ⚙️ CI/CD и отчётность

Автотесты API можно запускать в пайплайнах CI/CD (Jenkins, GitLab CI).
Для наглядных отчётов часто используют:

* **Allure Report** — красивые графические отчёты
* **ExtentReports** — HTML отчёты с логами

---

## 📝 Практика: форма логина в Postman

1. Открыть Postman
2. Создать `POST`-запрос к `/login` endpoint
3. Передать JSON в `Body`
4. Проверить статус, заголовки, тело
5. Попробовать негативные сценарии (пустой пароль, неверный логин)

---

## 🔗 Источники

* [Postman Learning Center](https://learning.postman.com/)
* [RestAssured Documentation](https://rest-assured.io/)
* [Swagger Documentation](https://swagger.io/docs/)
* [MDN HTTP Overview](https://developer.mozilla.org/en-US/docs/Web/HTTP)
* [OAuth 2.0 Specification](https://oauth.net/2/)

---

[**← Назад к оглавлению**](README.md)
