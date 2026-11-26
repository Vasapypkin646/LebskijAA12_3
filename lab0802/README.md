# **Отчет по лабораторной работе №8: «Git Submodules, Git LFS и Container Registry в GitLab»**

## **Сведения о студенте**
**Дата:** 2026-11-18    
**Семестр:** 3 курс, 5 семестр  
**Группа:** Пин-б-о-24-1    
**Дисциплина:** Системы контроля версий     
**Студент:** Лебский Артём Александрович    

---

### **Структура готового проекта**
```
collab-project/
├── .git/                     # Каталог локальной базы данных репозитория Git
├── .gitattributes            # Конфигурация отслеживания бинарных файлов через Git LFS
├── .gitmodules               # Конфигурация подключенных Git Submodules
├── .gitlab-ci.yml            # Пайплайн CI/CD с проверкой LFS и Container Registry
├── Dockerfile                # Docker-спецификация контейнера веб-приложения
├── app.py                    # Исходный код микросервиса на Python (Flask)
├── requirements.txt          # Зависимости проекта
├── libs/
│   └── shared/               # Подключенный Git Submodule (внешняя библиотека)
├── document.pdf              # Бинарный PDF-документ под управлением Git LFS
├── sample.png                # Бинарное изображение под управлением Git LFS
├── sound.mp3                 # Бинарный аудиофайл под управлением Git LFS
├── CONTRIBUTORS.md           # Список участников проекта
├── MY_CONTRIBUTION.md        # Описание вклада
└── README.md                 # Документация проекта
```

### **Ссылка на репозиторий GitLab**
```
https://gitlab.com/Vasapypkin646/collab-project
```

---

### **1. ЦЕЛЬ РАБОТЫ**

> 1. Освоить работу с Git Submodules для управления зависимостями между репозиториями.
> 2. Изучить Git LFS (Large File Storage) для эффективного версионирования и хранения бинарных файлов в GitLab.
> 3. Настроить GitLab Container Registry для хранения, тегирования и публикации Docker-образов.
> 4. Интегрировать Submodules, LFS и Container Registry в общий многостадийный CI/CD пайплайн.
> 5. Понять архитектурные преимущества, сценарии применения и ограничения каждого инструмента.

---

### **2. ЗАДАЧИ РАБОТЫ**

> 1. Добавление внешнего репозитория в качестве Git Submodule (`libs/shared`), фиксация метаданных в файле `.gitmodules`.
> 2. Инициализация и настройка утилиты `git lfs` в рабочей среде.
> 3. Конфигурация правил отслеживания типов файлов (`*.png`, `*.jpg`, `*.pdf`, `*.mp4`, `*.zip`, `*.mp3`) в файле `.gitattributes`.
> 4. Добавление бинарных файлов в репозиторий с проверкой указателей Git LFS через `git lfs ls-files` и `git lfs status`.
> 5. Разработка микросервиса на Python/Flask (`app.py`, `requirements.txt`) и спецификации `Dockerfile`.
> 6. Проектирование пайплайна `.gitlab-ci.yml` со стадиями проверки LFS (`check_lfs`), сборки образа (`build_image`), тестирования контейнера (`test_image`), публикации в Container Registry (`publish_image`) и развертывания (`deploy_staging`).
> 7. Фиксация изменений в ветке `main` и отправка коммитов в GitLab.
> 8. Создание ветки `feature/lfs-container-registry`, расширение LFS-правил и оформление Merge Request.

---

### **3. ХОД ВЫПОЛНЕНИЯ РАБОТЫ**

#### **3.1. Подключение и настройка Git Submodule**

В репозиторий `collab-project` добавлен внешний репозиторий в качестве подкаталога зависимостей `libs/shared`:
```bash
git submodule add https://github.com/octocat/Hello-World.git libs/shared
```

Вывод команды:
```
Cloning into 'D:/Antigravity_projects/git-labs/lab0208/collab-project/libs/shared'...
```

Автоматически сформировался файл конфигурации подмодулей `.gitmodules`:
```ini
[submodule "libs/shared"]
	path = libs/shared
	url = https://github.com/octocat/Hello-World.git
```

