# Коммиты и история в Git: создание, просмотр, откат и восстановление

Коммиты — это фундаментальные единицы истории в Git. Они фиксируют изменения, позволяют отслеживать прогресс, откатывать ошибки и восстанавливать утерянные данные. Эта статья охватывает ключевые команды и подходы к работе с коммитами: от создания до восстановления через reflog.

---

## 📌 Создание коммита

### Добавить изменения:
```bash
git add .
```

### Зафиксировать:
```bash
git commit -m "Добавлен тест авторизации"
```

Можно использовать `-a` для автоматического добавления изменённых файлов:
```bash
git commit -am "Обновлён тест логина"
```

---

## 🔍 Просмотр истории

### Стандартный лог:
```bash
git log
```

### Краткий формат:
```bash
git log --oneline
```

### С графом ветвлений:
```bash
git log --oneline --graph --all
```

---

## 🔄 Revert vs Reset

| Команда       | Назначение                          | Безопасность |
|---------------|--------------------------------------|--------------|
| `git revert`  | Создаёт обратный коммит              | ✅ безопасно |
| `git reset`   | Перемещает указатель ветки           | ⚠️ опасно    |

### Revert:
```bash
git revert <commit-hash>
```

### Reset:
```bash
git reset --soft <commit-hash>`   # сохраняет изменения
git reset --mixed <commit-hash>`  # сбрасывает индекс
git reset --hard <commit-hash>`   # удаляет всё

---

## 🧹 Drop коммита через интерактивный rebase

```bash
git rebase -i HEAD~3
```

В открывшемся редакторе можно заменить `pick` на `drop` или удалить строку.

---

## 🧭 Reflog: восстановление утерянных коммитов

Если коммит был удалён или ветка сброшена:

```bash
git reflog
```

Показывает все перемещения HEAD. Можно восстановить:
```bash
git checkout <commit-hash>
```

---

### 📋 Создание коммита в IntelliJ IDEA

1. Нажми **Commit** (в правом верхнем углу или `Ctrl+K`).
2. Выбери файлы → напиши сообщение → **Commit** или **Commit and Push**.

### 🔍 Просмотр истории

- Открой **Git → Log**.
- Можно фильтровать по ветке, автору, дате.
- Визуальный граф ветвлений показывает структуру проекта.

### 🔄 Revert и Reset

- **Revert**: правый клик по коммиту → **Revert Commit**.
- **Reset**: правый клик → **Reset Current Branch to Here** → выбери `soft`, `mixed`, `hard`.

### 🧭 Reflog и восстановление

- Открой **Git → Log** → вкладка **Reflog** (если включена).
- Найди нужное состояние HEAD → правый клик → **Checkout Revision**.

---

## 🔗 Источники:

- [Git Commit — Git SCM](https://git-scm.com/docs/git-commit)
- [Git Revert vs Reset — Atlassian](https://www.atlassian.com/git/tutorials/resetting-checking-out-and-reverting)
- [Git Reflog — GitHub Docs](https://docs.github.com/en/get-started/using-git/viewing-the-reflog)

---
[**← Назад к оглавлению**](README.md)