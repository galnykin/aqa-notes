# Практика разработки и интеграции тестового автоматизированного фреймворка в CI/CD

Автоматизированное тестирование становится особенно мощным, когда интегрируется в общий цикл разработки. Это позволяет запускать тесты при каждом изменении кода, отслеживать стабильность сборок и получать отчёты в реальном времени. Ниже рассмотрены ключевые понятия CI/CD, настройка серверной среды, работа с Jenkins и генерация отчётов.

---

## 🧭 Определения и понятия: CI и CD

- **CI (Continuous Integration)** — непрерывная интеграция. Каждый коммит в репозиторий автоматически запускает сборку и тесты.
- **CD (Continuous Delivery / Deployment)** — непрерывная доставка или развёртывание. После успешной сборки и тестирования приложение автоматически доставляется на сервер или в продакшн.

Цель: быстрое обнаружение ошибок, стабильные релизы, минимизация ручной работы.

---

## ⚙️ Настройка серверной среды

Для запуска тестов и сборки проекта на сервере необходимы:

- **JDK** — Java Development Kit, версия должна соответствовать проекту.
- **Git** — для клонирования репозитория.
- **Maven** — для сборки проекта и управления зависимостями.
- **Allure** — для генерации отчётов о тестировании.
- **Jenkins** — сервер автоматизации, управляющий процессом CI/CD.

Дополнительно: настройка переменных окружения, прав доступа, путей к инструментам.

---

## 🧰 Jenkins: автоматизация CI/CD

**Jenkins** — это open-source сервер, который позволяет автоматизировать сборку, тестирование и доставку проекта.

### Настройка Jenkins:
- Установка через `.war`, Docker или пакетный менеджер.
- Подключение JDK, Maven, Git в разделе **Global Tool Configuration**.
- Установка плагинов:
  - `Git Plugin`
  - `Maven Integration`
  - `JUnit`
  - `Allure Report`
  - `Pipeline`

### Создание задачи (Job):
- Тип: Freestyle Project или Pipeline.
- Источник: Git URL, ветка.
- Шаги:
  - Сборка Maven: `clean test`
  - Генерация отчёта: `allure:report`
  - Публикация результатов: `Publish JUnit test result report`

### Триггеры:
- **SCM Polling** — Jenkins проверяет изменения в Git.
- **Webhook** — GitHub/GitLab уведомляет Jenkins о коммите.
- **Schedule** — запуск по расписанию (`H/15 * * * *` — каждые 15 минут).

---

## 🔗 Интеграция с фреймворком

Фреймворк должен:
- Поддерживать запуск через Maven (`mvn test`)
- Генерировать отчёты (`target/allure-results`)
- Иметь конфигурацию в `pom.xml` для Allure
- Использовать аннотации `@Step`, `@Attachment` для отчётности

Пример `pom.xml` для Allure:
```xml
<plugin>
  <groupId>io.qameta.allure</groupId>
  <artifactId>allure-maven</artifactId>
  <version>2.11.2</version>
</plugin>
```

---

## 📊 Отчёты: Allure Reporting

**Allure** — инструмент для визуализации результатов тестов:
- Отображает шаги, вложения, скриншоты, параметры.
- Интегрируется с JUnit, TestNG, RestAssured.
- Генерирует HTML-отчёт:
  ```bash
  allure generate target/allure-results -o target/allure-report
  allure open target/allure-report
  ```

В Jenkins отчёты отображаются в виде вкладки после выполнения задачи.

---

## 🔗 Источники:
- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [Allure Framework](https://docs.qameta.io/allure/)
- [Maven Getting Started](https://maven.apache.org/guides/getting-started/)
- [Git SCM Integration](https://plugins.jenkins.io/git/)
- [CI/CD Concepts](https://www.redhat.com/en/topics/devops/what-is-ci-cd)

---
[**← Назад к оглавлению**](../../../README.md)
