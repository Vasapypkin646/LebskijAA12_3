# **Отчет по лабораторной работе №6: «GitLab CI/CD: Создание пайплайна для автоматизации сборки, тестирования и развертывания»**

## **Сведения о студенте**
**Дата:** 2026-10-28    
**Семестр:** 3 курс, 5 семестр  
**Группа:** Пин-б-о-24-1    
**Дисциплина:** Системы контроля версий     
**Студент:** Лебский Артём Александрович    

---

### **Структура готового проекта**
```
collab-project/
├── .git/                     # Каталог локальной базы данных репозитория Git
├── .gitlab-ci.yml            # Конфигурация пайплайна GitLab CI/CD
├── app.py                    # Исходный код веб-приложения (Flask)
├── requirements.txt          # Зависимости проекта
├── tests/                    # Каталог модульных тестов
│   └── test_app.py           # Тесты для pytest
├── CONTRIBUTORS.md           # Список участников проекта
├── MY_CONTRIBUTION.md        # Описание вклада
└── README.md                 # Документация проекта с описанием health check
```

### **Ссылка на репозиторий GitLab**
```
https://gitlab.com/Vasapypkin646/collab-project
```

---

### **1. ЦЕЛЬ РАБОТЫ**

> 1. Изучить основные концепции GitLab CI/CD: пайплайны, стадии, джобы, раннеры.
> 2. Освоить синтаксис файла `.gitlab-ci.yml` для описания пайплайнов.
> 3. Создать многостадийный пайплайн для автоматизации сборки, тестирования и развертывания приложения.
> 4. Настроить артефакты и кэширование для оптимизации времени выполнения.
> 5. Настроить правила выполнения джоб в зависимости от ветки и типа события (Merge Request pipelines).
> 6. Настроить окружения (staging/production) и проследить за процессом деплоя.

---

### **2. ЗАДАЧИ РАБОТЫ**

> 1. Подготовка структуры проекта и создание легковесного веб-приложения на Flask (`app.py`).
> 2. Написание автоматических тестов с использованием фреймворка `pytest`.
> 3. Настройка кэширования менеджера пакетов `pip` (`PIP_CACHE_DIR`).
> 4. Разработка конфигурации пайплайна `.gitlab-ci.yml` со стадиями `lint`, `build`, `test`, `deploy`.
> 5. Добавление джобы линтинга кода (`flake8`) для статического анализа качества кода.
> 6. Добавление джобы проверки уязвимостей зависимостей (`safety check`) с флагом `allow_failure: true`.
> 7. Настройка генерации и экспорта тестовых отчётов в формате JUnit (`report.xml`) через механизм артефактов GitLab.
> 8. Настройка окружения `staging` с ручным запуском развертывания (`when: manual`).
> 9. Создание ветки `feature/add-health-check` и оформление Merge Request для демонстрации работы Merge Request Pipelines.

---

### **3. ХОД ВЫПОЛНЕНИЯ РАБОТЫ**

#### **3.1. Создание микросервиса на Flask**

В корне репозитория создан файл веб-приложения `app.py`, предоставляющий корневой эндпоинт `/` и эндпоинт проверки жизнеспособности `/health`:

```python
#!/usr/bin/env python3
from flask import Flask, jsonify

app = Flask(__name__)


@app.route('/')
def home():
    return jsonify({"message": "Hello, CI/CD!", "version": "1.0.0"})


@app.route('/health')
def health():
    return jsonify({"status": "healthy"})


if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

В файле `requirements.txt` зафиксированы зависимости для работы приложения и проведения тестов:
```
flask==3.0.0
pytest==7.4.0
flake8==6.1.0
safety==2.3.5
```

---

#### **3.2. Разработка модульных тестов**

В каталоге `tests/` создан файл `test_app.py`, содержащий автотесты для проверки корректности статус-кодов и формата ответов JSON:

```python
import pytest
from app import app


@pytest.fixture
def client():
    return app.test_client()


