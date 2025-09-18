### Подробное руководство по занятию 2: Переменные и базовые типы данных в Python

Это руководство посвящено второй лекции курса по Python, которая фокусируется на переменных, их типах и использовании функции `print()`. Мы разберём каждую тему подробно, с акцентом на применение в автоматизации тестирования (AQA), где Python используется для написания тестов (например, с Selenium и pytest).

---

## 1. Занятие 2: Переменные, базовые типы данных

**Цель занятия**: Познакомить с концепцией переменных, основными типами данных в Python и расширенным использованием функции `print()`. Это базовые знания, необходимые для написания тестов, обработки данных и взаимодействия с веб-элементами в AQA.

**План занятия**:
1. Объяснение переменных и их роли в программировании.
2. Изучение базовых типов данных.
3. Практика с функцией `print()` для вывода данных.
4. Примеры в контексте AQA (например, работа с локаторами и результатами тестов).

**Связь с AQA**: Переменные и типы данных используются для хранения локаторов, результатов запросов, тестовых данных и взаимодействия с UI (например, ввод текста в Selenium).

---

## 2. Тема 2: Переменные

### 2.1. Что такое переменная?

- **Описание**: Переменная — это именованное место в памяти, которое хранит данные. В Python переменные создаются автоматически при присваивании значения.
- **Характеристики**:
    - **Имя**: Уникальное, должно следовать правилам (буквы, цифры, `_`, без пробелов, не начинается с цифры, не совпадает с зарезервированными словами).
    - **Значение**: Данные, которые хранит переменная.
    - **Динамическая типизация**: Тип переменной определяется автоматически и может меняться.
- **Пример**:
  ```python
  username = "John"  # Переменная username хранит строку
  age = 30           # Переменная age хранит целое число
  ```
- **Применение в AQA**:
    - Хранение локаторов:
      ```python
      login_button = (By.ID, "submit")  # Переменная для локатора
      driver.find_element(*login_button).click()
      ```
    - Хранение тестовых данных:
      ```python
      test_user = "testuser@example.com"
      driver.find_element(By.ID, "email").send_keys(test_user)
      ```

- **Связь с другими темами**:
    - **Локаторы**: Переменные хранят локаторы (например, `By.XPATH`), как в Page Object Model.
    - **Тестирование**: Переменные используются в assertions для сравнения ожидаемого и фактического результата:
      ```python
      expected_text = "Welcome"
      assert driver.find_element(By.ID, "message").get_text() == expected_text
      ```

---

### 2.2. Типы переменных (базовые типы данных)

Python поддерживает несколько базовых типов данных, которые используются в AQA для обработки входных данных, результатов и конфигураций.

#### 2.2.1. Основные типы данных
1. **Целые числа (`int`)**:
    - Хранят целые числа (без дробной части).
    - Пример:
      ```python
      age = 25
      user_id = 1001
      ```
    - **AQA**: Используется для хранения ID записей в БД или индексов элементов:
      ```python
      user_id = 1001
      query = f"SELECT * FROM users WHERE id = {user_id}"
      ```

2. **Числа с плавающей точкой (`float`)**:
    - Хранят дробные числа.
    - Пример:
      ```python
      price = 19.99
      rating = 4.5
      ```
    - **AQA**: Для проверки цен, рейтингов в e-commerce тестах:
      ```python
      price = 19.99
      assert driver.find_element(By.ID, "price").get_text() == str(price)
      ```

3. **Строки (`str`)**:
    - Хранят текст, включая Unicode.
    - Пример:
      ```python
      username = "John"
      error_message = "Invalid credentials"
      ```
    - **AQA**: Для ввода текста и проверки сообщений:
      ```python
      driver.find_element(By.ID, "username").send_keys("John")
      assert driver.find_element(By.ID, "error").get_text() == error_message
      ```

4. **Булевы значения (`bool`)**:
    - Хранят `True` или `False`.
    - Пример:
      ```python
      is_logged_in = True
      is_visible = False
      ```
    - **AQA**: Для проверки состояния элементов:
      ```python
      is_visible = driver.find_element(By.ID, "submit").is_displayed()
      assert is_visible
      ```

5. **Списки (`list`)**:
    - Упорядоченная изменяемая коллекция элементов.
    - Пример:
      ```python
      users = ["John", "Alice", "Bob"]
      ```
    - **AQA**: Для хранения тестовых данных:
      ```python
      users = ["user1", "user2"]
      for user in users:
          driver.find_element(By.ID, "username").send_keys(user)
          driver.find_element(By.ID, "submit").click()
      ```

