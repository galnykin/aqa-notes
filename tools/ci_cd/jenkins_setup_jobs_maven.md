# Jenkins: установка, структура задач, триггеры и интеграция с Maven

Jenkins — это open-source сервер автоматизации, широко используемый для CI/CD. Он позволяет запускать сборки, тесты, деплой и другие задачи по расписанию, при коммитах или вручную. Ниже — краткий обзор установки Jenkins, структуры задач, триггеров, Maven-интеграции и поведения по умолчанию.

---

## 📥 Установка Jenkins

Jenkins можно скачать с официального сайта:

- [🔗 jenkins.io/download](https://www.jenkins.io/download/)

Рекомендуется использовать версию **Long-Term Support (LTS)** — стабильную и регулярно обновляемую.

---

## 🧱 Структура: item, job, task

- **Item** — базовая единица в Jenkins, может быть job, pipeline, folder и т.д.
- **Job** — задача, которая выполняет действия: сборка, тестирование, деплой.
- **Task** — конкретный шаг внутри job, например, запуск Maven или скрипта.

---

## 🧬 Git: клонирование и триггеры

### Клонирование репозитория:
```bash
git clone https://github.com/your-org/project.git
```

В Jenkins job можно настроить Git-репозиторий как источник:
- URL
- ветка (`master`, `develop`, `feature/*`)
- credentials (если приватный)

### Триггеры: cron-расписание

Jenkins использует cron-формат:
```
MINUTE HOUR DAY MONTH WEEKDAY
```

Примеры:
- `H/15 * * * *` — каждые 15 минут
- `0 9 * * 1-5` — в 9:00 по будням
- `@daily` — каждый день

---

## ⚙️ Maven: запуск тестов

Jenkins поддерживает Maven через шаг:
```
Build step 'Invoke top-level Maven targets'
```

Пример команды:
```bash
clean test
```

Это:
- очищает `target` папку;
- компилирует проект;
- запускает тесты.

---

## 🔧 Настройки Jenkins

- **Settings** — глобальные параметры Jenkins.
- **Plugins** — расширения: Git, Maven, Allure, Slack, Email.
- **Global Tool Configuration** — настройка JDK, Maven, Git, Gradle и др.

---

## 🧪 Поведение по умолчанию

Если в job настроен запуск UI-тестов (например, Selenium), Jenkins может открыть браузер на сервере или агенте. Это требует:

- установленного браузера (Chrome, Firefox)
- драйвера (chromedriver, geckodriver)
- настройки headless-режима, если нет GUI

---

## 🔗 Источники:

- [📥 Скачать Jenkins (официальный сайт)](https://www.jenkins.io/download/)
- [📚 Документация по cron-триггерам](https://www.jenkins.io/doc/book/pipeline/syntax/#triggers)
- [🧰 Maven Integration Plugin](https://plugins.jenkins.io/maven-plugin/)
- [🔧 Настройка Global Tool Configuration](https://www.jenkins.io/doc/book/managing/tools/)

---
[**← Назад к оглавлению**](../../README.md)
