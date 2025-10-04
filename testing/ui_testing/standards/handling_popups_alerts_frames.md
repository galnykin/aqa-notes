# Обработка всплывающих окон, алертов, фреймов

## Работа с алертами

В UI-тестах алерты (всплывающие окна браузера) обрабатываются через `Alert` в Selenium WebDriver или методы Selenide. Используйте `switchTo().alert()` для взаимодействия с алертами: подтверждение, отклонение или получение текста.

Пример обработки алерта с Selenide:

```java
import com.codeborne.selenide.Selenide;
import static com.codeborne.selenide.Selenide.*;

public class AlertPage {
    private final SelenideElement triggerAlertButton = $("#alert-button");

    public void triggerAlert() {
        triggerAlertButton.click();
    }

    public String handleAlert() {
        String alertText = confirm(); // Подтверждает алерт и возвращает текст
        return alertText;
    }

    public void dismissAlert() {
        dismiss(); // Отклоняет алерт
    }
}
```

Пример теста с JUnit 5:

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static com.codeborne.selenide.Selenide.open;

public class AlertTest {
    @Test
    void testHandleAlert() {
        // Arrange
        AlertPage alertPage = new AlertPage();
        open("/alert-page");

        // Act
        alertPage.triggerAlert();
        String alertText = alertPage.handleAlert();

        // Assert
        assertEquals("Are you sure?", alertText, "Alert text should match");
    }
}
```

Пример теста с TestNG:

```java
import org.testng.annotations.Test;
import static org.testng.Assert.assertEquals;
import static com.codeborne.selenide.Selenide.open;

public class AlertTestNG {
    @Test
    void testHandleAlert() {
        // Given
        AlertPage alertPage = new AlertPage();
        open("/alert-page");

        // When
        alertPage.triggerAlert();
        String alertText = alertPage.handleAlert();

        // Then
        assertEquals(alertText, "Are you sure?", "Alert text should match");
    }
}
```

Для устойчивости:
- Используйте явные ожидания (`Selenide.Wait()`) перед взаимодействием с алертом.
- Проверяйте наличие алерта с помощью `try-catch` для обработки случаев, когда алерт не появляется.

## Переключение между фреймами

Фреймы (iframe) требуют переключения контекста с помощью `driver.switchTo().frame()`. После завершения работы возвращайтесь в основной контент с `switchTo().defaultContent()`.

Пример работы с фреймом в Selenide:

```java
import com.codeborne.selenide.SelenideElement;
import static com.codeborne.selenide.Selenide.*;

public class FramePage {
    private final SelenideElement frameContent = $("#frame-content");

    public void interactWithFrame(String frameId) {
        switchTo().frame(frameId);
        frameContent.shouldBe(visible).click();
        switchTo().defaultContent(); // Возврат в основной контент
    }
}
```

Пример теста с JUnit 5:

```java
import org.junit.jupiter.api.Test;
import static com.codeborne.selenide.Selenide.open;
import static com.codeborne.selenide.Condition.visible;

public class FrameTest {
    @Test
    void testInteractWithFrame() {
        // Arrange
        FramePage framePage = new FramePage();
        open("/frame-page");

        // Act
        framePage.interactWithFrame("myFrame");

        // Assert
        $("#main-content").shouldBe(visible); // Проверка возврата в основной контент
    }
}
```

Пример теста с TestNG:

```java
import org.testng.annotations.Test;
import static com.codeborne.selenide.Selenide.open;
import static com.codeborne.selenide.Condition.visible;

public class FrameTestNG {
    @Test
    void testInteractWithFrame() {
        // Given
        FramePage framePage = new FramePage();
        open("/frame-page");

        // When
        framePage.interactWithFrame("myFrame");

        // Then
        $("#main-content").shouldBe(visible); // Проверка возврата в основной контент
    }
}
```

Для устойчивости:
- Используйте стабильные локаторы для идентификации фреймов (ID или CSS).
- Проверяйте наличие фрейма с `isDisplayed()` перед переключением.

## Управление окнами и вкладками

Для работы с несколькими окнами или вкладками используйте `getWindowHandles()` и `switchTo().window()`. Это необходимо для тестирования сценариев с открытием новых вкладок (например, внешние ссылки).

Пример управления вкладками:

```java
import com.codeborne.selenide.Selenide;
import com.codeborne.selenide.SelenideElement;
import static com.codeborne.selenide.Selenide.*;

public class WindowPage {
    private final SelenideElement externalLink = $("#external-link");

