# Работа с ветками в Git: создание, переключение, слияние и выборочные коммиты

Работа с ветками — основа эффективного использования Git. Ветки позволяют изолировать изменения, вести параллельную разработку, тестировать фичи и управлять релизами. Эта статья охватывает ключевые операции с ветками: от создания до cherry-pick и удаления.

---

## 📌 Создание и переключение между ветками

### Создать новую ветку:
```bash
git branch feature/login
```

### Переключиться на неё:
```bash
git checkout feature/login
```

Или создать и сразу перейти:
```bash
git checkout -b feature/login
```

---

## 🔄 Работа с неправильной веткой: перенос изменений

Если ты начал работу не в той ветке:

1. Сохрани изменения:
   ```bash
   git stash
   ```

2. Переключись:
   ```bash
   git checkout correct-branch
   ```

3. Восстанови изменения:
   ```bash
   git stash pop
   ```

Или просто перенеси коммит через `cherry-pick`.

---

## 🍒 Cherry-pick: выборочный перенос коммитов

Позволяет взять конкретный коммит из другой ветки:

```bash
git cherry-pick <commit-hash>
```

Применимо, если нужно перенести фикс или фичу без полного слияния.

---

## 🔀 Слияние веток: merge vs rebase

### Merge:
```bash
git checkout master
git merge feature/login
```
- Сохраняет историю ветки
- Создаёт merge-коммит

### Rebase:
```bash
git checkout feature/login
git rebase master
```
- Переписывает историю
- Делает её линейной

**Когда использовать:**
- `merge` — для публичных веток
- `rebase` — для локальных фич, перед PR

---

## 🧹 Удаление ветки

### Локально:
```bash
git branch -d feature/login
```

### На сервере:
```bash
git push origin --delete feature/login
```
Вот дополнения к двум статьям — с примерами, как выполнять соответствующие действия в **IntelliJ IDEA**.

---

### 📌 Создание и переключение ветки в IntelliJ IDEA

1. Открой вкладку **Git** (внизу IDE).
2. Нажми на текущую ветку → **New Branch**.
3. Введи имя ветки (например, `feature/login`) → **Create**.
4. Для переключения — снова нажми на ветку → выбери нужную → **Checkout**.

### 🍒 Cherry-pick коммита через интерфейс

1. Открой **Git → Log**.
2. Найди нужный коммит в другой ветке.
3. Правый клик → **Cherry-pick**.
4. IntelliJ применит коммит в текущую ветку, предложит разрешить конфликты, если они есть.

### 🧹 Удаление ветки

- **Локально**: вкладка Git → ветка → **Delete**.
- **Удалённо**: через **Git → Manage Remotes** или вручную через терминал.

---

## 🔗 Источники:

- [Git Branching — Git SCM](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell)
- [Git Cherry-pick — Atlassian](https://www.atlassian.com/git/tutorials/cherry-pick)
- [Git Merge vs Rebase — GitHub Docs](https://docs.github.com/en/pull-requests/committing-changes-to-your-project/working-with-branches/merging-a-pull-request)

---
[**← Назад к оглавлению**](README.md)