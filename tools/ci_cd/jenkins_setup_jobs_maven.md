# Jenkins: установка, структура задач, триггеры и интеграция с Maven

Jenkins — это open-source сервер автоматизации, широко используемый для CI/CD. Он позволяет запускать сборки, тесты, деплой и другие задачи по расписанию, при коммитах или вручную. Ниже — обзор установки Jenkins, структуры задач, триггеров, Maven-интеграции и поведения по умолчанию, с акцентом на применение в AQA. Jenkins интегрируется с Git для клонирования репозиториев и автоматизации тестов (например, JUnit 5 или TestNG), что упрощает проверку UI/API. Материал связан с клиент-серверной моделью (Jenkins как сервер для запуска задач), тестированием (автоматизация тестов) и локаторами (проверка элементов в тестах).

---

## Установка Jenkins

Jenkins можно скачать с официального сайта:

- [jenkins.io/download](https://www.jenkins.io/download/)

Рекомендуется версия **Long-Term Support (LTS)** — стабильная и регулярно обновляемая.

### Шаги установки
1. **Скачивание**: Загрузите WAR-файл (для Java) или Docker-образ для контейнеризации.
2. **Запуск в Java**:
    - Требования: JDK 17+.
    - Команда:
      ```bash
      java -jar jenkins.war
      ```
    - Доступ: Откройте `http://localhost:8080` в браузере.
3. **Установка через Docker**:
   ```bash
   docker run -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
   ```
4. **Первоначальная настройка**:
    - Разблокировка: Используйте пароль из логов.
    - Установка плагинов: Выберите рекомендуемые (Git, Maven, Pipeline).
    - Создание пользователя: Укажите имя, пароль.
5. **Проверка**: Если интерфейс доступен, установка завершена.
6. **Обновление**: В Jenkins перейдите в Manage Jenkins → Manage Plugins → Updates.

В AQA Jenkins используется для запуска тестов Selenium или API-тестов (RestAssured), интегрируясь с Maven для сборки.

**Связь с другими темами**: Установка Jenkins аналогична настройке клиента в клиент-серверной модели для автоматизации задач.

---

## Структура: item, job, task

- **Item**: Базовая единица в Jenkins — любой объект, такой как job, pipeline, folder или view.
- **Job**: Задача, выполняющая последовательность шагов (build, test, deploy). Типы: Freestyle Project (простые задачи), Pipeline (скрипты на Groovy), Multibranch Pipeline (для нескольких веток Git).
- **Task**: Шаг внутри job, например, выполнение shell-команды, запуск Maven или Git pull.

### Создание job
1. В интерфейсе Jenkins: New Item → Freestyle project → OK.
2. Настройка:
    - Source Code Management: Git репозиторий.
    - Build Triggers: Cron или webhook.
    - Build Steps: Invoke top-level Maven targets (например, `clean test`).

Пример job для AQA: Запуск тестов JUnit 5/TestNG после коммита.

**Связь с другими темами**: Job в Jenkins — это "тест-кейс" для автоматизации, где шаги аналогичны проверке локаторов в тестах.

---

## Git: клонирование и триггеры

### Клонирование репозитория
- Команда в терминале:
  ```bash
  git clone https://github.com/your-org/project.git
  ```
- В Jenkins: В разделе Source Code Management укажите URL репозитория, ветку (`main` или `*` для всех), credentials (если приватный).
- Проверка: Jenkins клонирует репозиторий перед выполнением job.

### Триггеры: cron-расписание
- Jenkins использует cron-формат для расписания:
  ```
  MINUTE HOUR DAY MONTH WEEKDAY
  ```
- Примеры:
    - `H/15 * * * *` — каждые 15 минут (H — случайный час для распределения нагрузки).
    - `0 9 * * 1-5` — в 9:00 по будням (понедельник-пятница).
    - `@daily` — каждый день в полночь.
    - `H 8-16 * * *` — ежечасно между 8:00 и 16:00.
- Настройка: В job → Build Triggers → Build periodically → Укажите cron.

В AQA триггеры запускают тесты после пуша в Git или по расписанию для регрессии.

**Связь с другими темами**: Триггеры аналогичны ожиданию событий в тестах (например, WebDriverWait).

---

## Maven: запуск тестов

Jenkins поддерживает Maven через плагин Maven Integration. Maven — инструмент сборки для Java-проектов, интегрирующийся с JUnit 5 и TestNG.

### Настройка Maven в Jenkins
- Global Tool Configuration → Maven → Укажите имя установки (например, M3.9.9), укажите путь или автоматическую установку.
- В job: Build → Invoke top-level Maven targets → Выберите Maven версию → Goals: `clean test`.

### Пример Maven POM для тестов
```xml
<project>
    <groupId>com.example</groupId>
    <artifactId>aqa-project</artifactId>
    <version>1.0</version>

    <dependencies>
        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-java</artifactId>
            <version>4.25.0</version>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.11.0</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testng</groupId>
            <artifactId>testng</artifactId>
            <version>7.8.0</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.5.1</version>
            </plugin>
        </plugins>
    </build>
</project>
```

- `clean test`: Очищает target, компилирует и запускает тесты.
- В AQA: Запускает тесты Selenium с JUnit 5/TestNG.

### JUnit 5 в Maven
- Тесты запускаются по умолчанию с surefire-plugin.
- Пример теста:
  ```java
  import org.junit.jupiter.api.Test;
  import static org.junit.jupiter.api.Assertions.assertEquals;

  public class MavenTest {
      @Test
      void testAddition() {
          assertEquals(4, 2 + 2);
      }
  }
  ```

### TestNG в Maven
- Настройка в POM:
  ```xml
  <build>
      <plugins>
          <plugin>
              <groupId>org.apache.maven.plugins</groupId>
              <artifactId>maven-surefire-plugin</artifactId>
              <version>3.5.1</version>
              <configuration>
                  <suiteXmlFiles>
                      <suiteXmlFile>testng.xml</suiteXmlFile>
                  </suiteXmlFiles>
              </configuration>
          </plugin>
      </plugins>
  </build>
  ```
- testng.xml:
  ```xml
  <suite name="TestSuite">
      <test name="Test">
          <classes>
              <class name="com.example.MavenTest" />
          </classes>
      </test>
  </suite>
  ```
- Пример теста:
  ```java
  import org.testng.Assert;
  import org.testng.annotations.Test;

  public class MavenTestNG {
      @Test
      void testAddition() {
          Assert.assertEquals(4, 2 + 2);
      }
  }
  ```

**Связь с другими темами**: Maven запускает тесты для проверки локаторов в UI.

---

## Настройки Jenkins

- **Settings**: Manage Jenkins → Global Configuration — настройка путей, email, Slack.
- **Plugins**: Manage Jenkins → Manage Plugins — установите Git Plugin, Maven Integration, Allure для отчётов.
- **Global Tool Configuration**: Manage Jenkins → Global Tool Configuration — настройте JDK (Java 17+), Maven, Git, Gradle.
- **Пример настройки JDK**:
    - Name: JDK17
    - JAVA_HOME: /path/to/jdk-17
- **Пример настройки Maven**:
    - Name: M3.9.9
    - MAVEN_HOME: /path/to/maven

В AQA настройте плагины для запуска Selenium-тестов.

**Связь с другими темами**: Настройки Jenkins аналогичны настройке клиента в клиент-серверной модели.

---

## Поведение по умолчанию

Если в job настроен запуск UI-тестов (Selenium), Jenkins запускает браузер на агенте:
- Требуется браузер и драйвер (ChromeDriver).
- Headless-режим для CI/CD (без GUI):
  ```java
  ChromeOptions options = new ChromeOptions();
  options.addArguments("--headless");
  WebDriver driver = new ChromeDriver(options);
  ```
- Поведение: Тесты выполняются последовательно, если не настроен параллелизм (Pipeline с parallel).
- Ошибки: Если агент без браузера, тесты упадут с ошибкой.

**Связь с другими темами**: Поведение по умолчанию влияет на тестирование динамических элементов.

---

## Дополнительно: Testcontainers, Lombok, Page Object, Retry, CI/CD (GitHub Actions)

### Testcontainers
- Библиотека для запуска контейнеров (Docker) в тестах (например, БД MySQL для AQA).
- Maven зависимость:
  ```xml
  <dependency>
      <groupId>org.testcontainers</groupId>
      <artifactId>testcontainers</artifactId>
      <version>1.20.3</version>
      <scope>test</scope>
  </dependency>
  <dependency>
      <groupId>org.testcontainers</groupId>
      <artifactId>mysql</artifactId>
      <version>1.20.3</version>
      <scope>test</scope>
  </dependency>
  ```
- Пример (JUnit 5):
  ```java
  import org.junit.jupiter.api.Test;
  import org.testcontainers.containers.MySQLContainer;
  import org.testcontainers.utility.DockerImageName;
  import static org.junit.jupiter.api.Assertions.assertTrue;

  public class DatabaseTest {
      static MySQLContainer<?> mysql = new MySQLContainer<>(DockerImageName.parse("mysql:8.0"));

      @Test
      void testDatabaseConnection() {
          mysql.start();
          String jdbcUrl = mysql.getJdbcUrl();
          System.out.println("JDBC URL: " + jdbcUrl);
          assertTrue(mysql.isRunning());
          mysql.stop();
      }
  }
  ```
- Связь: Testcontainers создаёт изолированные среды для тестов БД.

### Lombok
- Библиотека для сокращения boilerplate-кода (геттеры, сеттеры, конструкторы) через аннотации.
- Maven зависимость:
  ```xml
  <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>1.18.34</version>
      <scope>provided</scope>
  </dependency>
  ```
- Пример в POM:
  ```java
  import lombok.Data;
  import lombok.AllArgsConstructor;

  @Data
  @AllArgsConstructor
  public class User {
      private String username;
      private String password;
  }

  public class LombokTest {
      @Test
      void testUser() {
          User user = new User("John", "pass123");
          assertEquals("John", user.getUsername());
      }
  }
  ```
- Связь: Lombok упрощает POM-классы в AQA.

### Page Object
- Шаблон для структурирования тестов: Каждый UI-экран — класс с элементами и методами.
- Пример:
  ```java
  import org.openqa.selenium.By;
  import org.openqa.selenium.WebDriver;
  import org.openqa.selenium.support.FindBy;
  import org.openqa.selenium.support.PageFactory;

  public class LoginPage {
      @FindBy(id = "username")
      private WebElement inputUsername;

      public LoginPage(WebDriver driver) {
          PageFactory.initElements(driver, this);
      }

      public void enterUsername(String username) {
          inputUsername.sendKeys(username);
      }
  }
  ```
- Связь: Page Object интегрируется с локаторами для устойчивых тестов.

### Retry
- Механизм для повторного запуска flaky тестов.
- В TestNG: Аннотация `@Test(retryAnalyzer = RetryAnalyzer.class)`.
- Пример RetryAnalyzer:
  ```java
  import org.testng.IRetryAnalyzer;
  import org.testng.ITestResult;

  public class RetryAnalyzer implements IRetryAnalyzer {
      private int retryCount = 0;
      private static final int maxRetryCount = 3;

      @Override
      public boolean retry(ITestResult result) {
          if (retryCount < maxRetryCount) {
              retryCount++;
              return true;
          }
          return false;
      }
  }
  ```
- В JUnit 5: Используйте расширения или flaky-тест-плагины.
- Связь: Retry помогает с нестабильными элементами.

### CI/CD (GitHub Actions)
- **Описание**: Автоматизация запуска тестов при пушах.
- Пример для Maven + JUnit/TestNG:
  ```yaml
  name: CI/CD with Maven
  on: [push]
  jobs:
    build-and-test:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v3
        - name: Set up JDK
          uses: actions/setup-java@v3
          with:
            java-version: '17'
        - name: Install dependencies
          run: mvn install
        - name: Run tests with Maven
          run: mvn test  # Запуск JUnit 5 или TestNG
  ```
- Связь: GitHub Actions интегрирует тесты с Selenium для проверки UI.

### Дополнительные практики
- **Allure**: Для отчётов.
    - Зависимость:
      ```xml
      <dependency>
          <groupId>io.qameta.allure</groupId>
          <artifactId>allure-junit5</artifactId>
          <version>2.29.0</version>
      </dependency>
      ```
    - Пример:
      ```java
      @Test
      @AllureId("123")
      @DisplayName("Тест логина")
      void testLogin() {
          // ...
      }
      ```
- **Logging (SLF4J + Logback)**:
    - Зависимости:
      ```xml
      <dependency>
          <groupId>org.slf4j</groupId>
          <artifactId>slf4j-api</artifactId>
          <version>2.0.16</version>
      </dependency>
      <dependency>
          <groupId>ch.qos.logback</groupId>
          <artifactId>logback-classic</artifactId>
          <version>1.5.12</version>
      </dependency>
      ```
    - Пример:
      ```java
      import org.slf4j.Logger;
      import org.slf4j.LoggerFactory;
  
      public class LoggingTest {
          private static final Logger logger = LoggerFactory.getLogger(LoggingTest.class);
  
          @Test
          void testWithLogging() {
              logger.info("Начало теста");
              logger.error("Ошибка в тесте");
          }
      }
      ```
- **Testcontainers**: Для контейнеризированных тестов (БД, Selenium Grid).
- **Lombok**: Для сокращения кода в POM-классах.

## Связь с другими темами
- **Клиент-серверная модель**: Jenkins запускает тесты для проверки взаимодействия клиента (UI) с сервером (API).
- **Локаторы**: В job Jenkins можно запускать тесты с локаторами для UI.
- **API-тестирование**: Maven-интеграция для запуска RestAssured-тестов.

## Источники
- [Скачать Jenkins](https://www.jenkins.io/download/)
- [Документация по cron-триггерам](https://www.jenkins.io/doc/book/pipeline/syntax/#triggers)
- [Maven Integration Plugin](https://plugins.jenkins.io/maven-plugin/)
- [Настройка Global Tool Configuration](https://www.jenkins.io/doc/book/managing/tools/)

---
[**← Назад к оглавлению**](README.md)