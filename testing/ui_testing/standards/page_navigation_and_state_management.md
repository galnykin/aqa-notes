# Навигация между страницами и состояниями

## Явные переходы между страницами

Для навигации между страницами в UI-тестах с использованием Selenium WebDriver или Selenide рекомендуется использовать методы, такие как `openPage()` или `navigateToSection()`. Эти методы инкапсулируют логику перехода, обеспечивая читаемость и переиспользуемость кода в рамках Page Object Model (POM).

Пример реализации `openPage()` в классе Page Object с Selenide:

```java
import com.codeborne.selenide.Selenide;
import com.codeborne.selenide.SelenideElement;
import static com.codeborne.selenide.Selenide.$;

public class DashboardPage {
    private final SelenideElement dashboardHeader = $("#dashboard-header");

    public void openPage() {
        Selenide.open("/dashboard");
        dashboardHeader.shouldBe(visible, Duration.ofSeconds(5)); // Проверка загрузки страницы
    }

    public void navigateToSection(String section) {
        $(String.format("a[href*='%s']", section)).click();
    }
}
```

Пример теста с JUnit 5:

```java
import org.junit.jupiter.api.Test;
import static com.codeborne.selenide.Condition.visible;
import static com.codeborne.selenide.Selenide.$;

public class NavigationTest {
    @Test
    void testNavigateToDashboard() {
        // Arrange
        DashboardPage dashboardPage = new DashboardPage();

        // Act
        dashboardPage.openPage();

        // Assert
        $("#dashboard-header").shouldBe(visible);
    }
}
```

Пример теста с TestNG:

```java
import org.testng.annotations.Test;
import static com.codeborne.selenide.Condition.visible;
import static com.codeborne.selenide.Selenide.$;

public class NavigationTestNG {
    @Test
    void testNavigateToDashboard() {
        // Given
        DashboardPage dashboardPage = new DashboardPage();

        // When
        dashboardPage.openPage();

        // Then
        $("#dashboard-header").shouldBe(visible);
    }
}
```

Для устойчивости тестов:
- Используйте стабильные локаторы (например, CSS-селекторы по ID или классам).
- Добавляйте явные ожидания с помощью `shouldBe(visible)` для проверки загрузки страницы.
- Проверяйте состояние элементов с помощью `isDisplayed()` и `isEnabled()`.

## Проверка загрузки страницы

Проверка загрузки страницы необходима для подтверждения корректной работы навигации. Используйте ключевые элементы, уникальные для страницы, чтобы убедиться, что она полностью загружена. Selenide упрощает этот процесс с помощью встроенных условий.

Пример проверки загрузки страницы:

```java
import com.codeborne.selenide.SelenideElement;
import static com.codeborne.selenide.Condition.visible;
import static com.codeborne.selenide.Selenide.$;

public class ProfilePage {
    private final SelenideElement profileTitle = $("#profile-title");

    public void openPage() {
        Selenide.open("/profile");
        profileTitle.shouldBe(visible, Duration.ofSeconds(5)); // Ожидание загрузки
    }

    public boolean isPageLoaded() {
        return profileTitle.isDisplayed();
    }
}
```

В тестах проверяйте результат:

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertTrue;

public class ProfilePageTest {
    @Test
    void testProfilePageLoads() {
        // Arrange
        ProfilePage profilePage = new ProfilePage();

        // Act
        profilePage.openPage();

        // Assert
        assertTrue(profilePage.isPageLoaded(), "Profile page should be loaded");
    }
}
```

Для TestNG:

```java
import org.testng.annotations.Test;
import static org.testng.Assert.assertTrue;

public class ProfilePageTestNG {
    @Test
    void testProfilePageLoads() {
        // Given
        ProfilePage profilePage = new ProfilePage();

        // When
        profilePage.openPage();

        // Then
        assertTrue(profilePage.isPageLoaded(), "Profile page should be loaded");
    }
}
```

## Использование URL-шаблонов и параметров

URL-шаблоны позволяют динамически формировать адреса страниц, что полезно для тестирования страниц с параметрами (например, `/user/{id}`). Используйте `String.format` или библиотеки для работы с URL.

Пример работы с URL-шаблонами:

```java
import com.codeborne.selenide.Selenide;
import com.codeborne.selenide.SelenideElement;
import static com.codeborne.selenide.Condition.visible;
import static com.codeborne.selenide.Selenide.$;

