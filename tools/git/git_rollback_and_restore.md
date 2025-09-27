# Откат и восстановление изменений в Git: reset, revert, stash и Local History

В процессе разработки часто возникает необходимость отменить изменения, откатить коммиты или временно сохранить работу. Git предоставляет гибкие инструменты для этого: `reset`, `revert`, `stash`, а IntelliJ IDEA дополняет их визуальными средствами, включая Local History. Эта статья охватывает все основные способы отката и восстановления — как в консоли, так и в интерфейсе IDE.

---

## 🔄 `git reset`: перемещение указателя ветки

### Варианты:

- `--soft` — откат коммита, изменения остаются в staged
- `--mixed` — откат коммита, изменения остаются в рабочей директории
- `--hard` — откат коммита и удаление изменений

```bash
git reset --soft HEAD~1
git reset --mixed HEAD~1
git reset --hard HEAD~1
```

### В IntelliJ IDEA:

1. Открой вкладку **Git → Log**
2. Правый клик по нужному коммиту → **Reset Current Branch to Here**
3. Выбери тип: `soft`, `mixed`, `hard`

---

## 🔁 `git revert`: безопасный откат коммита

Создаёт новый коммит, отменяющий изменения предыдущего, не затрагивая историю.

```bash
git revert <commit-hash>
```

### В IntelliJ IDEA:

1. Открой **Git → Log**
2. Правый клик по коммиту → **Revert Commit**
3. Подтверди создание обратного коммита

---

## 📦 `git stash`: временное сохранение изменений

Сохраняет текущие изменения и очищает рабочую директорию.

```bash
git stash
git stash list
git stash apply
git stash drop
```

### В IntelliJ IDEA:

1. Открой **Git → Stash Changes** (через `VCS → Git → Stash Changes`)
2. Введи описание → **Stash**
3. Для восстановления: **Git → Unstash Changes**

---

## 🧭 Local History в IntelliJ IDEA

Local History — это встроенный механизм IntelliJ, который отслеживает изменения файлов даже вне Git.

### Как использовать:

1. Правый клик по файлу → **Local History → Show History**
2. Выбери нужную версию → **Revert**
3. Можно сравнивать версии и восстанавливать удалённые участки

### Преимущества:

- Работает даже без коммитов
- Полезен при случайном удалении кода
- Не зависит от Git

---

## 🧠 Когда использовать что

| Сценарий                            | Инструмент         |
|-------------------------------------|--------------------|
| Откат последнего коммита            | `reset` или `revert` |
| Временное сохранение незавершённой работы | `stash`             |
| Восстановление удалённого кода      | `Local History`     |
| Безопасный откат в публичной ветке  | `revert`            |
| Полный сброс ветки                  | `reset --hard`      |

---

## 🔗 Источники:

- [Git Reset vs Revert — Atlassian](https://www.atlassian.com/git/tutorials/resetting-checking-out-and-reverting)
- [Git Stash — Git SCM](https://git-scm.com/docs/git-stash)
- [IntelliJ IDEA: Local History](https://www.jetbrains.com/help/idea/local-history.html)
- [Git Integration in IntelliJ](https://www.jetbrains.com/help/idea/version-control-integration.html)

---
[**← Назад к оглавлению**](README.md)