# Занятие 1: Введение в Python, Hello World

Это руководство посвящено первой лекции курса по Python, фокусирующейся на введении в язык, его характеристиках, установке Python и PyCharm, а также написании первой программы "Hello World". Мы разберём каждую тему в контексте автоматизации тестирования (AQA), где Python широко используется благодаря простоте и мощным библиотекам, таким как Selenium и pytest.

---

## 1. Введение в Python, Hello World

**Цель занятия**: Познакомить с основами Python, установить необходимые инструменты и написать первую программу. Это базовый шаг для начинающих QA-инженеров, которые будут использовать Python для написания автотестов (например, с Selenium WebDriver).

**План занятия**:
1. Объяснение, что такое Python и его роль в AQA.
2. Установка Python и настройка окружения.
3. Установка PyCharm как основной IDE.
4. Написание и запуск программы "Hello World".
5. Практика: запуск кода, отладка.

**Связь с AQA**: Python — один из самых популярных языков для автоматизации тестирования благодаря простоте синтаксиса, богатой экосистеме (Selenium, pytest, requests) и поддержке в CI/CD.

---

## 2.Введение в Python

### 2.1. Общая характеристика Python
- **Описание**: Python — высокоуровневый, интерпретируемый язык программирования с акцентом на читаемость и простоту. Создан Гвидо ван Россумом в 1991 году.
- **Ключевые особенности**:
    - **Простота синтаксиса**: Минималистичный, легко читаемый код, что делает его идеальным для начинающих в AQA.
    - **Кроссплатформенность**: Работает на Windows, macOS, Linux.
    - **Интерпретируемый**: Код выполняется построчно, без компиляции.
    - **Динамическая типизация**: Не нужно явно указывать типы переменных.
    - **Богатая экосистема**: Библиотеки для AQA (Selenium, pytest, requests), анализа данных (pandas), DevOps (ansible).
    - **Сообщество**: Активное, с множеством ресурсов и документации.
- **Применение в AQA**:
    - Тестирование веб-приложений (Selenium, Playwright).
    - Тестирование API (requests).
    - Генерация тестовых данных (faker).
    - Интеграция с CI/CD (Jenkins, GitHub Actions).

**Связь с другими темами**:
- **Клиент-серверная модель**: Python-код (например, Selenium) отправляет запросы к серверу (браузеру или API), как клиент в RESTful API.
- **Тестирование**: Python с pytest аналогичен JUnit для структурированных тестов.

**Пример кода**:
```python
print("Hello, AQA!")  # Простой вывод для теста
```

---

### 2.2. Установка Python

