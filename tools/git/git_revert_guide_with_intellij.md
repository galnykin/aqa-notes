# Выполнение revert commit в Git и IntelliJ IDEA

**Revert commit** в Git — это процесс создания нового коммита, который отменяет изменения, внесённые указанным коммитом, сохраняя историю изменений. Это особенно полезно в командной разработке, например, в автоматизации тестирования (AQA), где важно сохранять прозрачность истории. IntelliJ IDEA предоставляет удобный графический интерфейс для выполнения `git revert`, упрощая процесс, особенно при разрешении конфликтов. В данном руководстве описаны шаги для выполнения `git revert` как в командной строке, так и в IntelliJ IDEA, с примерами в контексте AQA и связью с другими темами, такими как локаторы, тестирование и клиент-серверная модель.

---

## 1. Что такое `git revert`?

- **Описание**: `git revert <commit>` создаёт новый коммит, который инвертирует изменения указанного коммита. Например, если коммит добавил тест, `git revert` создаст коммит, удаляющий этот тест.
- **Когда использовать**:
    - Отмена ошибочных изменений в общей ветке (`main`, `develop`).
    - Сохранение истории изменений (в отличие от `git reset`, который удаляет коммиты).
    - Исправление кода в AQA-проектах, например, некорректных тестов или локаторов.
- **Ключевые особенности**:
    - Безопасен для общих веток, так как не изменяет историю.
    - Может вызывать конфликты, которые нужно разрешать вручную.
    - Подходит для отмены как обычных, так и merge-коммитов.

---

## 2. Выполнение `git revert` в командной строке

### 2.1. Базовый процесс
1. **Найдите коммит для отмены**:
    - Используйте `git log` или `git log --oneline`:
      ```bash
      git log --oneline
      ```
      Пример вывода:
      ```
      abc1234 (HEAD -> main) Add login test
      def5678 Fix login page
      ghi9012 Initial commit
      ```
      Запомните хэш коммита (например, `abc1234`).

2. **Выполните `git revert`**:
    - Команда:
      ```bash
      git revert abc1234
      ```
    - Git откроет редактор для сообщения коммита (например, `Revert "Add login test"`).

3. **Сохраните сообщение**:
    - Сохраните и закройте редактор (в Vim: `:wq`).
    - Или используйте `--no-edit` для автоматического сообщения:
      ```bash
      git revert abc1234 --no-edit
      ```

4. **Проверьте результат**:
    - Новый revert-коммит появится в истории:
      ```bash
      git log --oneline
      ```
      ```
      fedcba9 (HEAD -> main) Revert "Add login test"
      abc1234 Add login test
      def5678 Fix login page
      ghi9012 Initial commit
      ```

5. **Отправьте изменения**:
   ```bash
   git push origin main
   ```

### 2.2. Работа с конфликтами
- Если изменения конфликтуют, Git сообщит:
  ```
  Auto-merging login_test.java
  CONFLICT (content): Merge conflict in login_test.java
  ```
- **Решение**:
    1. Откройте конфликтующий файл:
       ```
       <<<<<<< HEAD
       // Текущий код
       =======
       // Код из revert
       >>>>>>> abc1234
       ```
    2. Отредактируйте файл, оставив нужные изменения.
    3. Добавьте файл:
       ```bash
       git add login_test.java
       ```
    4. Завершите revert:
       ```bash
       git revert --continue
       ```

### 2.3. Отмена нескольких коммитов
- Для диапазона:
  ```bash
  git revert abc1234..def5678
  ```
- Для несмежных коммитов:
  ```bash
  git revert abc1234 def5678
  ```

### 2.4. Отмена merge-коммита
- Укажите родителя (обычно `-m 1`):
  ```bash
  git revert abc1234 -m 1
  ```

---

## 3. Выполнение `git revert` в IntelliJ IDEA

IntelliJ IDEA упрощает процесс `git revert` благодаря графическому интерфейсу и встроенным инструментам для разрешения конфликтов.