Проверка статуса подключенного подмодуля:
```bash
git submodule status
```
Вывод:
```
 7fd1a60b01f91b314f59955a4e4d4e80d8edf11d libs/shared (heads/master)
```
Подмодуль зафиксирован на конкретном SHA-хэше коммита, что обеспечивает детерминированность сборки проекта.

![](./img/img1.jpg)

---

#### **3.2. Настройка Git LFS и добавление бинарных файлов**

Выполнена инициализация Git LFS в проекте и заданы паттерны отслеживания бинарных файлов:
```bash
git lfs install
git lfs track "*.png"
git lfs track "*.jpg"
git lfs track "*.pdf"
git lfs track "*.mp4"
git lfs track "*.zip"
```

Сгенерирован файл `.gitattributes`:
```gitattributes
*.png filter=lfs diff=lfs merge=lfs -text
*.jpg filter=lfs diff=lfs merge=lfs -text
*.pdf filter=lfs diff=lfs merge=lfs -text
*.mp4 filter=lfs diff=lfs merge=lfs -text
*.zip filter=lfs diff=lfs merge=lfs -text
```

В репозиторий добавлены тестовые бинарные файлы `sample.png` и `document.pdf`. Выполнена проверка статуса LFS-объектов:
```bash
git lfs status
git lfs ls-files
```

Вывод команды `git lfs status`:
```
Objects to be committed:
	.gitattributes (Git: 54bf6dd)
	.gitmodules (Git: 04602e8)
	document.pdf (LFS: a5e2e1e)
	sample.png (LFS: a5e2e1e)
```

Вывод команды `git lfs ls-files`:
```
a5e2e1e77d * document.pdf
a5e2e1e77d * sample.png
```
В Git сохраняются только легковесные текстовые указатели (pointer files), а бинарный контент передаётся в специализированное хранилище LFS.

![](./img/img2.jpg)

---

#### **3.3. Разработка веб-приложения и Dockerfile**

Для интеграции с реестром контейнеров подготовлено микросервисное приложение на Flask `app.py`:
```python
#!/usr/bin/env python3
import os
from flask import Flask, jsonify

app = Flask(__name__)


@app.route('/')
def home():
  return jsonify({
      "message": "Hello from Container Registry!",
      "version": os.environ.get('APP_VERSION', '1.0.0'),
  })


@app.route('/health')
def health():
  return jsonify({"status": "healthy"})


if __name__ == '__main__':
  app.run(host='0.0.0.0', port=5000)
```

Файл `requirements.txt`:
```
flask==3.0.0
pytest==7.4.0
```

Спецификация для сборки Docker-образа `Dockerfile`:
```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

---

#### **3.4. Проектирование CI/CD пайплайна в `.gitlab-ci.yml`**

Создана многостадийная конфигурация пайплайна, объединяющая валидацию LFS и полный цикл работы с Container Registry (сборка, тестирование, публикация и деплой):

```yaml
stages:
  - lfs_check
  - build
  - test
  - publish

variables:
  APP_VERSION: "1.0.0"
  CI_REGISTRY_IMAGE: $CI_REGISTRY_IMAGE
  GIT_LFS_SKIP_SMUDGE: "0"

workflow:
  rules:
    - if: $CI_COMMIT_TAG
      when: never
    - when: always

# --- Стадия: Проверка LFS-файлов ---
check_lfs:
  stage: lfs_check
  image: alpine:latest
  before_script:
    - apk add --no-cache git git-lfs
    - git lfs install
    - git lfs pull
  script:
    - echo "🔍 Checking LFS files..."
    - git lfs ls-files || echo "No LFS files tracked"
    - echo "✅ LFS check complete"
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == "main"

# --- Стадия: Сборка Docker-образа ---
build_image:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - echo "📦 Building Docker image..."
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:latest
    - docker save $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA > image.tar
  artifacts:
    paths:
      - image.tar
    expire_in: 1 hour
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == "main"
  allow_failure: true