6. **Кортежи (`tuple`)**:
    - Упорядоченная неизменяемая коллекция.
    - Пример:
      ```python
      locator = (By.ID, "submit")  # Используется в Selenium
      ```
    - **AQA**: Для хранения локаторов в Page Object Model:
      ```python
      login_button = (By.ID, "submit")
      driver.find_element(*login_button).click()
      ```

7. **Словари (`dict`)**:
    - Коллекция пар "ключ-значение".
    - Пример:
      ```python
      user_data = {"username": "John", "password": "pass123"}
      ```
    - **AQA**: Для хранения тестовых данных:
      ```python
      user_data = {"username": "John", "password": "pass123"}
      driver.find_element(By.ID, "username").send_keys(user_data["username"])
      ```

8. **Множества (`set`)**:
    - Неупорядоченная коллекция уникальных элементов.
    - Пример:
      ```python
      unique_ids = {1, 2, 3}
      ```
    - **AQA**: Для проверки уникальности данных:
      ```python
      user_ids = {row["id"] for row in database_query}
      assert len(user_ids) == len(database_query)  # Проверка уникальности
      ```

9. **NoneType (`None`)**:
    - Обозначает отсутствие значения.
    - Пример:
      ```python
      result = None
      ```
    - **AQA**: Для проверки отсутствия элемента:
      ```python
      result = driver.find_elements(By.ID, "nonexistent")
      assert result == [], "Элемент не должен существовать"
      ```

#### 2.2.2. Проверка типа
- Используйте функцию `type()`:
  ```python
  print(type(42))  # <class 'int'>
  print(type("Hello"))  # <class 'str'>
  ```

#### 2.2.3. Применение в AQA
- **Локаторы**: Кортежи для хранения `By` и значения:
  ```python
  submit_button = (By.ID, "submit")
  ```
- **Тестовые данные**: Словари для структурированных данных:
  ```python
  test_data = {"username": "John", "expected": "Welcome"}
  ```
- **Результаты**: Строки и списки для валидации:
  ```python
  errors = [driver.find_element(By.CLASS_NAME, "error").get_text()]
  assert errors == ["Invalid credentials"]
  ```

**Связь с другими темами**:
- **Локаторы**: Переменные типа `tuple` используются для локаторов в Selenium.
- **Тестирование**: Переменные хранят ожидаемые и фактические значения для assertions.
- **ООП**: Переменные инкапсулируются в классах Page Object Model.

---

### 2.3. Подробнее о функции `print()`

#### 2.3.1. Описание
- **Функция `print()`**: Выводит данные в консоль. В AQA используется для отладки тестов, логирования и проверки промежуточных результатов.
- **Синтаксис**:
  ```python
  print(*objects, sep=' ', end='\n', file=sys.stdout, flush=False)
  ```
    - `objects`: Объекты для вывода (строки, числа и т.д.).
    - `sep`: Разделитель между объектами (по умолчанию пробел).
    - `end`: Символ в конце строки (по умолчанию `\n`).
    - `file`: Куда выводить (по умолчанию консоль).
    - `flush`: Принудительный сброс буфера.

#### 2.3.2. Примеры
- **Базовый вывод**:
  ```python
  print("Hello, World!")  # Hello, World!
  ```
- **Множественные объекты**:
  ```python
  name = "John"
  age = 30
  print(name, age)  # John 30
  ```
- **Кастомизация разделителя**:
  ```python
  print("John", "Doe", sep="-")  # John-Doe
  ```
- **Кастомизация конца строки**:
  ```python
  print("Hello", end="!")
  print("World")  # Hello!World
  ```
- **Форматированный вывод**:
    - f-строки (Python 3.6+):
      ```python
      username = "John"
      print(f"User: {username}")  # User: John
      ```
    - `str.format()`:
      ```python
      print("User: {}".format(username))  # User: John
      ```

#### 2.3.3. Применение в AQA
- **Отладка тестов**:
  ```python
  element = driver.find_element(By.ID, "username")
  print(f"Элемент найден: {element.is_displayed()}")  # True/False
  ```
- **Логирование результатов**:
  ```python
  error = driver.find_element(By.ID, "error").get_text()
  print(f"Ошибка: {error}")
  ```
- **Проверка данных**:
  ```python
  users = ["John", "Alice"]
  print("Тестируемые пользователи:", users, sep="\n")
  ```

#### 2.3.4. Расширенное использование
- **Вывод в файл**:
  ```python
  with open("log.txt", "w") as f:
      print("Test completed", file=f)
  ```
- **Отладка в pytest**:
  ```python
  def test_login():
      print("Запуск теста логина...")
      assert driver.find_element(By.ID, "message").get_text() == "Welcome"
  ```

**Связь с другими темами**:
- **Клиент-серверная модель**: `print()` — это способ логирования ответов сервера (например, текста элемента).
- **Тестирование**: `print()` помогает отлаживать тесты перед assertions.