### 3.1. Шаги в IntelliJ IDEA
1. **Откройте историю коммитов**:
    - Перейдите в **VCS → Git → Show History** (или используйте вкладку **Commit** в правом верхнем углу).
    - Альтернатива: Нажмите **Alt+9** для открытия вкладки **Version Control** → **Log**.

2. **Найдите коммит**:
    - В вкладке **Log** найдите коммит (например, `abc1234 Add login test`).
    - Щёлкните правой кнопкой мыши на коммите и выберите **Revert Commit**.

3. **Подтвердите действие**:
    - IntelliJ покажет изменения, которые будут отменены.
    - Подтвердите, нажав **Revert**.
    - IntelliJ автоматически создаст новый коммит с сообщением вида `Revert "Add login test"`.

4. **Разрешение конфликтов (если есть)**:
    - Если возникает конфликт, IntelliJ откроет окно **Merge Conflicts**:
        - Выберите, какие изменения оставить (Accept Yours, Accept Theirs или Merge вручную).
        - После разрешения нажмите **Apply**.
    - IntelliJ добавит файлы в индекс и предложит завершить revert.

5. **Отправьте изменения**:
    - Нажмите **VCS → Git → Push** или используйте **Ctrl+Shift+K** для отправки в удалённый репозиторий.

### 3.2. Полезные функции IntelliJ
- **Предпросмотр изменений**: Перед revert IntelliJ показывает дифф (разницу) коммита.
- **Графический конфликт-резолвер**: Удобный интерфейс для разрешения конфликтов.
- **Интеграция с CI/CD**: IntelliJ поддерживает запуск тестов (например, JUnit) после revert через **Run → Run Tests**.
- **Автоматическое сообщение**: IntelliJ генерирует сообщение коммита, но вы можете его отредактировать.

---

## 4. Практические примеры в контексте AQA

### 4.1. Отмена теста в Selenium (Java)
Предположим, вы добавили некорректный тест:
```java
// login_test.java (коммит abc1234)
@Test
void testInvalidLogin() {
    driver.get("https://example.com");
    driver.findElement(By.id("wrong-id")).sendKeys("invalid"); // Ошибочный локатор
}
```

#### В командной строке:
1. Найдите коммит:
   ```bash
   git log --oneline
   # abc1234 Add invalid login test
   ```
2. Выполните revert:
   ```bash
   git revert abc1234 --no-edit
   ```
3. Проверьте:
   ```bash
   git diff HEAD^ HEAD
   ```
4. Запустите тесты:
   ```bash
   mvn test
   ```

#### В IntelliJ IDEA:
1. Откройте **Version Control → Log**.
2. Найдите коммит `abc1234`, щёлкните ПКМ → **Revert Commit**.
3. Подтвердите и проверьте изменения в **Commit** панели.
4. Запустите тесты: **Run → Run 'All Tests'**.

### 4.2. Отмена изменений в локаторах
Если вы изменили локатор в Page Object Model:
```java
// login_page.java (коммит def5678)
public class LoginPage {
    private By username = By.xpath("//input[@id='wrong-id']"); // Ошибка
}
```

#### В IntelliJ IDEA:
1. В **Log** найдите `def5678`, выберите **Revert Commit**.
2. Если возникает конфликт, используйте конфликт-резолвер IntelliJ для восстановления `By.id("username")`.
3. Завершите revert и отправьте изменения.

---

## 5. Связь с другими темами

- **Локаторы и селекторы**:
    - Revert полезен для отмены некорректных локаторов, например, замена хрупкого XPath (`//input[1]`) на стабильный `By.id`:
      ```java
      // До: By.xpath("//input[1]")
      // После revert: By.id("username")
      ```

- **JSON Wire Protocol и RESTful Web Service**:
    - Если вы тестируете API в AQA, revert может отменить изменения в тестах, использующих неправильные эндпоинты:
      ```python
      # До: requests.get("https://api.example.com/wrong-endpoint")
      # После revert: requests.get("https://api.example.com/correct-endpoint")
      ```