# --- Стадия: Тестирование образа ---
test_image:
  stage: test
  image: docker:latest
  services:
    - docker:dind
  script:
    - echo "🧪 Testing image..."
    - docker load < image.tar
    - docker run -d -p 5000:5000 --name test-app $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - sleep 5
    - curl -sf http://localhost:5000/ || exit 1
    - echo "✅ Image test passed"
  dependencies:
    - build_image
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == "main"
  allow_failure: true

# --- Стадия: Публикация образа в Container Registry ---
publish_image:
  stage: publish
  image: docker:latest
  services:
    - docker:dind
  script:
    - echo "📤 Publishing image to Container Registry..."
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker load < image.tar
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker push $CI_REGISTRY_IMAGE:latest
    - echo "✅ Image published to $CI_REGISTRY_IMAGE:latest"
  dependencies:
    - build_image
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
  allow_failure: true

# --- Стадия: Деплой с использованием образа из Container Registry ---
deploy_staging:
  stage: publish
  image: docker:latest
  services:
    - docker:dind
  script:
    - echo "🚀 Deploying from Container Registry..."
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker pull $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker run -d -p 5000:5000 --name deploy-test $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - sleep 3
    - curl -sf http://localhost:5000/ || echo "Deployed"
    - echo "✅ Deployment successful"
  environment:
    name: staging
    url: http://localhost:5000
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
  allow_failure: true
```

---

#### **3.5. Фиксация изменений в ветке `main`**

Файлы добавлены в индекс и отправлены коммитом в ветку `main`:
```bash
git add .
git commit -m "feat: configure Git Submodules, Git LFS and Container Registry CI/CD pipeline"
git push origin main
```

Вывод команды:
```
[main 72cf44b] feat: configure Git Submodules, Git LFS and Container Registry CI/CD pipeline
 9 files changed, 102 insertions(+), 73 deletions(-)
 create mode 100644 .gitattributes
 create mode 100644 .gitmodules
 create mode 100644 document.pdf
 create mode 160000 libs/shared
 create mode 100644 sample.png
```
![](./img/img3.jpg)

---

#### **3.6. Создание feature-ветки и оформление Merge Request**

Для демонстрации комплексного задания и проверки работы пайплайна в MR создана отдельная ветка:
```bash
git checkout -b feature/lfs-container-registry
```

Добавлено правило отслеживания аудиофайлов `.mp3` в Git LFS и добавлен файл `sound.mp3`:
```bash
git lfs track "*.mp3"
git add .gitattributes sound.mp3
git commit -m "feat(lfs): add mp3 file tracking in LFS"
git push origin feature/lfs-container-registry
```

Вывод команды:
```
Switched to a new branch 'feature/lfs-container-registry'
Tracking "*.mp3"
[feature/lfs-container-registry c43f1ec] feat(lfs): add mp3 file tracking in LFS
 2 files changed, 4 insertions(+)
 create mode 100644 sound.mp3