def test_home(client):
    response = client.get('/')
    assert response.status_code == 200
    data = response.get_json()
    assert data['message'] == 'Hello, CI/CD!'


def test_health(client):
    response = client.get('/health')
    assert response.status_code == 200
    data = response.get_json()
    assert data['status'] == 'healthy'
```

---

#### **3.3. Разработка конфигурационного файла `.gitlab-ci.yml`**

Создан многостадийный пайплайн `.gitlab-ci.yml`, включающий стадии `lint`, `build`, `test` и `deploy`:

```yaml
# Предотвращение бесконечного цикла пайплайнов при создании тегов
workflow:
  rules:
    - if: $CI_COMMIT_TAG
      when: never
    - when: always

# Определение стадий пайплайна
stages:
  - lint
  - build
  - test
  - deploy

# Глобальные переменные окружения
variables:
  PIP_CACHE_DIR: "$CI_PROJECT_DIR/.cache/pip"

# Кэширование зависимостей для ускорения сборки
cache:
  paths:
    - .cache/pip

# --- Стадия: Линтинг ---
lint:
  stage: lint
  image: python:3.11-slim
  script:
    - echo "🔍 Проверка стиля кода..."
    - pip install flake8
    - flake8 app.py --max-line-length=120 --statistics
  rules:
    - if: $CI_PIPELINE_SOURCE == 'merge_request_event'
    - if: $CI_COMMIT_BRANCH == "main"

# --- Стадия: Сборка и установка зависимостей ---
build:
  stage: build
  image: python:3.11-slim
  script:
    - echo "📦 Установка зависимостей..."
    - pip install --cache-dir=$PIP_CACHE_DIR -r requirements.txt
    - echo "✅ Зависимости успешно установлены"
  artifacts:
    paths:
      - requirements.txt
    expire_in: 1 hour
  rules:
    - if: $CI_PIPELINE_SOURCE == 'merge_request_event'
    - if: $CI_COMMIT_BRANCH == "main"

# --- Стадия: Тестирование ---
test:
  stage: test
  image: python:3.11-slim
  script:
    - echo "🧪 Запуск модульных тестов..."
    - pip install -r requirements.txt
    - python -m pytest tests/ -v --junitxml=report.xml
  artifacts:
    when: always
    reports:
      junit: report.xml
    paths:
      - report.xml
    expire_in: 1 week
  rules:
    - if: $CI_PIPELINE_SOURCE == 'merge_request_event'
    - if: $CI_COMMIT_BRANCH == "main"

# --- Стадия: Проверка безопасности зависимостей ---
security_scan:
  stage: test
  image: python:3.11-slim
  script:
    - echo "🔒 Проверка безопасности зависимостей..."
    - pip install safety
    - safety check -r requirements.txt
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
  allow_failure: true

# --- Стадия: Деплой на Staging ---
deploy_staging:
  stage: deploy
  image: python:3.11-slim
  script:
    - echo "🚀 Развертывание на staging окружение..."
    - pip install -r requirements.txt
    - nohup python app.py > /tmp/flask-staging.log 2>&1 &
    - sleep 3
    - curl -sf http://localhost:5000/health
    - echo "✅ Staging deployment успешно проверен"
  environment:
    name: staging
    url: http://localhost:5000
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: manual
    - when: never
```

---

#### **3.4. Фиксация изменений в ветке `main`**

Файлы добавлены в индекс и зафиксированы коммитом:
```bash
git add app.py requirements.txt tests/ .gitlab-ci.yml
git commit -m "ci: add initial CI/CD pipeline and Flask application"
git push origin main
```

Вывод команды:
```
[main 62b7466] ci: add initial CI/CD pipeline and Flask application
 4 files changed, 143 insertions(+)
 create mode 100644 .gitlab-ci.yml
 create mode 100644 app.py
 create mode 100644 requirements.txt
 create mode 100644 tests/test_app.py