    public void openNewTab() {
        String mainWindow = webdriver().driver().getWebDriver().getWindowHandle();
        externalLink.click();

        // Переключение на новую вкладку
        for (String windowHandle : webdriver().driver().getWebDriver().getWindowHandles()) {
            if (!windowHandle.equals(mainWindow)) {
                switchTo().window(windowHandle);
                break;
            }
        }

        $("#new-tab-content").shouldBe(visible, Duration.ofSeconds(5));
    }

    public void returnToMainWindow() {
        switchTo().window(0);
    }
}
```

Пример теста с JUnit 5:

```java
import org.junit.jupiter.api.Test;
import static com.codeborne.selenide.Selenide.open;
import static com.codeborne.selenide.Condition.visible;

public class WindowTest {
    @Test
    void testNewTabNavigation() {
        // Arrange
        WindowPage windowPage = new WindowPage();
        open("/window-page");

        // Act
        windowPage.openNewTab();

        // Assert
        $("#new-tab-content").shouldBe(visible);
        windowPage.returnToMainWindow();
        $("#main-content").shouldBe(visible);
    }
}
```

Пример теста с TestNG:

```java
import org.testng.annotations.Test;
import static com.codeborne.selenide.Selenide.open;
import static com.codeborne.selenide.Condition.visible;

public class WindowTestNG {
    @Test
    void testNewTabNavigation() {
        // Given
        WindowPage windowPage = new WindowPage();
        open("/window-page");

        // When
        windowPage.openNewTab();

        // Then
        $("#new-tab-content").shouldBe(visible);
        windowPage.returnToMainWindow();
        $("#main-content").shouldBe(visible);
    }
}
```

## Проверка наличия и содержимого всплывающих элементов

Всплывающие элементы, такие как модальные окна, требуют проверки их наличия и содержимого. Используйте `isDisplayed()` и явные ожидания для устойчивости.

Пример проверки модального окна:

```java
import com.codeborne.selenide.SelenideElement;
import static com.codeborne.selenide.Selenide.*;
import static com.codeborne.selenide.Condition.*;

public class ModalPage {
    private final SelenideElement modal = $("#modal");
    private final SelenideElement modalText = $("#modal-text");

    public void openModal() {
        $("#open-modal").click();
        modal.shouldBe(visible, Duration.ofSeconds(5));
    }

    public String getModalText() {
        return modalText.getText();
    }

    public boolean isModalDisplayed() {
        return modal.isDisplayed();
    }
}
```

Пример теста с JUnit 5:

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static com.codeborne.selenide.Selenide.open;

public class ModalTest {
    @Test
    void testModalContent() {
        // Arrange
        ModalPage modalPage = new ModalPage();
        open("/modal-page");

        // Act
        modalPage.openModal();

        // Assert
        assertTrue(modalPage.isModalDisplayed(), "Modal should be displayed");
        assertEquals("Modal content", modalPage.getModalText(), "Modal text should match");
    }
}
```

Пример теста с TestNG:

```java
import org.testng.annotations.Test;
import static org.testng.Assert.assertTrue;
import static org.testng.Assert.assertEquals;
import static com.codeborne.selenide.Selenide.open;

public class ModalTestNG {
    @Test
    void testModalContent() {
        // Given
        ModalPage modalPage = new ModalPage();
        open("/modal-page");

        // When
        modalPage.openModal();

        // Then
        assertTrue(modalPage.isModalDisplayed(), "Modal should be displayed");
        assertEquals(modalPage.getModalText(), "Modal content", "Modal text should match");
    }
}
```

## Отладка

Для отладки в IntelliJ IDEA:
- Устанавливайте точки останова в методах работы с алертами, фреймами или вкладками.
- Проверяйте `window handles` или текст алерта в окне Debug.
- Используйте DevTools для анализа локаторов всплывающих элементов и фреймов.
- Логируйте переключения контекста с помощью SLF4J для анализа ошибок.

Пример логирования:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class InteractionLogger {
    private static final Logger logger = LoggerFactory.getLogger(InteractionLogger.class);

    public void logFrameSwitch(String frameId) {
        logger.info("Switching to frame: {}", frameId);
    }

    public void logAlertInteraction(String action) {
        logger.info("Performing alert action: {}", action);
    }
}
```

## Источники
- [Selenide Documentation](https://selenide.org/documentation.html)
- [Selenium WebDriver Documentation](https://www.selenium.dev/documentation/en/)

---
[**← Назад к оглавлению**](../README.md)