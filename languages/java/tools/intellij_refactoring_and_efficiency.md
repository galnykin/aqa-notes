# Эффективная работа в IntelliJ IDEA: вынос логики, горячие клавиши и лучшие практики

IntelliJ IDEA — мощная среда разработки, которая позволяет не только писать код, но и **рефакторить его быстро и безопасно**. Один из частых приёмов — **вынос логики в отдельный метод или утилитный класс**, особенно когда код начинает повторяться или становится слишком громоздким. В этой статье — как это делать быстро с помощью горячих клавиш, и какие ещё приёмы ускоряют работу в IDE.

---

## ✂️ Вынос логики в отдельный метод

### Сценарий:
У вас есть фрагмент кода, который повторяется или перегружает текущий метод. Его можно вынести в отдельный метод.

### Как сделать:

1. **Выделите фрагмент кода**
2. Нажмите `Ctrl + Alt + M` (Windows/Linux) или `⌘ + ⌥ + M` (Mac)
3. Введите имя нового метода → нажмите Enter

### Пример:
```java
// Было:
if (user.isActive() && user.getRole().equals("ADMIN")) {
    sendNotification(user);
}

// Стало:
if (isAdmin(user)) {
    sendNotification(user);
}

private boolean isAdmin(User user) {
    return user.isActive() && user.getRole().equals("ADMIN");
}
```

---

## 📦 Вынос логики в утилитный класс

### Сценарий:
Метод используется в нескольких классах — лучше вынести его в `Utils` или `Helper`.

### Как сделать:

1. Создайте новый класс: `Ctrl + Alt + Insert` → **Class**
2. Вставьте метод и сделайте его `static`
3. Используйте `Ctrl + Alt + Shift + T` → **Move** для переноса метода из текущего класса

---

## 🔥 Другие полезные горячие клавиши и приёмы

### 🔄 `Ctrl + Alt + V` — Вынести выражение в переменную
```java
int total = calculatePrice() + calculateTax();  
// Выделите выражение → `Ctrl + Alt + V` → получим:
int price = calculatePrice();
int tax = calculateTax();
int total = price + tax;
```

### 🧩 `Ctrl + Alt + F` — Вынести в поле класса
Полезно при работе с конструктором или DI.

### 📥 `Ctrl + Alt + P` — Вынести в параметр метода
Упрощает тестируемость и переиспользование.

### 🧼 `Ctrl + Alt + L` — Форматировать код
Приводит код к стандартному стилю.

### 🔍 `Ctrl + Shift + F` — Поиск по всему проекту
Быстро найти использование метода, переменной, текста.

### 🧠 `Ctrl + Shift + Alt + T` — Рефакторинг меню
Открывает все доступные рефакторинги: Extract, Move, Rename, Inline и др.

---

## 🧰 Советы по организации кода

- **Выносите повторяющуюся логику** в утилитные классы (`StringUtils`, `DateUtils`, `ValidationUtils`)
- **Разделяйте ответственность** — один метод = одна задача
- **Используйте шаблоны Live Templates** (`sout`, `psvm`, `fori`) для ускорения ввода
- **Настройте Code Style** под командные стандарты
- **Используйте TODO-комментарии** (`// TODO: validate input`) — IntelliJ их отслеживает

---

## 🔗 Источники:

- [IntelliJ IDEA Keymap Reference](https://resources.jetbrains.com/storage/products/intellij-idea/docs/IntelliJIDEA_ReferenceCard.pdf)
- [Refactoring in IntelliJ IDEA](https://www.jetbrains.com/help/idea/refactoring-source-code.html)
- [Live Templates Guide](https://www.jetbrains.com/help/idea/using-live-templates.html)

---
[**← Назад к оглавлению**](../README.md)
