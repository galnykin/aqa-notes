
# Основы SQL и работы с базами данных для автоматизатора

Работа с базами данных — важный навык для инженера по автоматизации. Он позволяет проверять корректность данных, подготавливать тестовые сценарии, очищать окружение и анализировать поведение системы на уровне хранения. SQL — универсальный язык для взаимодействия с реляционными СУБД, таким как PostgreSQL, MySQL, Oracle, H2 и др.

---

## 📘 Что такое SQL

**SQL (Structured Query Language)** — язык структурированных запросов, используемый для:
- извлечения данных;
- модификации таблиц;
- обновления и удаления записей;
- управления схемой базы данных.

---

## 🔍 Основные команды SQL

### 1. SELECT — выборка данных
```sql
SELECT * FROM users WHERE role = 'admin';
```

### 2. INSERT — добавление данных
```sql
INSERT INTO products (name, price) VALUES ('Pizza', 9.99);
```

### 3. UPDATE — обновление данных
```sql
UPDATE users SET password = 'newpass' WHERE id = 1;
```

### 4. DELETE — удаление данных
```sql
DELETE FROM orders WHERE status = 'cancelled';
```

---

## 🧰 Практика для автоматизатора

- **Проверка данных после UI-действий** — например, после регистрации пользователя.
- **Подготовка тестовых данных** — создание записей перед запуском тестов.
- **Очистка окружения** — удаление временных данных после тестов.
- **Сравнение ожидаемых и фактических значений** — через SQL-запросы.

---

## 🔗 Инструменты

- **DBeaver** — универсальный клиент для работы с БД.
- **pgAdmin** — для PostgreSQL.
- **MySQL Workbench** — для MySQL.
- **JDBC** — подключение к БД из Java.
- **Testcontainers** — запуск временных БД в Docker для автотестов.

---

## 🔗 Источники:

- [SQL Tutorial — W3Schools](https://www.w3schools.com/sql/)
- [DBeaver Universal DB Tool](https://dbeaver.io/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [JDBC Guide — Oracle](https://docs.oracle.com/javase/tutorial/jdbc/)

---
[**← Назад к оглавлению**](README.md)
