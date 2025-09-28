# Практика разработки и интеграции тестового автоматизированного фреймворка в CI/CD

Автоматизированное тестирование становится особенно мощным, когда интегрируется в общий цикл разработки. Это позволяет запускать тесты при каждом изменении кода, отслеживать стабильность сборок и получать отчёты в реальном времени. Ниже рассмотрены ключевые понятия CI/CD, настройка серверной среды, работа с Jenkins и GitHub Actions, а также генерация отчётов.

-----

## 🧭 Определения и понятия: CI/CD

- **CI (Continuous Integration / Непрерывная интеграция)** — это практика, при которой разработчики регулярно сливают свои изменения в центральный репозиторий. После каждого слияния происходит **автоматическая сборка** проекта и **запуск тестов**. Главная цель — раннее обнаружение ошибок интеграции.

- **CD (Continuous Delivery or Deployment / Непрерывная доставка или развёртывание)** — это следующий шаг после CI.

  - **Continuous Delivery**: После успешного прохождения всех тестов сборка автоматически доставляется в окружение (например, `staging`), где готова к ручному развёртыванию в `production`.
  - **Continuous Deployment**: Полностью автоматизированный процесс, где успешная сборка **автоматически развёртывается** в `production` без вмешательства человека.

**Ключевая философия**: "Fail fast, fail often" (Падай быстро, падай часто). Чем раньше будет найдена ошибка, тем дешевле её исправить.

-----

## ⚙️ Настройка серверной среды

Для запуска тестов на выделенном сервере (CI agent) требуется определённое окружение.

- **JDK**: Java Development Kit. Версия должна быть строго той же, что и в проекте (`Java 17+`).
- **Git**: Для клонирования исходного кода из репозитория.
- **Maven**: Для управления зависимостями, компиляции и запуска тестов.
- **Allure Commandline**: Инструмент для генерации HTML-отчёта из `allure-results`.
- **CI-сервер**: Программное обеспечение, которое оркестрирует весь процесс. Наиболее популярные — **Jenkins** и **GitHub Actions**.

### Современный подход: Docker

Чтобы избежать ручной установки и "загрязнения" CI-сервера, всё чаще используют **Docker**. Среда для сборки описывается в `Dockerfile`, что гарантирует её идентичность на машине разработчика и в CI. **Testcontainers** могут использовать Docker для запуска браузеров или баз данных прямо из тестов, обеспечивая полную изоляцию.

-----

## 🧰 CI/CD Инструменты: Jenkins vs GitHub Actions

### Jenkins: Pipeline as Code

**Jenkins** — это классический, мощный и гибкий open-source сервер автоматизации. Современный подход к работе с Jenkins — **Pipeline as Code**, где весь процесс сборки описывается в файле `Jenkinsfile`, который хранится в репозитории вместе с кодом.

**Пример `Jenkinsfile` (Declarative Pipeline):**

```groovy
pipeline {
    agent any // Запустить на любом доступном агенте

    tools {
        maven 'Maven-3.8.6' // Использовать Maven, настроенный в Jenkins
        jdk 'JDK-17'        // Использовать JDK, настроенный в Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                // Клонируем репозиторий
                git branch: 'main', url: 'https://github.com/your-repo/test-automated-framework.git'
            }
        }
        
        stage('Run Tests') {
            steps {
                // Запускаем тесты с помощью Maven
                // sh - команда для shell
                sh 'mvn clean test'
            }
        }
    }
    
    post {
        // Блок, который выполняется всегда после всех стадий
        always {
            // Архивируем результаты для Allure-плагина
            allure includeProperties: false, 
                   reportBuildPolicy: 'ALWAYS', 
                   results: [[path: 'target/allure-results']]
        }
    }
}
```

### GitHub Actions: Нативная интеграция

**GitHub Actions** — это CI/CD инструмент, встроенный напрямую в GitHub. Он управляется с помощью `.yml` файлов, хранящихся в директории `.github/workflows`. Это более современное и простое в настройке решение для проектов, размещённых на GitHub.

**Пример `.github/workflows/main.yml`:**