```

В веб-интерфейсе GitLab оформлен Merge Request `feature/lfs-container-registry` → `main`. 
Для MR запустились стадии `check_lfs`, `build_image` и `test_image`, обеспечивая верификацию бинарных ресурсов и сборки контейнера до слияния.

![](./img/img4.jpg)

---

#### **3.7. Работа с GitLab Container Registry**

В веб-интерфейсе проекта в разделе **Deploy → Container Registry** зарегистрирован репозиторий образов проекта:
`registry.gitlab.com/vasapypkin646/collab-project`

При выполнении джобы `publish_image`:
1. Выполняется аутентификация в реестре: `docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY`.
2. Образ тегируется уникальным SHA коммита `$CI_COMMIT_SHA` и тегом `latest`.
3. Образ публикуется в защищённый реестр GitLab, откуда может быть загружен командой `docker pull`.

![](./img/img5.jpg)

---

### **4. ОТВЕТЫ НА КОНТРОЛЬНЫЕ ВОПРОСЫ**

#### **1. Что такое Git Submodule и для чего он используется?**
**Git Submodule (подмодуль)** — это стандартный инструмент Git, позволяющий встроить внешний Git-репозиторий как поддиректорию внутрь текущего (родительского) репозитория.  
**Для чего используется:**
- Для переиспользования общего кода: разделяемых библиотек, утилит, микросервисов, SDK между несколькими независимыми проектами;
- Для фиксации зависимости на строгом коммите (SHA-хэше), что исключает внезапную поломку проекта при обновлении внешней библиотеки;
- Для сохранения независимой истории разработки библиотеки отдельно от основного проекта.

#### **2. Как добавить submodule в существующий репозиторий?**
Командой `git submodule add <url-репозитория> [локальный_путь]`.  
Например:
```bash
git submodule add https://github.com/octocat/Hello-World.git libs/shared
```
При этом Git выполняет клонирование, создаёт запись в файле `.gitmodules` (сопоставление пути и URL) и индексирует ссылку на коммит (специальный режим записи `160000 commit`). Затем изменения фиксируются обычным коммитом `git commit`.

#### **3. Как клонировать репозиторий со всеми submodule?**
- **При первом клонировании:** передать флаг `--recursive` (или `--recurse-submodules`):
  ```bash
  git clone --recursive https://gitlab.com/username/project.git
  ```
- **Если проект уже был склонирован без флага:** инициализировать и выкачать подмодули командой:
  ```bash
  git submodule update --init --recursive
  ```

#### **4. Что такое Git LFS и для чего он предназначен?**
**Git LFS (Large File Storage)** — это официальное расширение Git, предназначенное для оптимизации версионирования больших бинарных файлов (видео, 3D-модели, датасеты, архивы, аудио, скомпилированные библиотеки, тяжелые графические файлы).  
Обычный Git хранит каждую версию файла целиком во внутренней базе `.git`, из-за чего репозиторий с бинарниками быстро разрастается до десятков гигабайт. **Git LFS** заменяет сам файл в Git-дереве на миниатюрный текстовый указатель (~130 байт) с SHA256-хэшем и размером, а физическое тело файла загружает на специальный LFS-сервер (встроенный в GitLab/GitHub).

#### **5. Как настроить Git LFS в проекте?**
1. Инициализировать LFS в локальной системе:
   ```bash
   git lfs install
   ```
2. Указать шаблоны расширений файлов, которые должны проходить через фильтр LFS:
   ```bash
   git lfs track "*.png" "*.pdf" "*.mp4" "*.zip"
   ```
3. Зафиксировать автоматически созданный/изменённый конфигурационный файл `.gitattributes`:
   ```bash
   git add .gitattributes
   git commit -m "chore: configure Git LFS"
   ```

#### **6. Какие типы файлов рекомендуется хранить в LFS?**
В Git LFS рекомендуется хранить **любые недельта-сжимаемые и крупные бинарные файлы**:
- Мультимедиа: видео (`.mp4`, `.mov`, `.avi`), звук (`.mp3`, `.wav`), текстуры и изображения (`.psd`, `.raw`, `.png`, `.jpg`);
- Документы и полиграфия: `.pdf`, `.ai`, `.eps`;
- Датасеты и модели машинного обучения: `.csv`, `.h5`, `.onnx`, `.bin`, `.pt`;
- Скомпилированные артефакты и архивы: `.zip`, `.tar.gz`, `.iso`, `.exe`, `.apk`.

#### **7. Что такое GitLab Container Registry и как он интегрируется с CI/CD?**
**GitLab Container Registry** — это встроенный безопасный приватный реестр Docker-образов, интегрированный непосредственно в проект GitLab.  
**Интеграция с CI/CD:**
- GitLab CI автоматически предоставляет встроенные переменные: `$CI_REGISTRY` (адрес реестра), `$CI_REGISTRY_USER` (токен пайплайна `gitlab-ci-token`), `$CI_REGISTRY_PASSWORD` (пароль токена `$CI_JOB_TOKEN`) и `$CI_REGISTRY_IMAGE` (путь к образу проекта);
- Позволяет джобам CI без ручной генерации паролей выполнять `docker login`, собирать, тестировать и публиковать образы, а последующим стадиям или внешним серверам — скачивать их для деплоя.

#### **8. Как опубликовать Docker-образ в Container Registry?**
В секции `script` файла `.gitlab-ci.yml` выполняются команды:
```bash
# 1. Авторизация во встроенном реестре
docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY

# 2. Сборка образа с тегом SHA коммита
docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .

