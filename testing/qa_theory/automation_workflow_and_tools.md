# Практические знания автоматизатора: инструменты, процессы, сравнения

Автоматизация тестирования — это не только написание тестов, но и глубокое понимание процессов разработки, инструментов и архитектурных решений. Ниже собраны ключевые темы, которые должен знать современный инженер по автоматизации: от различий между API и фреймворками до команд Git и CI-процессов.

-----

## 🌐 Web App vs Mobile App API

Хотя оба типа API служат для обмена данными, их контекст и окружение различаются. Для автоматизатора на **Java + Rest Assured** ключевые отличия заключаются в конфигурации запроса.

- **Web App API**:

    - **Контекст**: Запросы отправляются из браузера. Аутентификация часто основана на **Cookies** или **JWT (JSON Web Token)** в заголовке `Authorization`.
    - **Тестирование**: Легко отслеживать в **DevTools** браузера. Тесты на **Rest Assured** имитируют эти браузерные запросы.

- **Mobile App API**:

    - **Контекст**: Запросы инициируются нативным кодом (Swift/Kotlin). Часто используется более строгая безопасность: **привязка SSL-сертификата (SSL Pinning)**, шифрование тела запроса, временные токены и специфичные заголовки (`User-Agent`, `X-Device-Id`).
    - **Тестирование**: Требует использования прокси-инструментов (Charles, Fiddler) для анализа трафика с устройства или эмулятора. Тесты на **Rest Assured** должны точно воспроизводить все необходимые заголовки и методы аутентификации.

### Пример на Rest Assured

Код для тестирования обоих типов API будет структурно похож. Разница — в деталях: URL, заголовках и теле запроса.

```java
import io.restassured.RestAssured;
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test; // Для JUnit 5
// import org.testng.annotations.Test; // Для TestNG
import static org.assertj.core.api.Assertions.assertThat;

public class ApiTest {

    @Test
    void testWebAppLogin() {
        // Условный API веб-приложения
        String responseBody = RestAssured
            .given()
                .contentType(ContentType.JSON)
                .body("{\"email\": \"user@example.com\", \"password\": \"secret\"}")
            .when()
                .post("https://webapp.api/v1/login")
            .then()
                .statusCode(200)
                .extract().body().asString();

        assertThat(responseBody).contains("authToken");
    }

    @Test
    void testMobileAppGetUserProfile() {
        // Условный API мобильного приложения
        String userProfile = RestAssured
            .given()
                // Специфичные заголовки для мобильного API
                .header("User-Agent", "MyApp/1.2.3 (Android 13)")
                .header("Authorization", "Bearer mobile_specific_jwt_token")
            .when()
                .get("https://mobileapp.api/v2/user/profile")
            .then()
                .statusCode(200)
                .extract().body().asString();

        assertThat(userProfile).contains("userId");
    }
}
```

-----

## 🧪 Selenium vs Selenide

**Selenium WebDriver** — это стандарт де-факто и низкоуровневый фреймворк, дающий полный контроль, но требующий написания boilerplate-кода (ожидания, поиск элементов, скриншоты). **Selenide** — это высокоуровневая обёртка над Selenium, созданная для упрощения UI-тестов и повышения их стабильности.

| Параметр | Selenium WebDriver | Selenide |
|---|---|---|
| **API** | Низкоуровневый: `driver.findElement(By.id("..."))` | Высокоуровневый и лаконичный: `$("#...")` |
| **Ожидания** | Нужно явно прописывать `WebDriverWait` | Встроенные "умные" ожидания. Тест ждёт, пока элемент станет видимым. |
| **Синтаксис** | Более громоздкий, требует больше строк кода | Лаконичный, интуитивно понятный (fluent API) |
| **Стабильность** | "Flaky" тесты из-за проблем с ожиданиями (StaleElementReferenceException) | Значительно стабильнее благодаря автоматическим ожиданиям и повторным попыткам. |
| **Настройка** | Требует ручной настройки драйвера (например, через **WebDriverManager**) | WebDriverManager встроен "под капотом". Настройка минимальна. |

### Сравнение в коде

**Задача**: Найти поле, ввести текст и нажать кнопку.

