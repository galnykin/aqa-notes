# Синхронизация с удалённым репозиторием: push, pull, force push и конфликты

Работа с удалённым репозиторием — ключевой аспект командной разработки. Git предоставляет команды для отправки, получения и синхронизации изменений, а IntelliJ IDEA делает эти процессы визуально понятными. Эта статья охватывает основные операции: `push`, `pull`, `fetch`, `force push`, а также работу с конфликтами.

---

## 🔼 Push: отправка изменений

### В терминале:
```bash
git push origin feature/login
```

### В IntelliJ IDEA:
1. Нажми **Git → Push** или `Ctrl+Shift+K`
2. Выбери ветку → **Push**
3. Если ветка новая, IntelliJ предложит создать её на сервере

---

## 🔽 Pull: получение и слияние изменений

### В терминале:
```bash
git pull origin master
```

### В IntelliJ IDEA:
1. Нажми **Git → Pull** или `Ctrl+T`
2. Выбери удалённую ветку → **Pull**
3. IntelliJ выполнит `fetch` + `merge` и покажет результат

---

## 🔄 Fetch: получение без слияния

```bash
git fetch origin
```

В IntelliJ:
- **Git → Fetch** — обновляет удалённые ветки, но не меняет локальные

---

## ⚠️ Force Push: принудительная отправка

Используется при переписывании истории (`rebase`, `reset`), но может затереть чужие коммиты.

```bash
git push --force
```

### В IntelliJ IDEA:
1. После `rebase` или `reset` → **Push**
2. Появится предупреждение → подтвердить **Force Push**

**Важно:** использовать только в личных ветках или по договорённости.

---

## 🔧 Конфликты при слиянии

### В терминале:
```bash
git pull
# Git сообщит о конфликте
# Разреши вручную → git add → git commit
```

### В IntelliJ IDEA:
1. При конфликте откроется окно **Merge Conflicts**
2. Выбери: **Accept Yours**, **Accept Theirs**, **Merge**
3. После разрешения → **Mark as Resolved**

---

## 🧠 Советы по синхронизации

- Всегда `pull` перед `push`, чтобы избежать конфликтов
- Используй `fetch` для предварительного анализа изменений
- Не делай `force push` в ветки, над которыми работают другие
- Разрешай конфликты осознанно, не по шаблону

---

## 🔗 Источники:

- [Git Push — Git SCM](https://git-scm.com/docs/git-push)
- [Git Pull — Atlassian](https://www.atlassian.com/git/tutorials/syncing)
- [IntelliJ IDEA Git Integration](https://www.jetbrains.com/help/idea/version-control-integration.html)
- [Git Merge Conflicts — GitHub Docs](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/addressing-merge-conflicts)

---
[**← Назад к оглавлению**](README.md)