# Написание тестов в Postman: основы, примеры, лучшие практики

Postman — это не только инструмент для отправки HTTP-запросов, но и мощная среда для написания автоматических проверок (тестов) прямо внутри каждого запроса. Эти тесты позволяют валидировать ответы, проверять структуру данных, коды состояния, заголовки и многое другое — без необходимости писать отдельный код на Java или Python.

---

## 🧪 Где писать тесты в Postman

В каждом запросе Postman есть вкладка **Tests**, где можно писать JavaScript-код, который будет выполняться после получения ответа от сервера.

---

## ✅ Примеры базовых тестов

### Проверка кода состояния:
```javascript
pm.test("Статус 200", function () {
    pm.response.to.have.status(200);
});
```

### Проверка наличия поля в JSON:
```javascript
pm.test("Ответ содержит токен", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.token).to.not.be.undefined;
});
```

### Проверка значения поля:
```javascript
pm.test("Имя пользователя — admin", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.user.name).to.eql("admin");
});
```

### Проверка заголовка ответа:
```javascript
pm.test("Content-Type — JSON", function () {
    pm.response.to.have.header("Content-Type");
    pm.expect(pm.response.headers.get("Content-Type")).to.include("application/json");
});
```

---

## 📦 Работа с JSON-ответом

Postman автоматически парсит JSON-ответы:
```javascript
var jsonData = pm.response.json();
pm.expect(jsonData.products.length).to.be.above(0);
```

Можно проверять вложенные поля, массивы, длину, типы данных.

---

## 🔁 Проверка нескольких условий

```javascript
pm.test("Проверка нескольких параметров", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.status).to.eql("success");
    pm.expect(jsonData.data).to.be.an("object");
    pm.expect(jsonData.data.id).to.be.a("number");
});
```

---

## 🧩 Использование переменных

Postman позволяет сохранять значения из ответа в переменные:
```javascript
var jsonData = pm.response.json();
pm.environment.set("authToken", jsonData.token);
```

Это удобно для цепочки запросов: сначала логин → сохранить токен → использовать в следующем запросе.

---

## 📊 Отчёты и интеграция

- Тесты отображаются в разделе **Test Results** после выполнения запроса.
- Можно запускать коллекции с тестами через **Collection Runner**.
- Для CI/CD используется **Newman** — CLI-инструмент для запуска Postman-коллекций.

---

## 🔗 Источники:
- [Postman Learning Center](https://learning.postman.com/docs/writing-scripts/script-references/test-examples/)
- [Postman JavaScript API](https://learning.postman.com/docs/writing-scripts/script-references/postman-sandbox-api/)
- [Newman CLI Tool](https://www.npmjs.com/package/newman)

---
[**← Назад к оглавлению**](../README.md)