#### Selenium WebDriver + WebDriverManager + JUnit 5/TestNG

```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import io.github.bonigarcia.wdm.WebDriverManager;
import java.time.Duration;

// ... в тестовом классе
WebDriver driver;

@BeforeEach // JUnit 5
// @BeforeMethod // TestNG
void setUp() {
    WebDriverManager.chromedriver().setup();
    driver = new ChromeDriver();
}

@Test
void seleniumTest() {
    driver.get("https://example.com");
    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    
    WebElement input = wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("search-input")));
    input.sendKeys("test query");

    WebElement button = driver.findElement(By.cssSelector(".search-button"));
    button.click();
}
```

#### Selenide + JUnit 5/TestNG (весь код)

```java
import com.codeborne.selenide.SelenideElement;
import static com.codeborne.selenide.Selenide.*;
import static com.codeborne.selenide.Condition.*;

// ... в тестовом классе
@Test
void selenideTest() {
    open("https://example.com");

    // Ожидания встроены в каждую команду
    $("#search-input").setValue("test query");
    $(".search-button").click();
    
    // Встроенные проверки (ассерты)
    $(".results").shouldHave(text("результаты по запросу"));
}
```

-----

## 🔁 RestAssured vs другие HTTP-клиенты

В экосистеме Java существует множество HTTP-клиентов, но их цели различаются. **Rest Assured** специально создан **для тестирования** API.

| Клиент | Основное назначение | Ключевые преимущества в контексте стека |
|---|---|---|
| **RestAssured** | **Тестирование REST API.** | **Fluent BDD-синтаксис** (`given/when/then`), встроенные механизмы валидации, простая интеграция с **JUnit/TestNG** и отчётами **Allure**. |
| **HttpClient** | Низкоуровневая работа с HTTP в production-коде. | Максимальная гибкость, но избыточен и сложен для написания тестов. |
| **OkHttp** | Эффективный HTTP-клиент, популярен в Android и Java-приложениях. | Производительность, поддержка HTTP/2. Для тестов менее удобен, чем Rest Assured. |
| **Retrofit** | Декларативный API-клиент. Описываешь интерфейс, Retrofit реализует его. | Идеален для клиентского кода (Android, JavaFX), но не для написания тестов. |

### Почему Rest Assured — лучший выбор для автотестов?

Он объединяет отправку запроса, парсинг ответа и ассерты в единый, читаемый блок.

#### JUnit 5

```java
import io.restassured.RestAssured;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

@Test
void getUserById_ReturnsCorrectUser() {
    given()
        .baseUri("https://reqres.in/api")
        .log().all() // Логирование запроса с помощью SLF4J
    .when()
        .get("/users/2")
    .then()
        .log().all() // Логирование ответа
        .statusCode(200)
        .body("data.id", equalTo(2))
        .body("data.first_name", equalTo("Janet"));
}
```

#### TestNG

```java
import io.restassured.RestAssured;
import org.testng.annotations.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.equalTo;

@Test
public class UserApiTest {
    @Test
    void getUserById_ReturnsCorrectUser() {
        given()
            // ... тот же код, что и в JUnit 5
        .when()
            .get("https://reqres.in/api/users/2")
        .then()
            .statusCode(200)
            .body("data.id", equalTo(2));
    }
}
```

-----

## ⚙️ Maven Lifecycle Commands

**Maven** управляет сборкой проекта на основе жизненного цикла, состоящего из фаз. Каждая фаза представляет собой шаг в процессе сборки.

- `mvn clean` — **Очистка**. Удаляет директорию `target`, где хранятся скомпилированные классы и собранные артефакты.
- `mvn validate` — **Валидация**. Проверяет корректность `pom.xml`.
- `mvn compile` — **Компиляция**. Компилирует исходный код проекта (из `src/main/java`) в байт-код и помещает его в `target/classes`.
- `mvn test` — **Тестирование**. Запускает тесты (из `src/test/java`) с помощью плагина **Surefire**. Сначала выполняет все предыдущие фазы (`compile` и др.). Это самая частая команда для локального запуска тестов.
- `mvn package` — **Упаковка**. Собирает скомпилированный код в дистрибутив, обычно **JAR**-файл.
- `mvn verify` — **Проверка**. Выполняет все проверки, включая интеграционные тесты (плагин **Failsafe**).
- `mvn install` — **Установка**. Копирует собранный артефакт в ваш локальный репозиторий Maven (`~/.m2`), делая его доступным для других ваших проектов.
- `mvn deploy` — **Развёртывание**. Копирует артефакт в удалённый репозиторий (например, Nexus, Artifactory), делая его доступным для других разработчиков.