---

## 3. Практические примеры в контексте AQA

### 3.1. Переменные и Selenium
Тест для ввода данных в форму:
```python
from selenium import webdriver
from selenium.webdriver.common.by import By

# Переменные
url = "https://example.com"
username = "John"
expected_message = "Welcome, John"

# Тест
driver = webdriver.Chrome()
driver.get(url)
driver.find_element(By.ID, "username").send_keys(username)
driver.find_element(By.ID, "submit").click()
actual_message = driver.find_element(By.ID, "message").get_text()
print(f"Ожидаемый текст: {expected_message}, Фактический: {actual_message}")
assert actual_message == expected_message
driver.quit()
```

### 3.2. Использование типов данных в pytest
Проверка списка пользователей:
```python
import pytest

def test_users():
    users = ["John", "Alice", "Bob"]  # Список
    expected_count = 3
    print(f"Количество пользователей: {len(users)}")
    assert len(users) == expected_count, "Неверное количество пользователей"
```

### 3.3. Форматированный вывод
Логирование тестовых данных:
```python
test_data = {"username": "John", "password": "pass123"}
print("Тестовые данные:", test_data, sep="\n")
driver.find_element(By.ID, "username").send_keys(test_data["username"])
driver.find_element(By.ID, "password").send_keys(test_data["password"])
```

---

## 4. Связь с другими темами

- **Локаторы и селекторы**:
    - Переменные хранят локаторы:
      ```python
      login_button = (By.CSS_SELECTOR, "[data-testid='submit']")
      driver.find_element(*login_button).click()
      ```
- **JSON Wire Protocol и RESTful Web Service**:
    - Переменные типа `str` используются для отправки данных в REST-запросах:
      ```python
      endpoint = "https://api.example.com/login"
      response = requests.post(endpoint, json={"username": "John"})
      print(response.json())
      ```
- **Клиент-серверная модель**:
    - Переменные хранят ответы сервера (например, текст элемента):
      ```python
      message = driver.find_element(By.ID, "message").get_text()
      print(f"Ответ сервера: {message}")
      ```
- **Тестирование (JUnit-подобные подходы)**:
    - Переменные в assertions:
      ```python
      expected = "Success"
      actual = driver.find_element(By.ID, "status").get_text()
      assert actual == expected
      ```
- **ООП**:
    - Переменные в Page Object Model:
      ```python
      class LoginPage:
          input_user = (By.ID, "username")
  
          def __init__(self, driver):
              self.driver = driver
  
          def enter_username(self, username):
              self.driver.find_element(*self.input_user).send_keys(username)
      ```

---

## 5. Преимущества и недостатки

### 5.1. Переменные
- **Преимущества**:
    - Упрощают хранение и повторное использование данных.
    - Динамическая типизация ускоряет разработку.
- **Недостатки**:
    - Ошибки из-за неправильного использования типов (например, сложение `str` и `int`).

### 5.2. Типы данных
- **Преимущества**:
    - Гибкость: Подходят для любых задач в AQA.
    - Простота работы со списками, словарями.
- **Недостатки**:
    - Требуется понимание типов для избежания ошибок.

### 5.3. `print()`
- **Преимущества**:
    - Простота отладки.
    - Гибкость форматирования (f-строки).
- **Недостатки**:
    - Чрезмерное использование может засорять консоль.

---

## 6. Рекомендации

- **Имена переменных**:
    - Используйте описательные имена: `username`, `login_button`, `error_message`.
    - Следуйте PEP 8 (например, snake_case: `test_data`).
- **Типы данных**:
    - Используйте `str` для текстов, `list` для коллекций, `dict` для тестовых данных.
    - Проверяйте типы с помощью `type()` при отладке.
- **print()**:
    - Используйте f-строки для читаемого вывода:
      ```python
      print(f"Тест для {username} завершён")
      ```
    - Логируйте в файл для CI/CD:
      ```python
      with open("test_log.txt", "a") as f:
          print(f"Результат: {result}", file=f)
      ```
- **AQA**:
    - Храните локаторы в переменных:
      ```python
      submit_button = (By.ID, "submit")
      ```
    - Используйте словари для тестовых данных:
      ```python
      test_data = {"username": "John", "password": "pass123"}
      ```
- **CI/CD**:
    - Интегрируйте тесты с pytest:
      ```bash
      pytest test_login.py
      ```

---

## 7. Источники
- [Python Official Documentation](https://docs.python.org/3/tutorial/introduction.html)
- [PEP 8: Style Guide for Python Code](https://pep8.org/)
- [Selenium Python Documentation](https://selenium-python.readthedocs.io/)
- [pytest Documentation](https://docs.pytest.org/)

---

[**&#x2190; Назад к оглавлению**](README.md)