#### 2.2.1. Шаги установки
1. **Скачивание**:
    - Перейдите на [python.org](https://www.python.org/downloads/).
    - Рекомендуемая версия: Python 3.12 или новее (на август 2025).
    - Выберите установщик для вашей ОС (Windows, macOS, Linux).
2. **Установка**:
    - **Windows**:
        - Запустите установщик.
        - Убедитесь, что отмечена опция **Add Python to PATH**.
        - Выберите **Install Now** или настройте путь (например, `C:\Python312`).
    - **macOS**:
        - Установите через установщик или Homebrew (`brew install python`).
    - **Linux**:
        - Установите через пакетный менеджер, например: `sudo apt install python3` (Ubuntu).
3. **Проверка**:
    - Откройте терминал (Command Prompt, Terminal).
    - Выполните:
      ```bash
      python --version
      ```
      Ожидаемый вывод: `Python 3.12.x`.
    - Проверьте pip (менеджер пакетов):
      ```bash
      pip --version
      ```

#### 2.2.2. Настройка окружения
- **Виртуальное окружение**:
    - Создайте для изоляции зависимостей:
      ```bash
      python -m venv venv
      ```
    - Активируйте:
        - Windows: `venv\Scripts\activate`
        - macOS/Linux: `source venv/bin/activate`
    - Деактивируйте: `deactivate`
- **Применение в AQA**: Виртуальные окружения помогают изолировать библиотеки (например, Selenium) для разных проектов.

#### 2.2.3. Проблемы и решения
- **Ошибка "python не является внутренней командой"**: Убедитесь, что Python добавлен в PATH.
- **Конфликт версий**: Удалите старые версии Python или используйте `py -3` (Windows).

**Связь с другими темами**: Установка Python аналогична настройке Selenium WebDriver (например, WebDriverManager для драйверов).

---

### 2.3. Установка PyCharm

#### 2.3.1. Почему PyCharm?
- **Описание**: PyCharm — мощная IDE от JetBrains для Python, с поддержкой автодополнения, отладки, интеграции с Git и тестирования.
- **Преимущества в AQA**:
    - Поддержка pytest и Selenium.
    - Удобная отладка тестов.
    - Интеграция с виртуальными окружениями.
- **Альтернативы**: VS Code, Jupyter Notebook, IDLE (но PyCharm предпочтительнее для AQA).

#### 2.3.2. Шаги установки
1. **Скачивание**:
    - Перейдите на [jetbrains.com/pycharm](https://www.jetbrains.com/pycharm/download/).
    - Выберите **Community Edition** (бесплатная) или **Professional Edition** (платная, для продвинутых функций).
2. **Установка**:
    - Запустите установщик.
    - Windows: Выберите путь (например, `C:\Program Files\JetBrains\PyCharm`).
    - macOS/Linux: Следуйте инструкциям установщика.
3. **Настройка**:
    - Запустите PyCharm.
    - Создайте новый проект: File → New Project.
    - Укажите интерпретатор Python (выберите установленный Python или создайте виртуальное окружение).
    - Установите плагины (например, pytest, Git).
4. **Проверка**:
    - Создайте файл `hello.py` (см. ниже).
    - Запустите: ПКМ → Run 'hello'.

#### 2.3.3. Проблемы и решения
- **PyCharm не видит Python**: Укажите интерпретатор вручную (File → Settings → Project → Python Interpreter).
- **Медленный запуск**: Убедитесь, что выбрана Community Edition для меньшей нагрузки.

**Связь с другими темами**: PyCharm аналогична IntelliJ IDEA для Java, используется для отладки тестов и локаторов.

---

### 2.4. Первая программа: Hello World

#### 2.4.1. Описание
- Программа "Hello World" — традиционный пример для знакомства с языком.
- Цель: Научиться писать и запускать код, понимать структуру Python-программы.

#### 2.4.2. Код
```python
# hello.py
print("Hello, World!")
```

#### 2.4.3. Запуск
- **В PyCharm**:
    1. Создайте файл `hello.py` (ПКМ на проекте → New → Python File).
    2. Вставьте код выше.
    3. ПКМ на файле → Run 'hello'.
    4. Ожидаемый вывод в консоли: `Hello, World!`.
- **В терминале**:
  ```bash
  python hello.py
  ```

#### 2.4.4. Применение в AQA
- Простая программа может быть расширена до теста:
  ```python
  # test_hello.py
  def test_hello_world():
      expected = "Hello, World!"
      actual = "Hello, World!"
      assert expected == actual, "Test failed!"
  ```
- Запустите с pytest:
  ```bash
  pytest test_hello.py
  ```

**Связь с другими темами**: "Hello World" — это аналог теста в JUnit, где проверяется простое условие (assert).

---

## 3. Практические примеры в контексте AQA

### 3.1. Простой Selenium-тест
После освоения "Hello World" можно написать базовый тест с Selenium:
```python
# test_login.py
from selenium import webdriver
from selenium.webdriver.common.by import By

driver = webdriver.Chrome()
driver.get("https://example.com")
driver.find_element(By.ID, "username").send_keys("John")
driver.quit()
```

### 3.2. Интеграция с pytest
```python
# test_example.py
import pytest

def test_hello():
    assert "Hello, World!" == "Hello, World!", "Strings do not match"
```

Запуск:
```bash
pytest test_example.py
```

### 3.3. Отладка в PyCharm
- Установите точку останова (breakpoint) в `hello.py` (клик на полях слева от строки).
- Запустите в режиме Debug (ПКМ → Debug 'hello').
- Проверьте переменные в окне Debug.

**Связь с другими темами**: Отладка в PyCharm аналогична отладке локаторов в IntelliJ IDEA.

---

## 4. Связь с другими темами

- **Локаторы и селекторы**:
    - Python с Selenium использует локаторы (`By.ID`, `By.XPATH`), аналогично поиску данных в SQL:
      ```python
      driver.find_element(By.XPATH, "//input[@id='username']").send_keys("John")
      ```

- **JSON Wire Protocol и RESTful Web Service**:
    - Selenium в Python отправляет RESTful-запросы к драйверу браузера, что похоже на отправку SQL-запросов к серверу БД.

- **Клиент-серверная модель**:
    - Python (клиент) взаимодействует с браузером или сервером БД через API (Selenium или pymysql).

- **Тестирование (JUnit-подобные подходы)**:
    - pytest в Python аналогичен JUnit:
      ```python
      @pytest.fixture
      def setup():
          driver = webdriver.Chrome()
          yield driver
          driver.quit()
  
      def test_login(setup):
          setup.get("https://example.com")
          assert setup.title == "Example Domain"
      ```

- **ООП**:
    - Page Object Model в Python:
      ```python
      class LoginPage:
          def __init__(self, driver):
              self.driver = driver
              self.input_user = (By.ID, "username")
  
          def enter_username(self, username):
              self.driver.find_element(*self.input_user).send_keys(username)
      ```

---

## 5. Преимущества и недостатки

### 5.1. Python
- **Преимущества**:
    - Простота синтаксиса: Идеально для начинающих QA.
    - Богатая экосистема: Selenium, pytest, requests.
    - Быстрая разработка тестов.
- **Недостатки**:
    - Интерпретируемый язык: Медленнее для вычислений.
    - Требует строгого управления зависимостями.

### 5.2. PyCharm
- **Преимущества**:
    - Удобная отладка и автодополнение.
    - Интеграция с pytest и Git.
- **Недостатки**:
    - Community Edition ограничена в функциях.
    - Требует ресурсов на слабых машинах.

---

## 6. Рекомендации

- **Установка Python**:
    - Используйте Python 3.12+ и добавьте в PATH.
    - Создавайте виртуальные окружения для каждого проекта:
      ```bash
      python -m venv venv
      ```
- **PyCharm**:
    - Используйте Community Edition для простоты.
    - Настройте интерпретатор и плагины (pytest, Git).
- **Hello World**:
    - Пишите простые программы для понимания синтаксиса.
    - Перейдите к тестам с pytest после освоения.
- **AQA**:
    - Начните с простых Selenium-тестов:
      ```python
      driver.get("https://example.com")
      driver.find_element(By.ID, "submit").click()
      ```
- **CI/CD**:
    - Интегрируйте тесты в GitHub Actions:
      ```yaml
      name: Run Tests
      on: [push]
      jobs:
        test:
          runs-on: ubuntu-latest
          steps:
            - uses: actions/checkout@v3
            - name: Set up Python
              uses: actions/setup-python@v4
              with:
                python-version: '3.12'
            - name: Install dependencies
              run: pip install pytest selenium
            - name: Run tests
              run: pytest
      ```

---

## 7. Источники
- [Python Official Documentation](https://docs.python.org/3/)
- [PyCharm Documentation](https://www.jetbrains.com/pycharm/documentation/)
- [Selenium Python Documentation](https://selenium-python.readthedocs.io/)
- [pytest Documentation](https://docs.pytest.org/)

---

[**&#x2190; Назад к оглавлению**](README.md)