В **CI/CD (GitHub Actions)** пайплайне чаще всего используется команда `mvn clean verify` или `mvn clean test`, чтобы полностью проверить проект перед слиянием.

-----

## 📦 JAR vs WAR

| Формат | Расшифровка | Назначение | Структура |
|---|---|---|---|
| **JAR** | **J**ava **AR**chive | Библиотека, консольное приложение или самодостаточный микросервис. | Содержит `.class` файлы, метаданные (`META-INF`) и ресурсы. Может быть исполняемым, если содержит `main` класс. |
| **WAR** | **W**eb **A**rchive | Веб-приложение, предназначенное для развёртывания на сервере приложений (Tomcat, Jetty). | Содержит `.class` файлы, JSP, HTML, JavaScript, а также библиотеки в `WEB-INF/lib`. |

**Контекст для автоматизатора:**

* Ваш фреймворк для автотестов всегда будет собираться в **JAR**.
* Тестируемое приложение может быть развёрнуто из **WAR** или запущено как исполняемый **JAR** (типично для Spring Boot).
* С помощью **Testcontainers** вы можете в своих тестах поднять Docker-контейнер с тестируемым приложением, запущенным из JAR/WAR, обеспечив полностью изолированную среду для тестов.

-----

## 🔀 Git: конфликт master vs branch

Конфликт слияния (merge conflict) возникает, когда Git не может автоматически объединить изменения из разных веток, так как они затрагивают одни и те же строки в одном файле.

**Пример сценария:**

1.  Вы создали ветку `feature/login-test` от `master`.
2.  В вашей ветке вы изменили локатор в классе `LoginPage.java`.
3.  Одновременно другой разработчик влил в `master` изменения, которые затрагивали тот же локатор.
4.  Вы пытаетесь влить свежий `master` в свою ветку (`git pull origin master`) и получаете конфликт.

**Процесс решения:**

1.  Git покажет conflicted-файлы. Откройте их в IDE.
2.  Вы увидите маркеры конфликта:
    ```java
    <<<<<<< HEAD // Начало ваших изменений
    private final String loginButton = "#login-btn-new";
    ======= // Разделитель
    private final String loginButton = "#login-button-main";
    >>>>>>> master // Конец изменений из ветки master
    ```
3.  **Вручную** отредактируйте код, оставив правильную версию и удалив маркеры Git.
4.  Сообщите Git, что конфликт разрешён:
    ```bash
    git add src/main/java/com/example/LoginPage.java
    ```
5.  Завершите слияние, создав коммит:
    ```bash
    git commit -m "Merge branch 'master' into feature/login-test and resolve conflicts"
    ```

-----

## 📥 Pull Request vs Merge Request

Это одно и то же по своей сути — запрос на вливание изменений из одной ветки в другую. Название зависит от платформы.

| Платформа | Название | Сокращение |
|---|---|---|
| **GitHub**, Bitbucket | Pull Request | **PR** |
| **GitLab**, Azure DevOps | Merge Request | **MR** |

**Важность для автоматизации:**
Создание PR/MR — это ключевой триггер для **CI/CD** пайплайна. В **GitHub Actions** вы настраиваете workflow, который автоматически запускается при создании PR в `master`. Этот workflow выполняет шаги:

1.  `checkout` кода.
2.  Настройка **Java 17**.
3.  Запуск тестов: `mvn clean test`.
4.  Генерация **Allure Report**.
5.  Публикация отчёта и статуса тестов обратно в PR.

Это гарантирует, что нерабочий код не попадёт в основную ветку.

-----

## 🔍 Code Review и варианты слияния

После одобрения PR/MR его нужно влить. Существует несколько стратегий:

- **Create a Merge Commit**: Создаёт новый "коммит слияния" в целевой ветке (`master`). Сохраняет всю историю коммитов из feature-ветки. История становится нелинейной и более сложной, но информативной.
- **Squash and Merge**: Объединяет все коммиты из feature-ветки в **один большой коммит** и добавляет его в `master`. Делает историю `master` очень чистой и линейной. Идеально, если промежуточные коммиты (`"fix typo"`, `"wip"`) не важны.
- **Rebase and Merge**: "Перемещает" коммиты из feature-ветки поверх последних изменений в `master`, а затем вливает их. Делает историю линейной без создания коммита слияния. Считается самой чистой, но и самой сложной стратегией (требует `git rebase`).

**Чаще всего используется `Squash and Merge` для чистоты истории основной ветки.**

-----

## 📌 Последовательность работы над задачей в Git

1.  **Получить задачу**: Например, в Jira `TEST-42: Implement login page tests`.
2.  **Переключиться на основную ветку и обновить её**:
    ```bash
    git checkout master
    git pull origin master 
    ```
3.  **Создать новую ветку от `master`**: Имя ветки должно быть информативным.
    ```bash
    # Формат: type/ticket-id-short-description
    git checkout -b feature/TEST-42-login-page-tests
    ```
4.  **Написать код и тесты**: Реализовать `LoginPage.java` и `LoginTest.java`.
5.  **Запустить тесты локально**: Убедиться, что всё работает.
    ```bash
    mvn clean test
    ```
6.  **Сделать коммит (или несколько)**: Коммиты должны быть атомарными.
    ```bash
    git add .
    git commit -m "TEST-42: Add Page Object for Login and positive test case"
    ```
7.  **Отправить ветку в удалённый репозиторий**:
    ```bash
    git push -u origin feature/TEST-42-login-page-tests
    ```
8.  **Создать Pull Request** в GitHub/GitLab, добавить описание и запросить ревью.
9.  **CI/CD пайплайн** автоматически запустит тесты.
10. После одобрения и успешного билда **слить PR/MR** в `master`.

-----

## 📊 Логирование и отчётность в тестах

### Logging (SLF4J + Logback)

Логирование — это запись информации о ходе выполнения теста. Это критически важно для анализа упавших тестов в CI.

- **SLF4J** — это фасад (API), который вы используете в коде.
- **Logback** — это реализация, которая пишет логи в консоль или файл.

С помощью **Lombok** можно легко добавить логгер в класс:

```java
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;

@Slf4j // Аннотация Lombok
public class MyLoggingTest {

    @Test
    void someActionTest() {
        log.info("Начинаем тест..."); // Уровень INFO
        // ... тестовая логика ...
        int userId = 123;
        log.debug("Получили ID пользователя: {}", userId); // Уровень DEBUG
        
        try {
            // ... что-то, что может упасть
        } catch (Exception e) {
            log.error("Произошла ошибка при выполнении действия!", e); // Уровень ERROR с трассировкой
        }
        log.info("Тест завершен.");
    }
}
```

### Allure Framework

**Allure** — это инструмент для создания интерактивных и наглядных отчётов о тестировании. Он интегрируется с **JUnit 5** и **TestNG** через аннотации.

```java
import io.qameta.allure.*;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

@Epic("Личный кабинет")
@Feature("Авторизация пользователя")
@DisplayName("Тесты на авторизацию")
public class LoginAllureTest {

    @Test
    @Story("Успешная авторизация")
    @Severity(SeverityLevel.BLOCKER)
    @Description("Тест проверяет, что зарегистрированный пользователь может успешно войти в систему.")
    @Owner("Иванов И.И.")
    void successfulLoginTest() {
        openLoginPage();
        enterCredentials("user", "password");
        verifyLoginSuccess();
    }

    @Step("Открываем страницу логина")
    private void openLoginPage() { /* ... */ }

    @Step("Вводим логин '{username}' и пароль")
    private void enterCredentials(String username, String password) { /* ... */ }

    @Step("Проверяем успешную авторизацию")
    private void verifyLoginSuccess() { /* ... */ }
}
```

После запуска тестов командой `mvn clean test` Allure сгенерирует результаты, из которых можно построить HTML-отчёт командой `allure serve`.

-----

[**← Назад к оглавлению**](README.md)