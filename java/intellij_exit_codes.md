### Коды завершения консоли в IntelliJ IDEA

Коды завершения (exit codes) в IntelliJ IDEA — это числовые значения, возвращаемые Java Virtual Machine (JVM) или операционной системой после завершения выполнения программы. Они отображаются в консоли IntelliJ IDEA в виде сообщения, например, `Process finished with exit code 0`. Эти коды указывают на статус завершения процесса и помогают определить, успешно ли выполнена программа или произошла ошибка. Ниже представлено подробное объяснение кодов завершения с примерами и рекомендациями по их обработке в IntelliJ IDEA.

---

#### Общее понятие кодов завершения

- **Коды завершения** — это целые числа, возвращаемые процессом при его завершении. Они стандартизированы в UNIX-подобных системах (и частично в Windows) и используются для передачи информации о результате выполнения программы.
- В Java код завершения задаётся через метод `System.exit(int code)`, где `code` — произвольное целое число.
- В IntelliJ IDEA коды завершения отображаются в окне **Run** или **Debug** после завершения программы.

---

#### Основные коды завершения

1. **Код 0**:
    - **Значение**: Успешное завершение программы без ошибок.
    - **Описание**: Указывает, что программа выполнилась корректно, без исключений или проблем.
    - **Пример**:
      ```java
      public class Main {
          public static void main(String[] args) {
              System.out.println("Программа завершена успешно");
              System.exit(0); // Явное указание кода 0
          }
      }
      ```
        - **Вывод в консоли IntelliJ IDEA**: `Process finished with exit code 0`
    - **Контекст**: Это стандартный код для нормального завершения программы. Если программа завершается без вызова `System.exit()`, JVM автоматически возвращает 0, если не было необработанных исключений.

