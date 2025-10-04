# Интеграция Allure с CI/CD

Интеграция Allure с CI/CD обеспечивает автоматическую генерацию отчётов после запуска тестов, их публикацию в системах вроде Jenkins, GitLab или TeamCity, а также добавление ссылок на отчёты в pull request или issue. Это упрощает анализ результатов для команд. В статье рассматриваются настройки для популярных CI/CD-систем. Используется технологический стек: Java 17, Maven, SLF4J с Logback, JUnit 5, TestNG, Selenium WebDriver, RestAssured, Allure и Jenkins.

## Автоматическая генерация отчёта после прогона тестов

Allure генерирует отчёты автоматически после выполнения тестов через Maven-плагин или скрипты в CI/CD.

### Настройка Maven для Allure

Добавьте плагин `allure-maven` в `pom.xml`:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>io.qameta.allure</groupId>
            <artifactId>allure-maven</artifactId>
            <version>2.12.0</version>
            <configuration>
                <reportVersion>2.29.0</reportVersion>
                <resultsDirectory>${project.build.directory}/allure-results</resultsDirectory>
            </configuration>
        </plugin>
    </plugins>
</build>
```

**Команда для генерации**:

```bash
mvn clean test allure:report
```

- Тесты генерируют данные в `target/allure-results`.
- Команда `allure:report` создаёт отчёт в `target/site/allure-maven-plugin`.

### Локальная генерация

Для просмотра отчёта локально:

```bash
mvn allure:serve
```

Отчёт доступен по адресу `http://localhost:1234` с автоматической генерацией после тестов.

## Публикация отчёта в Jenkins, GitLab, TeamCity или через Allure Server

### Jenkins

Установите плагин Allure Jenkins. Настройте `Jenkinsfile` для автоматической публикации:

```groovy
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh 'mvn clean test'
            }
        }
        stage('Generate Allure Report') {
            steps {
                allure([
                    includeProperties: false,
                    jdk: '',
                    results: [[path: 'target/allure-results']]
                ])
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'target/allure-results/**, logs/*.log', allowEmptyArchive: true
        }
    }
}
```

- Отчёт публикуется в интерфейсе Jenkins после прогона тестов.
- Доступен по ссылке в билде (Build History > Allure Report).

### GitLab

Используйте GitLab CI/CD с Maven. В `.gitlab-ci.yml`:

```yaml
stages:
  - test
  - report

test:
  stage: test
  script:
    - mvn clean test
  artifacts:
    paths:
      - target/allure-results/
    expire_in: 1 week

report:
  stage: report
  script:
    - mvn allure:report
  artifacts:
    paths:
      - target/site/allure-maven-plugin/
    expire_in: 1 week
  dependencies:
    - test
```

- Отчёт доступен в GitLab CI/CD Job > Artifacts.
- Для публикации в Merge Requests используйте GitLab Pages или внешний Allure Server.

### TeamCity

В TeamCity настройте build step с Maven. Добавьте post-build script:

```bash
##teamcity[buildStat reportFolder='target/allure-results' reportType='allure']
```

Или используйте плагин Allure TeamCity для автоматической генерации и публикации отчёта в UI TeamCity.

### Allure Server

Allure Server — самостоятельный сервер для хранения и просмотра отчётов. Запустите Docker-контейнер:

```bash
docker run -it -p 5050:5050 frankescobar/allure-server
```

Загружайте результаты через API:

```bash
curl -X POST "http://localhost:5050/allure-docker-service/history/1" \
  -H "Content-Type: application/json" \
  -d '{"results": ["target/allure-results"]}'
```

- Отчёты доступны по `http://localhost:5050`.
- Интегрируйте с CI/CD через скрипты для автоматической загрузки.

## Ссылки на отчёты в pull request или issue

Автоматическое добавление ссылок на отчёты в PR или issue упрощает коммуникацию.

### Jenkins

В `Jenkinsfile` добавьте пост-обработку для комментариев в GitHub/Jira:

```groovy
post {
    success {
        script {
            env.ALLURE_URL = env.BUILD_URL + "allure/"
            sh """
                curl -X POST \
                  -H "Authorization: token \${GITHUB_TOKEN}" \
                  -d "{\"body\": \"Allure Report: ${env.ALLURE_URL}\"}" \
                  https://api.github.com/repos/\${GIT_REPO}/issues/\${PULL_REQUEST_NUMBER}/comments
            """
        }
    }
}
```

- Требует плагина GitHub Integration и переменных `GITHUB_TOKEN`, `GIT_REPO`, `PULL_REQUEST_NUMBER`.

### GitLab

В `.gitlab-ci.yml` используйте API для комментариев в MR:

```yaml
report:
  stage: report
  script:
    - mvn allure:report
    - REPORT_URL="${CI_PROJECT_URL}/-/jobs/${CI_JOB_ID}/artifacts"
    - curl --request POST --header "PRIVATE-TOKEN: ${GITLAB_TOKEN}" \
      --data "body=Allure Report: ${REPORT_URL}" \
      "${CI_MERGE_REQUEST_PROJECT_URL}/merge_requests/${CI_MERGE_REQUEST_IID}/notes"
  artifacts:
    paths:
      - target/site/allure-maven-plugin/
  when: on_success
```