```yaml
name: Run Automated Tests

on:
  push:
    branches: [ main ] # Триггер: пуш в ветку main
  pull_request:
    branches: [ main ] # Триггер: создание Pull Request в main

jobs:
  build:
    runs-on: ubuntu-latest # Запуск на виртуальной машине с Ubuntu

    steps:
    - name: Checkout repository
      uses: actions/checkout@v3 # Стандартное действие для клонирования репозитория

    - name: Set up JDK 17
      uses: actions/setup-java@v3 # Настройка Java
      with:
        java-version: '17'
        distribution: 'temurin'
        cache: 'maven' # Кэширование зависимостей Maven

    - name: Run tests with Maven
      run: mvn clean test # Запуск тестов

    - name: Generate Allure Report
      uses: simple-elf/allure-report-action@v1.7
      if: always() # Выполнить шаг, даже если тесты упали
      with:
        report_dir: target/allure-report
        results_dir: target/allure-results
        
    - name: Deploy report to GitHub Pages
      if: always()
      uses: peaceiris/actions-gh-pages@v3
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: ./target/allure-report
```

-----

## 🔗 Подготовка фреймворка для CI/CD

Чтобы тесты стабильно работали на CI-сервере, фреймворк должен быть к этому готов.

### 1\. Headless-режим для браузеров

На CI-серверах нет графического интерфейса, поэтому браузеры должны запускаться в **headless-режиме** (без отрисовки окна).

**Пример для Chrome в `WebDriverFactory`:**

```java
import org.openqa.selenium.chrome.ChromeOptions;

ChromeOptions options = new ChromeOptions();
// Аргументы для запуска в headless-режиме
options.addArguments("--headless");
options.addArguments("--disable-gpu");
options.addArguments("--window-size=1920,1080");
options.addArguments("--no-sandbox"); // Важно для запуска в Docker/Linux

WebDriver driver = new ChromeDriver(options);
```

### 2\. Управление конфигурацией

Нельзя "зашивать" в код URL, логины и пароли. Конфигурация должна передаваться извне во время запуска. Это можно сделать через **системные переменные (system properties)**.

**Пример запуска из CI:**
`mvn clean test -Denv=stage -Dbrowser=headless_chrome`

**Чтение в коде:**
`String environment = System.getProperty("env", "dev"); // "dev" - значение по умолчанию`

### 3\. Автоматические скриншоты при падении

Для анализа ошибок в **Allure-отчёте** критически важны скриншоты. Это легко автоматизировать с помощью **JUnit 5 `TestWatcher`**.

```java
import io.qameta.allure.Attachment;
import org.junit.jupiter.api.extension.ExtensionContext;
import org.junit.jupiter.api.extension.TestWatcher;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;

public class AllureTestWatcher implements TestWatcher {

    @Override
    public void testFailed(ExtensionContext context, Throwable cause) {
        WebDriver driver = WebDriverFactory.getDriver(); // Получаем драйвер из нашей фабрики
        if (driver instanceof TakesScreenshot) {
            saveScreenshot(((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES));
        }
    }

    @Attachment(value = "Screenshot on failure", type = "image/png")
    private byte[] saveScreenshot(byte[] screenShot) {
        return screenShot;
    }
}
```

Этот "watcher" нужно зарегистрировать в `BaseTest` с помощью аннотации `@ExtendWith(AllureTestWatcher.class)`.

-----

## 📊 Отчёты: Allure в CI/CD

В CI-контексте **Allure** — это не просто отчёт, а инструмент аналитики:

- **История сборок**: Allure-плагин в Jenkins хранит историю запусков, позволяя отслеживать динамику падений.
- **Тренд тестов**: Показывает, какие тесты стали "flaky" (нестабильными).
- **Окружение**: Автоматически прикрепляет информацию о среде запуска (версия Java, ОС, URL стенда).
- **Интеграция**: В отчёте можно настроить ссылки на задачи в Jira или на коммиты в Git.

-----

## 🔗 Источники:

- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [Jenkins Pipeline Syntax](https://www.jenkins.io/doc/book/pipeline/syntax/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Allure Framework Documentation](https://docs.qameta.io/allure/)
- [Running Selenium in Headless Mode](https://www.selenium.dev/blog/2023/headless-is-going-away/)
- [CI/CD Concepts by Red Hat](https://www.redhat.com/en/topics/devops/what-is-ci-cd)

-----

[**← Назад к оглавлению**](../README.md)