- **Клиент-серверная модель**:
    - Revert помогает исправить конфигурацию Selenium Grid, например, неправильный URL хаба:
      ```java
      // До: new RemoteWebDriver(new URL("http://wrong-hub:4444"), capabilities)
      // После revert: new RemoteWebDriver(new URL("http://correct-hub:4444"), capabilities)
      ```

- **Тестирование (JUnit-подобные подходы)**:
    - Revert отменяет ошибочные тесты JUnit 5, сохраняя историю:
      ```java
      @Test
      void testBroken() {
          assertTrue(false); // Ошибочный тест
      }
      ```
    - После revert тест удаляется, и вы можете проверить стабильность через `mvn test`.

- **ООП**:
    - Revert восстанавливает стабильные реализации Page Object Model:
      ```java
      public class LoginPage {
          private By submitButton = By.cssSelector("[data-testid='submit']"); // Стабильный локатор
      }
      ```

---

## 6. Преимущества и недостатки `git revert`

### Преимущества
- **Сохранение истории**: Не изменяет существующую историю, безопасно для командной работы.
- **Прозрачность**: Новый коммит ясно указывает на отмену.
- **Интеграция с IntelliJ**: Графический интерфейс упрощает процесс и разрешение конфликтов.
- **Подходит для AQA**: Позволяет быстро исправлять тесты или локаторы.

### Недостатки
- **Конфликты**: Требуют ручного разрешения, что может быть сложно без опыта.
- **Увеличение истории**: Добавляет новые коммиты, что может усложнить `git log`.
- **Merge-коммиты**: Требуют указания родителя (`-m`).

---

## 7. Рекомендации

- **Используйте `git revert` для общих веток**:
    - Не применяйте `git reset` в `main` или `develop`, чтобы не нарушить историю.
- **Проверяйте изменения**:
    - В командной строке: `git show <commit>`.
    - В IntelliJ: Используйте предпросмотр диффа перед revert.
- **Разрешайте конфликты в IntelliJ**:
    - Графический резолвер IntelliJ упрощает работу с конфликтами.
- **Тестируйте после revert**:
    - Запустите автотесты (например, JUnit 5) после revert:
      ```bash
      mvn test
      ```
- **Документируйте**:
    - Указывайте причину в сообщении:
      ```bash
      git revert abc1234 -m "Reverting due to incorrect locator"
      ```
- **Интеграция с CI/CD**:
    - Настройте запуск тестов после revert в GitHub Actions или Jenkins.

---

## 8. Полный пример в контексте AQA

### Сценарий
Вы добавили тест с некорректным локатором:
```java
// login_test.java (коммит abc1234)
@Test
void testLogin() {
    driver.get("https://example.com");
    driver.findElement(By.xpath("//input[@id='wrong-id']")).sendKeys("testuser");
}
```

#### В командной строке:
1. Найдите коммит:
   ```bash
   git log --oneline
   # abc1234 Add login test with wrong locator
   ```
2. Выполните revert:
   ```bash
   git revert abc1234 --no-edit
   ```
3. Проверьте:
   ```bash
   git diff HEAD^ HEAD
   ```
4. Запустите тесты:
   ```bash
   mvn test
   ```
5. Отправьте:
   ```bash
   git push origin main
   ```

#### В IntelliJ IDEA:
1. Откройте **Version Control → Log** (Alt+9).
2. Найдите `abc1234`, ПКМ → **Revert Commit**.
3. Подтвердите, разрешите конфликты (если есть) в резолвере.
4. Запустите тесты: **Run → Run 'All Tests'**.
5. Отправьте изменения: **VCS → Git → Push**.

---

## 9. Источники
- [Git Documentation: git revert](https://git-scm.com/docs/git-revert)
- [Atlassian: Git Revert Tutorial](https://www.atlassian.com/git/tutorials/undoing-changes/git-revert)
- [JetBrains: Revert Commits in IntelliJ](https://www.jetbrains.com/help/idea/undo-changes.html#revert-commit)
- [Baeldung: Git Revert](https://www.baeldung.com/git-revert)

---
[**← Назад к оглавлению**](README.md)