- Требует переменной `GITLAB_TOKEN` в настройках проекта.

### TeamCity

В TeamCity используйте build feature "GitHub Issues" или скрипт:

```bash
##teamcity[publishIssueComment issueId='PR-123' text='Allure Report: %teamcity.build.checkoutDir%/target/site/allure-maven-plugin']
```

- Ссылка на отчёт добавляется автоматически в комментарий к issue/PR.

### Общий подход для Allure Server

В скрипте CI/CD добавьте комментарий с ссылкой на Allure Server:

```bash
REPORT_ID=$(curl -X POST "http://allure-server:5050/allure-docker-service/history/1" -d '{"results": ["target/allure-results"]}' | jq -r '.id')
echo "Allure Report: http://allure-server:5050/history/1/${REPORT_ID}" | curl -X POST -H "Authorization: token ${GITHUB_TOKEN}" -d @- https://api.github.com/repos/${GIT_REPO}/issues/${PULL_REQUEST_NUMBER}/comments
```

## Лучшие практики и отладка

### Автоматическая генерация

- Запускайте `mvn clean test allure:report` в CI/CD для полной генерации.
- Храните `target/allure-results` как артефакты для повторной генерации.

### Публикация отчётов

- **Jenkins**: Используйте плагин Allure для встроенного просмотра.
- **GitLab/TeamCity**: Архивируйте отчёты как artifacts, генерируйте по требованию.
- **Allure Server**: Подходит для централизованного хранения, интегрируйте через Docker в CI/CD.

### Ссылки в PR/issue

- Автоматизируйте комментарии через API (GitHub, GitLab).
- Указывайте прямые ссылки на отчёты для быстрого доступа.
- Тестируйте скрипты локально перед CI/CD.

### Отладка

- В IntelliJ IDEA запускайте `mvn allure:serve` для локальной проверки.
- Проверяйте артефакты в CI/CD-логах на наличие `target/allure-results`.
- Для API-интеграции используйте `curl` для теста ссылок.

### Распространённые ошибки

- **Отсутствие артефактов**: Не архивируйте `target/allure-results`, отчёт не генерируется.
- **Неправильные переменные**: Ошибки в `GITHUB_TOKEN` или `CI_JOB_ID` блокируют комментарии.
- **Переполнение**: Слишком большие отчёты замедляют CI/CD; используйте ротацию.

## Дополнительно

### Пример полного Jenkinsfile

```groovy
pipeline {
    agent any
    environment {
        GITHUB_TOKEN = credentials('github-token')
    }
    stages {
        stage('Test') {
            steps {
                sh 'mvn clean test'
            }
        }
        stage('Allure Report') {
            steps {
                allure([
                    includeProperties: false,
                    jdk: '',
                    results: [[path: 'target/allure-results']]
                ])
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'target/allure-results/**, logs/*.log', allowEmptyArchive: true
        }
        success {
            script {
                env.ALLURE_URL = env.BUILD_URL + "allure/"
                sh """
                    curl -X POST \
                      -H "Authorization: token ${GITHUB_TOKEN}" \
                      -d '{"body": "Allure Report: ${ALLURE_URL}"}' \
                      https://api.github.com/repos/${GIT_REPO}/issues/${PULL_REQUEST_NUMBER}/comments
                """
            }
        }
    }
}
```

### Использование Testcontainers для Allure Server

```java
import io.qameta.allure.Epic;
import io.qameta.allure.Feature;
import io.qameta.allure.Owner;
import io.qameta.allure.Severity;
import io.qameta.allure.SeverityLevel;
import io.qameta.allure.Story;
import io.qameta.allure.Tag;
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testcontainers.containers.GenericContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

@Testcontainers
@Epic("Интеграционное тестирование")
@Feature("Allure Server")
@Story("Загрузка отчёта")
@Owner("devops_team")
@Tag("integration")
@Severity(SeverityLevel.MINOR)
public class AllureServerTest {
    private static final Logger logger = LoggerFactory.getLogger(AllureServerTest.class);

    @Container
    private static final GenericContainer<?> allureServer = new GenericContainer<>("frankescobar/allure-server")
            .withExposedPorts(5050);

    @Test
    void testAllureServerIntegration() {
        logger.info("Allure Server доступен на {}:{}", allureServer.getHost(), allureServer.getFirstMappedPort());
        // Симуляция загрузки отчёта
    }
}
```

## Источники

- [Allure Framework](https://allurereport.org/docs/)
- [Allure Jenkins Plugin](https://plugins.jenkins.io/allure-jenkins/)
- [GitLab CI/CD Documentation](https://docs.gitlab.com/ee/ci/)
- [TeamCity Allure Plugin](https://www.jetbrains.com/help/teamcity/allure.html)
- [Allure Server](https://github.com/allure-framework/allure2/tree/master/allure-server)
- [SLF4J User Manual](http://www.slf4j.org/manual.html)
- [Logback Documentation](https://logback.qos.ch/documentation.html)

---
[**← Назад к оглавлению**](../README.md)