public class UserPage {
    private final SelenideElement userName = $("#user-name");

    public void openUserPage(String userId) {
        String url = String.format("/user/%s", userId);
        Selenide.open(url);
        userName.shouldBe(visible, Duration.ofSeconds(5));
    }
}
```

Пример теста с динамическим URL:

```java
import org.junit.jupiter.api.Test;

public class UserPageTest {
    @Test
    void testOpenUserPage() {
        // Arrange
        UserPage userPage = new UserPage();
        String userId = "123";

        // Act
        userPage.openUserPage(userId);

        // Assert
        $("#user-name").shouldHave(text("User 123"));
    }
}
```

Для TestNG:

```java
import org.testng.annotations.Test;

public class UserPageTestNG {
    @Test
    void testOpenUserPage() {
        // Given
        UserPage userPage = new UserPage();
        String userId = "123";

        // When
        userPage.openUserPage(userId);

        // Then
        $("#user-name").shouldHave(text("User 123"));
    }
}
```

## Переходы между вкладками и окнами

Selenium WebDriver и Selenide поддерживают работу с несколькими вкладками и окнами через управление `window handles`. Это полезно для тестирования сценариев, где открываются новые вкладки (например, при переходе по внешним ссылкам).

Пример переключения между вкладками:

```java
import com.codeborne.selenide.Selenide;
import com.codeborne.selenide.SelenideElement;
import static com.codeborne.selenide.Selenide.$;
import static com.codeborne.selenide.Selenide.switchTo;

public class ExternalLinkPage {
    private final SelenideElement externalLink = $("#external-link");

    public void openExternalLinkInNewTab() {
        String mainWindow = Selenide.webdriver().driver().getWebDriver().getWindowHandle();
        externalLink.click(); // Открывает новую вкладку

        // Переключение на новую вкладку
        for (String windowHandle : Selenide.webdriver().driver().getWebDriver().getWindowHandles()) {
            if (!windowHandle.equals(mainWindow)) {
                switchTo().window(windowHandle);
                break;
            }
        }

        // Проверка загрузки новой вкладки
        $("#new-tab-content").shouldBe(visible, Duration.ofSeconds(5));
    }

    public void returnToMainWindow() {
        switchTo().window(0); // Возврат к первой вкладке
    }
}
```

Пример теста с JUnit 5:

```java
import org.junit.jupiter.api.Test;

public class ExternalLinkTest {
    @Test
    void testOpenExternalLink() {
        // Arrange
        ExternalLinkPage externalLinkPage = new ExternalLinkPage();

        // Act
        externalLinkPage.openExternalLinkInNewTab();

        // Assert
        $("#new-tab-content").shouldBe(visible);
        externalLinkPage.returnToMainWindow();
        $("#main-content").shouldBe(visible);
    }
}
```

Пример теста с TestNG:

```java
import org.testng.annotations.Test;

public class ExternalLinkTestNG {
    @Test
    void testOpenExternalLink() {
        // Given
        ExternalLinkPage externalLinkPage = new ExternalLinkPage();

        // When
        externalLinkPage.openExternalLinkInNewTab();

        // Then
        $("#new-tab-content").shouldBe(visible);
        externalLinkPage.returnToMainWindow();
        $("#main-content").shouldBe(visible);
    }
}
```

## Отладка

Для отладки навигации в IntelliJ IDEA:
- Устанавливайте точки останова в методах `openPage()` или `navigateToSection()` для проверки URL и состояния страницы.
- Используйте DevTools браузера для анализа локаторов ключевых элементов.
- Проверяйте `window handles` в дебаггере при работе с вкладками.
- Логируйте URL и состояние элементов с помощью SLF4J для анализа ошибок.

Пример логирования:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class NavigationLogger {
    private static final Logger logger = LoggerFactory.getLogger(NavigationLogger.class);

    public void logNavigation(String pageUrl) {
        logger.info("Navigating to: {}", pageUrl);
    }
}
```

## Источники
- [Selenide Documentation](https://selenide.org/documentation.html)
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/en/)

---
[**← Назад к оглавлению**](../README.md)