# Интеграция автоматизации с CI/CD: зачем и как это работает

Интеграция автотестов в CI/CD — ключевой шаг для обеспечения стабильности, скорости и предсказуемости разработки. В рамках буткамп-проекта AQA, где участники примеряют на себя роль продакт-менеджеров, понимание этой интеграции помогает не только запускать тесты, но и планировать релизы, управлять качеством и отслеживать регрессии.

---

## 🚀 Что такое CI/CD

- **CI (Continuous Integration)** — непрерывная интеграция: автоматическая сборка и тестирование при каждом коммите.
- **CD (Continuous Delivery/Deployment)** — непрерывная доставка: автоматическая публикация изменений на staging или production.

---

## 🧪 Зачем подключать автотесты к CI/CD

- **Автоматическая проверка при каждом изменении**
- **Раннее обнаружение багов**
- **Снижение ручной нагрузки**
- **Повышение доверия к сборке**
- **Возможность релизов "по кнопке"**

---

## 🔁 Как это работает

1. Разработчик пушит код в репозиторий
2. CI-сервер (например, Jenkins) запускает:
    - Сборку проекта
    - Запуск unit-тестов
    - Запуск UI/API автотестов
    - Генерацию отчёта
3. В случае успеха — деплой на тестовый сервер
4. В случае падения — уведомление в Slack, почту, Jira

---

## 🧰 Инструменты

| Задача                  | Инструмент               |
|-------------------------|--------------------------|
| CI/CD сервер            | Jenkins, GitHub Actions, GitLab CI |
| Сборка проекта          | Maven, Gradle            |
| Запуск тестов           | JUnit, TestNG, Selenide  |
| Отчёты                  | Allure, ReportPortal     |
| Уведомления             | Slack, Email, Telegram   |

---

## 🧠 Рекомендации для AQA-проекта

- **Разделяйте типы тестов**: smoke → regression → full suite
- **Запускайте smoke-тесты при каждом коммите**
- **Регрессию — по расписанию или перед релизом**
- **Храните отчёты в артефактах сборки**
- **Добавьте флаг `--tags smoke` для выборочного запуска**

---

## 📋 Пример Jenkins pipeline (Groovy)

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test -Dgroups=smoke'
            }
        }
        stage('Report') {
            steps {
                allure includeProperties: false, jdk: '', results: [[path: 'target/allure-results']]
            }
        }
    }
}
```

---

## 🔗 Источники:

- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [Allure Framework](https://docs.qameta.io/allure/)
- [GitHub Actions Guide](https://docs.github.com/en/actions)
- [CI/CD for Test Automation — TestProject](https://blog.testproject.io/2020/03/09/ci-cd-for-test-automation/)

---
[**← Назад к оглавлению**](README.md)
