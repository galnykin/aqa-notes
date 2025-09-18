### Методы класса `Scanner` для чтения данных из консоли

Класс `Scanner` в Java (пакет `java.util`) предоставляет удобный способ чтения данных из различных источников, включая стандартный поток ввода (`System.in`), файлы и строки. В контексте консольного ввода `Scanner` используется для получения данных, введённых пользователем.

---

#### Основные методы класса `Scanner`

1. **`next()`**:
    - **Описание**: Читает одно "слово" — последовательность символов до следующего пробельного символа (пробел, табуляция, перенос строки).
    - **Возвращаемый тип**: `String`.
    - **Особенности**:
        - Игнорирует начальные пробельные символы.
        - Читает до первого пробела, даже если строка содержит несколько слов.
    - **Пример**:
      ```java
      import java.util.Scanner;
 
      public class Main {
          public static void main(String[] args) {
              Scanner scanner = new Scanner(System.in);
              System.out.print("Введите слово: ");
              String word = scanner.next(); // Читает "Hello" из "Hello World"
              System.out.println("Вы ввели: " + word);
          }
      }
      ```
        - Ввод: `Hello World`
        - Вывод: `Вы ввели: Hello`

2. **`nextLine()`**:
    - **Описание**: Читает всю строку до символа переноса строки (`\n`).
    - **Возвращаемый тип**: `String`.
    - **Особенности**:
        - Читает все символы, включая пробелы, до конца строки.
        - Может вызвать проблему, если используется после методов, читающих отдельные токены (например, `nextInt()`), так как оставляет символ переноса строки в буфере.
    - **Пример**:
      ```java
      import java.util.Scanner;
 
      public class Main {
          public static void main(String[] args) {
              Scanner scanner = new Scanner(System.in);
              System.out.print("Введите строку: ");
              String line = scanner.nextLine(); // Читает "Hello World"
              System.out.println("Вы ввели: " + line);
          }
      }
      ```
        - Ввод: `Hello World`
        - Вывод: `Вы ввели: Hello World`

3. **`nextInt()`**:
    - **Описание**: Читает следующее целое число (тип `int`).
    - **Возвращаемый тип**: `int`.
    - **Особенности**:
        - Ожидает ввод числа в правильном формате, иначе выбросит `InputMismatchException`.
        - Игнорирует пробельные символы перед числом.
        - Оставляет символ переноса строки в буфере, что может повлиять на последующий вызов `nextLine()`.
    - **Пример**:
      ```java
      import java.util.Scanner;
 
      public class Main {
          public static void main(String[] args) {
              Scanner scanner = new Scanner(System.in);
              System.out.print("Введите число: ");
              int number = scanner.nextInt();
              System.out.println("Вы ввели: " + number);
          }
      }
      ```
        - Ввод: `42`
        - Вывод: `Вы ввели: 42`

4. **`nextDouble()`**:
    - **Описание**: Читает следующее число с плавающей точкой (тип `double`).
    - **Возвращаемый тип**: `double`.
    - **Особенности**:
        - Поддерживает форматы, такие как `3.14`, `-0.5`, `1e-10`.
        - Выбрасывает `InputMismatchException`, если ввод не является числом с плавающей точкой.
        - Зависит от локализации (например, в некоторых локалях ожидает запятую вместо точки).
    - **Пример**:
      ```java
      import java.util.Scanner;
 
      public class Main {
          public static void main(String[] args) {
              Scanner scanner = new Scanner(System.in);
              System.out.print("Введите число с плавающей точкой: ");
              double value = scanner.nextDouble();
              System.out.println("Вы ввели: " + value);
          }
      }
      ```
        - Ввод: `3.14`
        - Вывод: `Вы ввели: 3.14`

---

#### Дополнительные методы класса `Scanner`

Помимо упомянутых методов, `Scanner` предоставляет другие полезные методы для чтения данных:

- **`nextBoolean()`**:
    - Читает значение типа `boolean` (`true` или `false`).
    - Пример:
      ```java
      Scanner scanner = new Scanner(System.in);
      System.out.print("Введите true или false: ");
      boolean flag = scanner.nextBoolean();
      System.out.println("Вы ввели: " + flag);
      ```
        - Ввод: `true`
        - Вывод: `Вы ввели: true`

- **`nextFloat()`**:
    - Читает число с плавающей точкой типа `float`.
    - Пример:
      ```java
      Scanner scanner = new Scanner(System.in);
      System.out.print("Введите число с плавающей точкой: ");
      float value = scanner.nextFloat();
      System.out.println("Вы ввели: " + value);
      ```
        - Ввод: `2.718`
        - Вывод: `Вы ввели: 2.718`

- **`nextLong()`**:
    - Читает целое число типа `long`.
    - Пример:
      ```java
      Scanner scanner = new Scanner(System.in);
      System.out.print("Введите большое число: ");
      long number = scanner.nextLong();
      System.out.println("Вы ввели: " + number);
      ```
        - Ввод: `1234567890`
        - Вывод: `Вы ввели: 1234567890`

- **`nextByte()`**, **`nextShort()`**:
    - Читают значения типов `byte` и `short` соответственно.
    - Используются реже из-за ограниченного диапазона этих типов.

- **`hasNext()`, `hasNextInt()`, `hasNextDouble()`, и т.д.**:
    - Проверяют, есть ли в потоке ввода следующий токен указанного типа.
    - Возвращают `true`, если токен доступен, и `false` в противном случае.
    - Пример:
      ```java
      Scanner scanner = new Scanner(System.in);
      System.out.print("Введите число или текст: ");
      if (scanner.hasNextInt()) {
          int number = scanner.nextInt();
          System.out.println("Число: " + number);
      } else {
          String text = scanner.next();
          System.out.println("Текст: " + text);
      }
      ```