```
![](./img/img1.jpg)

---

#### **3.5. Создание ветки `feature/add-health-check` и Merge Request**

Для демонстрации работы пайплайнов для Merge Request создана отдельная функциональная ветка:
```bash
git checkout -b feature/add-health-check
# Добавление документации в README.md
git add README.md
git commit -m "docs: add health check description"
git push origin feature/add-health-check
```

Вывод команды:
```
Switched to a new branch 'feature/add-health-check'
[feature/add-health-check ed5928f] docs: add health check description
 1 file changed, 4 insertions(+)
```
![](./img/img2.jpg)

В веб-интерфейсе GitLab оформлен Merge Request: `feature/add-health-check` $\to$ `main`.
Благодаря директиве:
```yaml
rules:
  - if: $CI_PIPELINE_SOURCE == 'merge_request_event'
```
для открытого Merge Request автоматически запустился изолированный MR Pipeline со стадиями `lint`, `build` и `test` без запуска стадии `deploy`.

![](./img/img3.jpg)

---

#### **3.6. Выполнение пайплайна и ручной деплой на staging**

После слияния MR в основную ветку `main` запустился полный пайплайн:
1. **lint:** выполнена проверка `flake8`, ошибок форматирования кода не обнаружено.
2. **build:** установлен кэш зависимостей в директорию `.cache/pip`.
3. **test:** запущен `pytest`, сгенерирован артефакт `report.xml`, выполнено сканирование `safety`.
4. **deploy_staging:** джоба переведена в режим ожидания ручного подтверждения (`when: manual`). При нажатии кнопки Play в UI GitLab джоба успешно запустила сервис и проверила статус `/health`.

![](./img/img4.jpg)

---

### **4. ОТВЕТЫ НА КОНТРОЛЬНЫЕ ВОПРОСЫ**

#### **1. Что такое GitLab Runner и какие типы раннеров существуют?**
**GitLab Runner** — это изолированный агент/приложение, которое опрашивает сервер GitLab, забирает назначенные задачи (джобы) из очереди пайплайна и выполняет их в целевом окружении (Docker-контейнеры, виртуальные машины, Kubernetes или локальный shell).
Существуют следующие типы раннеров:
- **Shared runners (общие):** доступны для всех проектов на уровне инстанса GitLab;
- **Group runners (групповые):** привязаны к определённой группе проектов и доступны для всех репозиториев внутри группы;
- **Project/Specific runners (проектные):** назначены только конкретному проекту для выполнения специфичных задач.

#### **2. В чём разница между `stages` и `jobs` в GitLab CI/CD?**
- **Stages (стадии)** определяют логическую последовательность и фазы выполнения пайплайна (например, `build` $\to$ `test` $\to$ `deploy`). Стадии всегда выполняются строго последовательно. Следующая стадия начинается только тогда, когда все джобы предыдущей завершились успешно.
- **Jobs (джобы)** — это атомарные задачи, содержащие конкретные исполняемые скрипты (`script`). Все джобы, принадлежащие одной и той же стадии, выполняются **параллельно** (при наличии доступных свободных раннеров).

#### **3. Как настроить кэширование зависимостей в пайплайне?**
Кэширование настраивается с помощью блока `cache: paths:` на глобальном уровне или внутри конкретной джобы:
```yaml
variables:
  PIP_CACHE_DIR: "$CI_PROJECT_DIR/.cache/pip"

cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - .cache/pip
```
Кэш сохраняет скачанные пакеты между последовательными запусками пайплайнов, избавляя от повторной загрузки библиотек из интернета.

#### **4. Что такое артефакты и как они используются между джобами?**
**Артефакты (artifacts)** — это файлы или каталоги, генерируемые джобой в процессе выполнения (скомпилированные бинарники, отчеты тестов `report.xml`, архивы), которые загружаются на сервер GitLab. Они могут передаваться между зависимыми джобами последующих стадий через директиву `dependencies`, а также скачиваться пользователем через веб-интерфейс GitLab.

#### **5. В чём отличие Merge Request пайплайнов от обычных Branch пайплайнов?**
- **Branch pipeline** запускается при каждом пуше в любую ветку и выполняет стандартный набор задач.
- **Merge Request pipeline** инициируется событием создания или обновления Merge Request (`CI_PIPELINE_SOURCE == 'merge_request_event'`). Он позволяет запускать предварительные проверки, линтинг и тесты для верификации возможности слияния, исключая запуск задач релиза или развертывания на production.

#### **6. Как настроить ручной запуск джобы с помощью `when: manual`?**
В блок `rules` или напрямую в конфигурацию джобы добавляется параметр `when: manual`:
```yaml
deploy_prod:
  stage: deploy
  script:
    - echo "Deploying..."
  when: manual