2. **Код 130 (SIGINT)**:
    - **Значение**: Программа прервана пользователем, обычно через нажатие `Ctrl+C` в консоли.
    - **Описание**: Код 130 соответствует сигналу `SIGINT` (сигнал прерывания) в UNIX-подобных системах. В IntelliJ IDEA это происходит, если пользователь вручную останавливает выполнение программы (например, нажимает красную кнопку "Stop" в окне **Run**).
    - **Пример**:
      ```java
      public class Main {
          public static void main(String[] args) {
              while (true) {
                  System.out.println("Бесконечный цикл");
                  try {
                      Thread.sleep(1000);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
          }
      }
      ```
        - Если остановить программу через кнопку **Stop** в IntelliJ IDEA, консоль покажет:
          ```
          Process finished with exit code 130 (interrupted by signal 2: SIGINT)
          ```
    - **Решение**:
        - Если код 130 нежелателен, добавьте обработку прерывания:
          ```java
          public class Main {
              public static void main(String[] args) {
                  Runtime.getRuntime().addShutdownHook(new Thread(() -> {
                      System.out.println("Программа завершена gracefully");
                      // Освобождение ресурсов
                  }));
                  while (true) {
                      System.out.println("Бесконечный цикл");
                      try {
                          Thread.sleep(1000);
                      } catch (InterruptedException e) {
                          System.out.println("Прерывание получено");
                          System.exit(0); // Завершение с кодом 0
                      }
                  }
              }
          }
          ```
        - Это позволяет выполнить "грациозное" завершение (graceful shutdown) и вернуть код 0 вместо 130.[](https://stackoverflow.com/questions/29887088/java-program-exit-with-code-130/71597904)

3. **Код 1**:
    - **Значение**: Общая ошибка выполнения программы.
    - **Описание**: Код 1 обычно указывает на необработанное исключение или проблему в программе (например, синтаксическую ошибку, отсутствие зависимости, неверную конфигурацию).
    - **Пример**:
      ```java
      public class Main {
          public static void main(String[] args) {
              throw new RuntimeException("Произошла ошибка");
          }
      }
      ```
        - **Вывод в консоли**: `Process finished with exit code 1`, с сообщением об исключении:
          ```
          Exception in thread "main" java.lang.RuntimeException: Произошла ошибка
          ```
    - **Решение**:
        - Проверьте логи в консоли IntelliJ IDEA для деталей ошибки.
        - Используйте отладчик (**Run → Debug**, Shift+F9) для поиска причины.
        - Для Spring Boot проектов код 1 часто связан с `BeanCreationException`. Проверьте конфигурацию бинов в `application.properties` или `application.yml`.[](https://stackoverflow.com/questions/55791675/what-does-process-finished-with-exit-code-1-mean-in-intellij-idea)

4. **Код -1**:
    - **Значение**: Низкоуровневая ошибка JVM или проблема с конфигурацией.
    - **Описание**: Может указывать на отсутствие необходимых библиотек, неправильную версию JDK или проблемы с настройками IntelliJ IDEA.
    - **Пример**:
      ```java
      public class Main {
          public static void main(String[] args) {
              // Код, вызывающий сбой JVM, например, из-за несовместимости библиотек
          }
      }
      ```
        - **Вывод в консоли**: `Process finished with exit code -1`
    - **Решение**:
        - Проверьте версию JDK: **File → Project Structure → SDKs**.
        - Обновите зависимости в `pom.xml` (Maven) или `build.gradle` (Gradle).
        - Очистите кэш: **File → Invalidate Caches / Restart**.

5. **Код 100 (Spark или другие фреймворки)**:
    - **Значение**: Специфическая ошибка, связанная с конфигурацией фреймворка, например, Spark.
    - **Описание**: Встречается при использовании библиотек, таких как Spark, из-за неправильной настройки зависимостей или ресурсов.
    - **Пример**:
      ```java
      import spark.Spark;
 
      public class Main {
          public static void main(String[] args) {
              Spark.get("/hello", (req, res) -> "Hello World");
          }
      }
      ```
        - Если зависимости Spark отсутствуют, консоль может показать:
          ```
          Process finished with exit code 100
          ```
    - **Решение**:
        - Проверьте зависимости в `pom.xml` или `build.gradle`.
        - Убедитесь, что версия Spark совместима с JDK.[](https://coderanch.com/t/689544/intellij-idea/ide/Process-finished-exit-code)

6. **Код -1073741819 (0xC0000005) или -1073740791 (0xC0000409)**:
    - **Значение**: Ошибка на уровне операционной системы (например, сбой драйвера или JVM).
    - **Описание**: Часто связана с графическими приложениями (Swing, JavaFX) из-за проблем с драйверами видеокарты (особенно NVIDIA).
    - **Пример**:
      ```java
      import javax.swing.JFrame;
 
      public class Main {
          public static void main(String[] args) {
              JFrame frame = new JFrame("Test");
              frame.setVisible(true);
          }
      }
      ```
        - **Вывод в консоли**: `Process finished with exit code -1073741819 (0xC0000005)`
    - **Решение**:
        - Обновите драйверы видеокарты или используйте стандартные драйверы Microsoft.
        - Запустите программу вне IDE: `java -jar my-app.jar`.
        - Проверьте настройки JVM: **Run → Edit Configurations → VM options** (например, `-Dprism.order=sw` для JavaFX).[](https://intellij-support.jetbrains.com/hc/en-us/community/posts/115000082624-Running-Swing-application-from-IDE-fails-with-exit-code-1073740791-but-starts-from-JAR-artifact)

7. **Код 2**:
    - **Значение**: Ошибка конфигурации или проблема с фреймворком (например, Play Framework).
    - **Описание**: Может указывать на проблемы с настройкой SBT, Maven или других инструментов сборки.
    - **Пример**:
        - При запуске Play Framework проекта в IntelliJ IDEA с некорректным `build.sbt`:
          ```
          Process finished with exit code 2
          ```
    - **Решение**:
        - Проверьте конфигурацию SBT: **File → Settings → Build, Execution, Deployment → Build Tools → SBT**.
        - Обновите зависимости: **SBT → Refresh SBT Project**.[](https://github.com/playframework/playframework/issues/11261)

---

#### Другие коды завершения

- **Коды 1–255**: В UNIX-подобных системах коды в диапазоне 0–255 часто стандартизированы. Например:
    - **128 + n**: Указывает на завершение процесса сигналом `n` (например, 130 = 128 + 2, где 2 — `SIGINT`).
    - **137**: Превышение лимита памяти (`SIGKILL`).
    - **143**: Завершение сигналом `SIGTERM` (например, при остановке процесса системой).
- **Произвольные коды**: В Java программа может возвращать любой целочисленный код через `System.exit()`. Например, разработчик может задать код 42 для обозначения конкретной ошибки:
  ```java
  System.exit(42); // Пользовательский код завершения
  ```

---

#### Работа с кодами завершения в IntelliJ IDEA

1. **Просмотр кодов завершения**:
    - Коды отображаются в окне **Run** или **Debug** в нижней части IDE.
    - Если программа завершилась с ненулевым кодом, проверьте логи в консоли для деталей (например, стек вызовов исключений).

2. **Отладка**:
    - Установите точки останова (Ctrl+F8) для анализа поведения программы перед завершением.
    - Используйте **Run → Debug** (Shift+F9) для пошагового выполнения.
    - Проверьте значения переменных и стек вызовов в окне **Debug**.

3. **Настройка конфигураций запуска**:
    - Перейдите в **Run → Edit Configurations**.
    - Настройте параметры JVM (`VM options`), аргументы программы (`Program arguments`) или переменные окружения (`Environment variables`) для устранения ошибок.

4. **Обработка кодов завершения**:
    - Для перехвата кода 130 (`SIGINT`) используйте `Runtime.getRuntime().addShutdownHook`:
      ```java
      Runtime.getRuntime().addShutdownHook(new Thread(() -> {
          System.out.println("Завершение программы...");
          // Освобождение ресурсов
      }));
      ```
    - Для преобразования кода 130 в 0 используйте `SecurityManager`:
      ```java
      class Converting130To0SecurityManager extends SecurityManager {
          @Override
          public void checkExit(int status) {
              if (status == 130) {
                  System.exit(0);
              } else {
                  super.checkExit(status);
              }
          }
          @Override
          public void checkPermission(java.security.Permission perm) {
              // Разрешить все действия
          }
      }
      public class Main {
          public static void main(String[] args) {
              System.setSecurityManager(new Converting130To0SecurityManager());
              while (true) {
                  System.out.println("Бесконечный цикл");
                  try {
                      Thread.sleep(1000);
                  } catch (InterruptedException e) {
                      System.exit(0);
                  }
              }
          }
      }
      ```
        - Это преобразует код 130 в 0 при прерывании.[](https://stackoverflow.com/questions/29887088/java-program-exit-with-code-130/71597904)

5. **Диагностика ошибок**:
    - Для ненулевых кодов (1, -1, 100, и т.д.) проверьте логи в окне **Run** или включите подробное логирование: **Run → Edit Configurations → VM options → -verbose:gc** (для диагностики проблем с памятью).
    - Используйте **Analyze → Inspect Code** для поиска потенциальных ошибок в коде.
    - Очистите кэш при странных ошибках: **File → Invalidate Caches / Restart**.

---

#### Рекомендации

- **Проверяйте логи**: Всегда анализируйте сообщения в консоли IntelliJ IDEA для понимания причины ненулевого кода завершения.
- **Настройте JDK**: Убедитесь, что версия JDK соответствует проекту (**File → Project Structure → SDKs**).
- **Обновляйте зависимости**: Для Maven/Gradle обновляйте зависимости через **Maven → Reload Project** или **Gradle → Refresh Gradle Project**.
- **Используйте отладчик**: Для сложных ошибок (например, коды -1, -1073741819) используйте отладчик или запускайте программу вне IDE (`java -jar`).
- **Графические приложения**: Для Swing/JavaFX обновляйте драйверы видеокарты и проверяйте параметры JVM.
- **Документируйте коды**: Если используете пользовательские коды завершения, документируйте их в Javadoc или комментариях:
  ```java
  /**
   * Завершает программу с кодом:
   * 0 - успешное завершение
   * 1 - ошибка ввода
   * 2 - ошибка конфигурации
   */
  public static void main(String[] args) {
      System.exit(1); // Ошибка ввода
  }
  ```

---
[**&#x2190; Назад к оглавлению**](README.md)