- **`useDelimiter(String pattern)`**:
    - Задаёт разделитель для токенов (по умолчанию — пробельные символы).
    - Пример:
      ```java
      Scanner scanner = new Scanner("a,b,c").useDelimiter(",");
      while (scanner.hasNext()) {
          System.out.println(scanner.next()); // Вывод: a, b, c
      }
      ```

---

#### Проблемы и особенности использования

1. **Проблема с `nextLine()` после `nextInt()` или других методов**:
    - После вызова `nextInt()`, `nextDouble()` и подобных методов в буфере остаётся символ переноса строки (`\n`), который `nextLine()` может случайно прочитать.
    - **Решение**: Добавьте `scanner.nextLine()` для очистки буфера перед вызовом `nextLine()`:
      ```java
      Scanner scanner = new Scanner(System.in);
      System.out.print("Введите число: ");
      int number = scanner.nextInt();
      scanner.nextLine(); // Очистка буфера
      System.out.print("Введите строку: ");
      String line = scanner.nextLine();
      System.out.println("Число: " + number + ", строка: " + line);
      ```

2. **Обработка исключений**:
    - Методы, такие как `nextInt()` или `nextDouble()`, выбрасывают `InputMismatchException`, если ввод не соответствует ожидаемому типу.
    - Используйте `try-catch` или `hasNext...()` для проверки:
      ```java
      Scanner scanner = new Scanner(System.in);
      System.out.print("Введите число: ");
      try {
          int number = scanner.nextInt();
          System.out.println("Число: " + number);
      } catch (java.util.InputMismatchException e) {
          System.out.println("Ошибка: введите целое число");
      }
      ```

3. **Закрытие `Scanner`**:
    - Если `Scanner` создан для `System.in`, его закрытие (`scanner.close()`) закроет стандартный поток ввода, что может помешать дальнейшему вводу.
    - Рекомендация: Избегайте закрытия `Scanner` для `System.in` или используйте `try-with-resources` для других источников:
      ```java
      try (Scanner scanner = new Scanner(new File("input.txt"))) {
          String line = scanner.nextLine();
          System.out.println(line);
      } catch (Exception e) {
          System.out.println("Ошибка: " + e.getMessage());
      }
      ```

4. **Локализация**:
    - `nextDouble()` зависит от локализации (например, ожидает `3.14` в США, но `3,14` в некоторых странах Европы).
    - Установите локализацию: `scanner.useLocale(Locale.US);`.

---

#### Использование в IntelliJ IDEA

- **Автодополнение**:
    - IntelliJ IDEA предлагает методы `Scanner` при вводе (Ctrl+Space).
    - Например, начните писать `scanner.n` → выберите `nextLine()` или `nextInt()`.

- **Отладка ввода**:
    - Установите точку останова (Ctrl+F8) на строке с `scanner.next...()` для проверки введённых данных.
    - Используйте окно **Debug** для просмотра значений переменных.

- **Консоль**:
    - При запуске программы (**Run → Run**, Shift+F10) консоль IntelliJ IDEA позволяет вводить данные через `Scanner`.
    - Для тестирования ввода используйте **Run → Edit Configurations → Program arguments** или перенаправьте ввод из файла:
      ```java
      Scanner scanner = new Scanner(new File("input.txt"));
      ```

- **Анализ кода**:
    - IntelliJ IDEA предупреждает о потенциальных `InputMismatchException` или незакрытых `Scanner`.
    - Используйте **Analyze → Inspect Code** для проверки корректности использования.

---

#### Примеры практического использования

1. **Смешанный ввод**:
   ```java
   import java.util.Scanner;

   public class Main {
       public static void main(String[] args) {
           Scanner scanner = new Scanner(System.in);
           System.out.print("Введите имя: ");
           String name = scanner.nextLine();
           System.out.print("Введите возраст: ");
           int age = scanner.nextInt();
           System.out.print("Введите рост (в метрах): ");
           double height = scanner.nextDouble();
           System.out.printf("Имя: %s, Возраст: %d, Рост: %.2f м%n", name, age, height);
       }
   }
   ```
    - Ввод: `Alice`, `25`, `1.65`
    - Вывод: `Имя: Alice, Возраст: 25, Рост: 1.65 м`

2. **Циклический ввод с проверкой**:
   ```java
   import java.util.Scanner;

   public class Main {
       public static void main(String[] args) {
           Scanner scanner = new Scanner(System.in);
           System.out.println("Введите числа (или 'stop' для завершения):");
           while (scanner.hasNext()) {
               if (scanner.hasNextInt()) {
                   int number = scanner.nextInt();
                   System.out.println("Число: " + number);
               } else {
                   String input = scanner.next();
                   if (input.equals("stop")) break;
                   System.out.println("Не число: " + input);
               }
           }
       }
   }
   ```
    - Ввод: `10`, `abc`, `20`, `stop`
    - Вывод: `Число: 10`, `Не число: abc`, `Число: 20`

---

#### Рекомендации

- Используйте `hasNext...()` для проверки типа ввода перед вызовом методов, чтобы избежать `InputMismatchException`.
- Очищайте буфер с помощью `scanner.nextLine()` после методов, читающих токены (`nextInt()`, `nextDouble()`), если планируется использовать `nextLine()`.
- Избегайте закрытия `Scanner` для `System.in` в консольных приложениях.
- Устанавливайте локализацию (`useLocale`) для корректного чтения чисел с плавающей точкой в разных регионах.
- В IntelliJ IDEA настраивайте автодополнение и анализ кода для упрощения работы с `Scanner`.
- Тестируйте ввод в консоли IDE или используйте файлы для автоматизации ввода при отладке.

---

[**&#x2190; Назад к оглавлению**](README.md)