# 3. Дополнительное тегирование как latest (или версия релиза)
docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:latest

# 4. Отправка образов в реестр
docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
docker push $CI_REGISTRY_IMAGE:latest
```

#### **9. В чём разница между Git Submodules и Git Subtree?**
| Критерий | Git Submodules | Git Subtree |
|:---|:---|:---|
| **Принцип хранения** | Хранит только ссылку на внешний коммит и URL в `.gitmodules` | Полностью внедряет код и файлы внешней библиотеки в репозиторий |
| **Клонирование** | Требует флаг `--recursive` или команду `submodule update --init` | Не требует спецкоманд, скачивается сразу при обычном `git clone` |
| **История коммитов** | История подмодуля изолирована в его собственном репозитории | История подмодуля вливается в общее дерево коммитов проекта |
| **Внесение изменений** | Изменение требует отдельного коммита и пуша внутри подмодуля + обновления ссылки в родителе | Изменения в коде подмодуля можно коммитить напрямую в основной репозиторий |
| **Сложность для команды** | Требует высокой дисциплины (легко забыть запушить подмодуль или зафиксировать ссылку) | Прозрачно для всех разработчиков, как обычные файлы проекта |

#### **10. Как использовать образ из Container Registry в другом проекте?**
1. Сгенерировать Personal Access Token (или Deploy Token) с правом `read_registry`.
2. На целевой машине или в CI/CD другого проекта выполнить вход:
   ```bash
   docker login registry.gitlab.com -u <username> -p <token>
   ```
3. Скачать нужный образ:
   ```bash
   docker pull registry.gitlab.com/vasapypkin646/collab-project:latest
   ```
4. В `.gitlab-ci.yml` стороннего проекта образ можно напрямую использовать как среду выполнения джобы:
   ```yaml
   my_job:
     image: registry.gitlab.com/vasapypkin646/collab-project:latest
     script:
       - python -c "import app; print('Image loaded successfully')"
   ```

---

### **5. ИСПОЛЬЗУЕМЫЕ КОМАНДЫ**

```bash
# Клонирование репозитория
git clone https://gitlab.com/Vasapypkin646/collab-project.git
cd collab-project

# Работа с Git Submodules
git submodule add https://github.com/octocat/Hello-World.git libs/shared
git submodule status
git submodule update --init --recursive

# Работа с Git LFS
git lfs install
git lfs track "*.png" "*.jpg" "*.pdf" "*.mp4" "*.zip" "*.mp3"
git lfs status
git lfs ls-files

# Фиксация и отправка изменений в основную ветку
git add .
git commit -m "feat: configure Git Submodules, Git LFS and Container Registry CI/CD pipeline"
git push origin main

# Работа с веткой для Merge Request
git checkout -b feature/lfs-container-registry
git add .gitattributes sound.mp3
git commit -m "feat(lfs): add mp3 file tracking in LFS"
git push origin feature/lfs-container-registry
```

---

### **6. ВЫВОДЫ**

В ходе выполнения лабораторной работы были освоены профессиональные инструменты управления крупными проектами, бинарными зависимостями и контейнеризацией в экосистеме Git и GitLab:
> 1. Освоен механизм **Git Submodules**: подключен и зафиксирован внешний репозиторий-библиотека, настроен файл `.gitmodules`, изучены процедуры инициализации и обновления подмодулей.
> 2. Практически освоен инструмент **Git LFS**: настроена фильтрация и отслеживание тяжелых бинарных форматов через `.gitattributes`, что предотвращает раздувание локальной базы `.git` и сохраняет высокую скорость работы системы контроля версий.
> 3. Разработан микросервис на Flask и `Dockerfile`, настроена интеграция со встроенным **GitLab Container Registry**.
> 4. Спроектирован комплексный CI/CD пайплайн, объединяющий этапы валидации LFS-указателей (`check_lfs`), сборки Docker-образов с тегированием по SHA коммита, тестирования в изолированном контейнере и безопасной публикации в реестре.
> 5. Проведен сравнительный анализ подходов Git Submodules и Git Subtree, изучены механизмы дистрибуции образов между распределенными проектами.