```
Такая джоба не запускается автоматически, а ожидает нажатия кнопки оператором («Play») в веб-интерфейсе GitLab.

#### **7. Что такое `workflow: rules` и как его использовать для предотвращения бесконечных циклов?**
`workflow: rules` определяет правила на уровне всего пайплайна (будет ли вообще создан пайплайн). Например, если джоба пайплайна в процессе работы создаёт Git-тег и пушит его обратно, это может вызвать запуск нового пайплайна и зацикливание. Для предотвращения используется директива:
```yaml
workflow:
  rules:
    - if: $CI_COMMIT_TAG
      when: never
    - when: always
```

#### **8. Как настроить переменные окружения в GitLab CI/CD?**
Переменные можно настраивать:
1. В файле `.gitlab-ci.yml` через секцию `variables:`;
2. В веб-интерфейсе GitLab: **Settings $\to$ CI/CD $\to$ Variables** (поддерживаются маскированные и защищённые переменные для безопасного хранения паролей и API-токенов);
3. Использовать предопределенные системные переменные GitLab (например, `CI_COMMIT_BRANCH`, `CI_PROJECT_NAME`, `CI_PIPELINE_ID`).

#### **9. Что такое `allow_failure` и когда его используют?**
Директива `allow_failure: true` указывает, что даже в случае ошибки скрипта в данной джобе пайплайн не прерывается и продолжает выполнение последующих стадий (джоба помечается оранжевым предупреждающим знаком). Используется для экспериментальных тестов, проверок безопасности (`safety check`) или некритичных линтеров.

#### **10. Как настроить автоматический запуск пайплайна по расписанию?**
В интерфейсе GitLab:
1. Перейти в меню **Build $\to$ Pipeline schedules** (или **Settings $\to$ CI/CD $\to$ Schedules**).
2. Нажать **New schedule**.
3. Указать описание задачи, целевую ветку (`main`) и расписание в формате cron (например, `0 2 * * *` для ночной сборки).
4. Сохранить расписание.

---

### **5. ИСПОЛЬЗУЕМЫЕ КОМАНДЫ**

```bash
# Клонирование репозитория
git clone https://gitlab.com/Vasapypkin646/collab-project.git
cd collab-project

# Создание файлов проекта и тестов
# (app.py, requirements.txt, tests/test_app.py, .gitlab-ci.yml)

# Добавление и фиксация изменений
git add app.py requirements.txt tests/ .gitlab-ci.yml
git commit -m "ci: add initial CI/CD pipeline and Flask application"
git push origin main

# Создание функциональной ветки для Merge Request
git checkout -b feature/add-health-check
git add README.md
git commit -m "docs: add health check description"
git push origin feature/add-health-check
```

---

### **6. ВЫВОДЫ**

В ходе выполнения лабораторной работы были освоены принципы проектирования и реализации CI/CD пайплайнов в экосистеме GitLab:
> 1. Разработано микросервисное веб-приложение на Flask с набором модульных тестов на базе `pytest`.
> 2. Создана многоэтапная конфигурация `.gitlab-ci.yml`, охватывающая стадии статического анализа (`flake8`), кэширования зависимостей, модульного тестирования с экспортом JUnit-отчётов и контролируемого развертывания на staging.
> 3. Настроена интеграция пайплайнов с процессом рецензирования кода через Merge Request Pipelines.
> 4. Освоены продвинутые возможности GitLab CI: кэширование, артефакты сборки, переменные окружения и условные правила выполнения (`rules`).

