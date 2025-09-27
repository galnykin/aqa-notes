# Практика работы с jsoup и log4j2: парсинг HTML и логирование в Java

Автоматизатору важно уметь работать с HTML-документами и вести логирование событий. Ниже представлены две ключевые Java-библиотеки: **jsoup** — для парсинга HTML, и **log4j2** — для логирования.

---

## 🧾 1. jsoup: парсинг HTML из строки

**jsoup** — это Java-библиотека для работы с HTML. Она позволяет:

- превратить строку HTML в объект `Document`;
- извлекать элементы по тегам, классам, id;
- модифицировать HTML;
- работать с реальными веб-страницами.

### Пример: парсинг HTML из строки
```java
String html = "<html><body><p>Hello, world!</p></body></html>";
Document doc = Jsoup.parse(html);
String text = doc.select("p").text(); // "Hello, world!"
```

### Возможности:
- `doc.title()` — получить заголовок страницы
- `doc.select("div.class")` — выбрать элементы по CSS-селектору
- `element.attr("href")` — получить значение атрибута
- `element.text()` — получить текст внутри тега

### Полезные материалы:
- [Cookbook: Parse from String](https://jsoup.org/cookbook/input/parse-document-from-string)
- [Baeldung: Java with jsoup](https://www.baeldung.com/java-with-jsoup)
- [Официальный сайт jsoup](https://jsoup.org/)

---

## 📋 2. log4j2: логирование событий

**log4j2** — это мощная библиотека для логирования в Java. Она позволяет:

- записывать сообщения в консоль, файл, HTML, JSON;
- управлять уровнями логов (`INFO`, `DEBUG`, `ERROR`);
- использовать шаблоны форматирования;
- отделить логи от бизнес-логики.

### Пример: базовое логирование
```java
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

public class LoginTest {
    private static final Logger logger = LogManager.getLogger(LoginTest.class);

    public void login() {
        logger.info("Начало теста логина");
        logger.debug("Вводим имя пользователя");
        logger.error("Ошибка авторизации");
    }
}
```

### Уровни логирования:
- `TRACE` — подробная отладка
- `DEBUG` — технические детали
- `INFO` — бизнес-события
- `WARN` — потенциальные проблемы
- `ERROR` — ошибки
- `FATAL` — критические сбои

### Конфигурация:
- `log4j2.xml` или `log4j2.properties`
- Настройка appenders (куда писать) и layouts (как писать)

### Полезные материалы:
- [Урванов: логирование с log4j2](https://urvanov.ru/2019/07/04/%D0%BB%D0%BE%D0%B3%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D1%81-log4j-2-%D0%B2-java/)
- [Официальная документация log4j2](https://logging.apache.org/log4j/2.12.x/)

---

## 🔗 Источники:

- [Парсинг HTML из строки (jsoup cookbook)](https://jsoup.org/cookbook/input/parse-document-from-string)
- [Работа с jsoup на Java (Baeldung)](https://www.baeldung.com/java-with-jsoup)
- [Официальный сайт jsoup](https://jsoup.org/)
- [Логирование с log4j2 (Урванов)](https://urvanov.ru/2019/07/04/логирование-с-log4j-2-в-java/)
- [Документация log4j2](https://logging.apache.org/log4j/2.12.x/)
---
[**← Назад к оглавлению**](../../../README.md)

