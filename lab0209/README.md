# **Отчет по лабораторной работе №9: «Стратегии ветвления в команде. Git Flow, GitLab Flow и Trunk-based Development»**

## **Сведения о студенте**
**Дата:** 2026-11-25    
**Семестр:** 3 курс, 5 семестр  
**Группа:** Пин-б-о-24-1    
**Дисциплина:** Системы контроля версий     
**Студент:** Лебский Артём Александрович    

---

### **Структура готового проекта**
```
collab-project/
├── .git/                     # Каталог локальной базы данных репозитория Git
├── .gitlab-ci.yml            # CI/CD пайплайн с поддержкой GitLab Flow (staging / production)
├── COMPARISON.md             # Сравнительный анализ стратегий Git Flow, GitLab Flow и Trunk-based
├── auth.py                   # Модуль аутентификации (разработан в ветке feature/user-authentication)
├── calc.py                   # Модуль калькулятора (разработан в ветке feature/add-calculator)
├── fix.py                    # Критический патч безопасности (разработан в ветке hotfix/critical-bug)
├── version.py                # Файл версионирования релиза (создан в ветке release/v1.0.0)
├── app.py                    # Микросервис на Flask
├── requirements.txt          # Зависимости проекта
├── Dockerfile                # Docker-спецификация контейнера
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

> 1. Изучить три основные стратегии ветвления: Git Flow, GitLab Flow и Trunk-based Development.
> 2. Сравнить стратегии по сложности, гибкости, рискам возникновения конфликтов слияния и применимости в различных типах проектов.
> 3. Освоить практическую реализацию Git Flow и GitLab Flow в распределённом репозитории GitLab.
> 4. Научиться управлять жизненным циклом релиза, создавать release-ветки, фичи и хотфиксы (hotfix) с корректным распространением изменений между ветками.
> 5. Выбрать и обосновать оптимальную стратегию ветвления под конкретные условия командной разработки.

---

### **2. ЗАДАЧИ РАБОТЫ**

> 1. Инициализация и настройка структуры веток Git Flow: основной ветки стабильного кода `main` и интеграционной ветки разработки `develop`.
> 2. Разработка новой функциональности в изолированной ветке `feature/user-authentication` (создание `auth.py`) и её слияние в `develop`.
> 3. Формирование релизного цикла: создание релизной ветки `release/v1.0.0`, фиксация версии `version.py`, слияние в `main` с установкой релизного тега `v1.0.0` и обратное слияние в `develop`.
> 4. Отработка сценария срочного исправления дефекта в production: создание ветки `hotfix/critical-bug` от `main`, реализация патча `fix.py`, слияние в `main` (тег `v1.0.1`) и синхронизация с `develop`.
> 5. Реализация модели GitLab Flow: разработка модуля `calc.py` в ветке `feature/add-calculator`, слияние в `main` через Merge Request и настройка CI/CD пайплайна с окружениями `staging` и `production`.
> 6. Составление аналитической таблицы сравнения стратегий ветвления в файле `COMPARISON.md`.
> 7. Анализ графа истории коммитов (`git log --graph --oneline`).

---

### **3. ХОД ВЫПОЛНЕНИЯ РАБОТЫ**

#### **3.1. Создание интеграционной ветки `develop`**

В соответствии с методологией Git Flow была создана постоянная ветка разработки `develop`, ответвлённая от стабильной ветки `main`:
```bash
git checkout -b develop
```
Ветка `develop` аккумулирует всю новую функциональность, предназначенную для следующего релиза.

![](./img/img1.jpg)

---

#### **3.2. Разработка функциональности в feature-ветке (Git Flow)**

Для реализации модуля авторизации от ветки `develop` создана изолированная ветка `feature/user-authentication`:
```bash
git checkout -b feature/user-authentication develop
```

Создан программный модуль `auth.py`:
```python
#!/usr/bin/env python3
"""Модуль аутентификации пользователей."""


def login(username, password):
  if username == "admin" and password == "secret":
    return {"status": "success", "token": "fake-jwt-token"}
  return {"status": "error", "message": "Invalid credentials"}


def logout(session_id):
  return {"status": "success", "message": "Logged out"}


def register(username, password, email):
  return {"status": "success", "message": "User registered"}
```

Файл добавлен в индекс, зафиксирован коммитом и слит в ветку `develop` с флагом `--no-ff` (для сохранения топологии ветвления в истории Git):
```bash
git add auth.py
git commit -m "feat: add user authentication module"
git checkout develop
git merge --no-ff feature/user-authentication -m "Merge branch 'feature/user-authentication' into develop"
```

Вывод команды:
```
[feature/user-authentication 692f62c] feat: add user authentication module
Switched to branch 'develop'
Merge made by the 'ort' strategy.
 auth.py | 16 ++++++++++++++++
```

![](./img/img2.jpg)

---

#### **3.3. Создание релизной ветки `release/v1.0.0` и выпуск релиза**

Когда функциональность в `develop` стабилизировалась, создана релизная ветка `release/v1.0.0`:
```bash
git checkout -b release/v1.0.0 develop
```

В релизной ветке зафиксирована версия продукта в файле `version.py`:
```python
VERSION = "1.0.0"
RELEASE_NAME = "Initial Production Release"
```

Выполнены коммит и слияние релиза в обе ключевые ветки:
1. **В `main`** — код переходит в релизное состояние, устанавливается аннотированный Git-тег `v1.0.0`:
   ```bash
   git add version.py
   git commit -m "chore(release): bump version to 1.0.0"
   git checkout main
   git merge --no-ff release/v1.0.0 -m "Merge branch 'release/v1.0.0' into main"
   git tag -a v1.0.0 -m "Release v1.0.0"
   ```
2. **В `develop`** — для синхронизации внесенных в релизной ветке правок с текущей разработкой:
   ```bash
   git checkout develop
   git merge --no-ff release/v1.0.0 -m "Merge branch 'release/v1.0.0' into develop"
   ```

![](./img/img3.jpg)

---

#### **3.4. Отработка критического исправления (Hotfix)**

При обнаружении критической уязвимости в production-коде ветки `main` создана ветка срочного исправления `hotfix/critical-bug`:
```bash
git checkout -b hotfix/critical-bug main
```

Создан файл патча `fix.py`:
```python
#!/usr/bin/env python3
"""Критическое исправление безопасности."""


def apply_security_patch():
  print("Critical bug fixed: patched authentication vulnerability")
  return True


if __name__ == "__main__":
  apply_security_patch()
```

Изменения зафиксированы и незамедлительно влиты:
1. **В `main`** с созданием минорного патч-тега `v1.0.1`:
   ```bash
   git add fix.py
   git commit -m "fix: resolve critical authentication bug"
   git checkout main
   git merge --no-ff hotfix/critical-bug -m "Merge branch 'hotfix/critical-bug' into main"
   git tag -a v1.0.1 -m "Hotfix v1.0.1"
   ```
2. **В `develop`**, чтобы баг не вернулся со следующим плановым релизом:
   ```bash
   git checkout develop
   git merge --no-ff hotfix/critical-bug -m "Merge branch 'hotfix/critical-bug' into develop"
   ```

![](./img/img4.jpg)

---

#### **3.5. Реализация стратегии GitLab Flow**

Для демонстрации подхода GitLab Flow разработана функция калькулятора через прямую feature-ветку от `main`:
```bash
git checkout -b feature/add-calculator main
```

Создан файл `calc.py`:
```python
#!/usr/bin/env python3
"""Модуль калькулятора для демонстрации GitLab Flow."""


class Calculator:

  def add(self, a, b):
    return a + b

  def subtract(self, a, b):
    return a - b

  def multiply(self, a, b):
    return a * b

  def divide(self, a, b):
    if b == 0:
      raise ValueError("Cannot divide by zero")
    return a / b


if __name__ == "__main__":
  calc = Calculator()
  print(f"2 + 2 = {calc.add(2, 2)}")
```

Выполнены коммит и слияние в `main`:
```bash
git add calc.py
git commit -m "feat: add calculator module"
git checkout main
git merge --no-ff feature/add-calculator -m "Merge branch 'feature/add-calculator' into main"
```

Сконфигурирован файл `.gitlab-ci.yml`, отражающий концепцию GitLab Flow с автоматическим прогоном тестов, деплоем в окружение `staging` и ручным деплоем (`when: manual`) в `production`:
```yaml
stages:
  - test
  - deploy

variables:
  APP_VERSION: "1.0.1"

workflow:
  rules:
    - if: $CI_COMMIT_TAG
      when: never
    - when: always

# --- Стадия тестирования ---
test:
  stage: test
  image: python:3.11-slim
  script:
    - echo "Running unit tests for Calculator..."
    - python -c "from calc import Calculator; c=Calculator(); assert c.add(1,2)==3; assert c.multiply(3,4)==12; assert c.subtract(10,5)==5"
    - echo "✅ All tests passed successfully!"
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == "main"
    - if: $CI_COMMIT_BRANCH == "develop"

# --- Стадия деплоя на staging (автоматический) ---
deploy_staging:
  stage: deploy
  image: alpine:latest
  script:
    - echo "🚀 Deploying to Staging environment..."
    - echo "Version: $APP_VERSION"
    - echo "✅ Staging deployment complete"
  environment:
    name: staging
    url: https://staging.example.com
  rules:
    - if: $CI_COMMIT_BRANCH == "main"

# --- Стадия деплоя на production (ручной) ---
deploy_production:
  stage: deploy
  image: alpine:latest
  script:
    - echo "🚀 Deploying to Production environment..."
    - echo "Version: $APP_VERSION"
    - echo "✅ Production deployment complete"
  environment:
    name: production
    url: https://production.example.com
  when: manual
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
```

![](./img/img5.jpg)

---

#### **3.6. Сравнительный анализ стратегий ветвления**

В корне проекта сформирован документ `COMPARISON.md`, детально описывающий различия между Git Flow, GitLab Flow и Trunk-based Development:

| Критерий | Git Flow | GitLab Flow | Trunk-based Development |
|:---|:---|:---|:---|
| **Сложность** | Высокая | Средняя | Низкая |
| **Количество постоянных веток** | 2 (`main`, `develop`) | 1 (`main`) + ветки окружений | 1 (`main` / `trunk`) |
| **Вспомогательные ветки** | `feature/*`, `release/*`, `hotfix/*` | `feature/*`, окружения (`staging`, `prod`) | Короткоживущие `feature/*` (1-2 дня) |
| **Релизный цикл** | Плановый, фиксированный (версионированный) | Непрерывная доставка (Continuous Delivery) | Непрерывный деплой (Continuous Deployment) |
| **Поддержка hotfix'ов** | ✅ Отличная (выделенная ветка `hotfix/*`) | ⚠️ Через обычную feature-ветку | ⚠️ Фикс в main или cherry-pick |
| **Использование Feature-флагов** | ❌ Не требуется (изоляция в ветках) | ⚠️ Опционально | ✅ Обязательно для незавершенных фич |
| **Интеграция с CI/CD** | ⚠️ Средняя (сложные правила триггеров) | ✅ Отличная (нативно под CI/CD) | ✅ Максимальная (быстрые частые сборки) |
| **Риск конфликтов слияния** | ⚠️ Высокий (долгоживущие ветки) | ⚠️ Средний | ✅ Минимальный (частые слияния) |
| **Когда использовать** | Крупные enterprise-продукты с циклами релизов | Веб-сервисы, SaaS, команды 5-15 разработчиков | Стартапы, зрелые DevOps-команды, микросервисы |

---

#### **3.7. Граф истории ветвления репозитория**

Сгенерирован граф коммитов командой `git log --graph --oneline --all`:
```
*   e996872 (develop) Merge branch 'main' into develop
|\  
| * 3ea433c (HEAD -> main) docs: add strategies comparison and update CI/CD pipeline
| *   fd8e676 Merge branch 'feature/add-calculator' into main
| |\  
| | * 4db41ae (feature/add-calculator) feat: add calculator module
| |/  
| *   5f11b65 (tag: v1.0.1) Merge branch 'hotfix/critical-bug' into main
| |\  
* | \   70dfed9 Merge branch 'hotfix/critical-bug' into develop
|\ \ \  
| | |/  
| |/|   
| * | 05736cc (hotfix/critical-bug) fix: resolve critical authentication bug
| |/  
| *   ee78154 (tag: v1.0.0) Merge branch 'release/v1.0.0' into main
| |\  
* | \   9c516cd Merge branch 'release/v1.0.0' into develop
|\ \ \  
| | |/  
| |/|   
| * | b5a7288 (release/v1.0.0) chore(release): bump version to 1.0.0
|/ /  
* |   f3fe491 Merge branch 'feature/user-authentication' into develop
|\ \  
| |/  
|/|   
| * 692f62c (feature/user-authentication) feat: add user authentication module
|/  
* 72cf44b feat: configure Git Submodules, Git LFS and Container Registry CI/CD pipeline
```
Граф наглядно иллюстрирует:
1. Изоляцию фичи в `develop`.
2. Расхождение релизной ветки с последующим слиянием в `main` (тег `v1.0.0`) и `develop`.
3. Оперативное ветвление hotfix от `main` со слиянием в `main` (тег `v1.0.1`) и синхронизацией в `develop`.
4. Слияние фичи по модели GitLab Flow напрямую в `main`.

---

### **4. ОТВЕТЫ НА КОНТРОЛЬНЫЕ ВОПРОСЫ**

#### **1. Какие основные ветки существуют в Git Flow?**
В модели Git Flow выделяют две **постоянные (долгоживущие)** ветки:
- **`main` (ранее `master`):** отражает текущее стабильное production-состояние системы. В этой ветке каждый коммит соответствует боевому релизу и обязательно помечается тегом версии (`v1.0.0`, `v1.0.1`). Прямые коммиты разработчиков сюда запрещены.
- **`develop`:** интеграционная ветка разработки. Содержит полный набор изменений, готовых к включению в следующий плановый релиз. Именно от неё ответвляются новые фичи.

#### **2. Для чего нужна ветка `develop` в Git Flow?**
Ветка `develop` служит буфером интеграции:
- Позволяет изолировать повседневную разработку от стабильного боевого кода в `main`;
- Служит точкой слияния для всех завершённых `feature`-веток команды;
- Позволяет тестировать взаимодействие новых компонентов между собой до того, как начнётся процедура стабилизации релиза.

#### **3. Как создаётся и сливается релизная ветка?**
1. **Создание:** когда в `develop` собраны все запланированные функции, создаётся ветка `release/vX.Y.Z` (например, `git checkout -b release/v1.0.0 develop`).
2. **Стабилизация:** в ней правятся исключительно баги, обновляются конфигурации, номера версий (`version.py`) и документация. Новая функциональность сюда не добавляется.
3. **Слияние:**
   - Вливается в **`main`**, где коммит маркируется релизным тегом: `git tag -a v1.0.0`;
   - Вливается обратно в **`develop`**, чтобы внесённые исправления не потерялись в будущем.
   После этого ветка `release/*` может быть удалена.

#### **4. В чём отличие hotfix от feature-ветки?**
| Критерий | Feature-ветка | Hotfix-ветка |
|:---|:---|:---|
| **Точка ответвления** | Создаётся строго от **`develop`** | Создаётся строго от **`main`** |
| **Цель** | Разработка нового запланированного функционала | Срочное устранение критического сбоя в production |
| **Куда сливается** | Сливается только в **`develop`** | Сливается одновременно в **`main`** и в **`develop`** |
| **Тегирование** | Не тегируется | Тегируется патч-версией (`v1.0.1`) |
| **Срочность** | Плановая (недели / спринты) | Экстренная (часы) |

#### **5. Какие ветки существуют в GitLab Flow?**
В GitLab Flow используется одна главная постоянная ветка — **`main`**, от которой создаются короткоживущие **`feature/*`** ветки (сливаемые обратно в `main` через Merge Request).  
Дополнительно модель использует:
- **Ветки окружений (Environment branches):** например, ветки `staging`, `pre-production`, `production`. Деплой в соответствующую среду происходит при переносе коммита из `main` в ветку окружения.
- **Релизные ветки (Release branches):** например, `stable-1-0`, `stable-2-0` (актуально для коробочных продуктов, требующих поддержки старых версий).

#### **6. Как организован деплой на staging в GitLab Flow?**
В GitLab Flow деплой на staging может быть организован двумя путями:
1. **Автоматически при слиянии коммита в `main`:** как только Merge Request одобрен и слит в `main`, раннер автоматически запускает джобу деплоя на staging-сервер.
2. **Через выделенную ветку `staging`:** разработчик делает MR из `main` в `staging`, и при слиянии триггерится пайплайн развертывания.

#### **7. В чём суть Trunk-based Development?**
Суть Trunk-based Development заключается в отказе от долгоживущих веток:
- Все разработчики работают над одной центральной веткой (**`trunk`** или **`main`**);
- Feature-ветки либо не создаются вовсе, либо живут крайне мало (не более 1–2 дней) и содержат небольшие порции изменений (small batches);
- Разработчики сливают свой код в основную ветку по нескольку раз в день, что практически полностью устраняет проблему сложных и болезненных конфликтов слияния (Merge Hell).

#### **8. Какие преимущества дают feature-флаги в Trunk-based?**
**Feature Flags (Feature Toggles)** позволяют разделить понятия *развертывания кода (Deployment)* и *включения функционала (Release)*:
- Недописанная или экспериментальная функция может быть залита в `main` и даже развёрнута в production, но оставаться невидимой для пользователей (скрыта условием `if (featureEnabled)`);
- Позволяют проводить A/B-тестирование и постепенную выкатку (Canary releases) на процент аудитории;
- Дают возможность мгновенного отключения проблемной функции в проде без выполнения срочного отката (Rollback) или хотфикса.

#### **9. Какую стратегию вы выберете для стартапа и почему?**
Для стартапа оптимален **Trunk-based Development** (либо облегчённый **GitLab Flow**):
- **Максимальный Time-to-Market:** стартапу критически важно проверять гипотезы и доставлять ценность клиентам ежедневно, а не раз в квартал;
- **Минимум бюрократии:** нет накладных расходов на поддержание сложной сети веток (`develop`, `release`, `hotfix`);
- **Небольшая команда:** при составе команды до 5–7 человек синхронизация через короткоживущие ветки происходит быстро и безболезненно.

#### **10. В каких случаях стоит использовать Git Flow?**
Git Flow оправдан в ситуациях, когда:
- Продукт распространяется как коробочное ПО или мобильные/десктопные приложения с фиксированными циклами версионирования (Semantic Versioning);
- Требуется длительное регрессионное тестирование кандидатов в релиз отдельным отделом тестирования (QA);
- Разработчики должны одновременно поддерживать и патчить несколько выпущенных версий продукта (например, версии 1.2, 2.0 и 2.1);
- В проекте занято большое число распределённых разработчиков со строгим разграничением прав на выпуск релизов.

---

### **5. ИСПОЛЬЗУЕМЫЕ КОМАНДЫ**

```bash
# Клонирование репозитория
git clone https://gitlab.com/Vasapypkin646/collab-project.git
cd collab-project

# Реализация Git Flow
git checkout -b develop
git checkout -b feature/user-authentication develop
git add auth.py
git commit -m "feat: add user authentication module"
git checkout develop
git merge --no-ff feature/user-authentication -m "Merge branch 'feature/user-authentication' into develop"

# Создание и завершение релиза
git checkout -b release/v1.0.0 develop
git add version.py
git commit -m "chore(release): bump version to 1.0.0"
git checkout main
git merge --no-ff release/v1.0.0 -m "Merge branch 'release/v1.0.0' into main"
git tag -a v1.0.0 -m "Release v1.0.0"
git checkout develop
git merge --no-ff release/v1.0.0 -m "Merge branch 'release/v1.0.0' into develop"

# Создание и применение Hotfix
git checkout -b hotfix/critical-bug main
git add fix.py
git commit -m "fix: resolve critical authentication bug"
git checkout main
git merge --no-ff hotfix/critical-bug -m "Merge branch 'hotfix/critical-bug' into main"
git tag -a v1.0.1 -m "Hotfix v1.0.1"
git checkout develop
git merge --no-ff hotfix/critical-bug -m "Merge branch 'hotfix/critical-bug' into develop"

# Реализация GitLab Flow
git checkout -b feature/add-calculator main
git add calc.py
git commit -m "feat: add calculator module"
git checkout main
git merge --no-ff feature/add-calculator -m "Merge branch 'feature/add-calculator' into main"

# Просмотр истории и графа веток
git log --graph --oneline --all
git tag -n
```

---

### **6. ВЫВОДЫ**

В ходе выполнения лабораторной работы были всесторонне изучены и применены на практике современные отраслевые методологии ветвления в системах контроля версий:
> 1. На практике освоена классическая модель **Git Flow**: выстроена двухуровневая иерархия постоянных веток `main` и `develop`, отработаны сценарии изоляции функционала (`feature/*`), стабилизации релиза (`release/*`) и выпуска критических патчей (`hotfix/*`) с соблюдением двусторонней синхронизации.
> 2. Реализована модель **GitLab Flow**: продемонстрирована работа с короткоживущими ветками от `main`, настроен пайплайн развертывания с привязкой к окружениям `staging` и `production`.
> 3. Изучена философия **Trunk-based Development**, основанная на частых коммитах в единый ствол и применении Feature Flags для безопасного сокрытия недоработанного функционала.
> 4. Составлен подробный сравнительный анализ стратегий ветвления (`COMPARISON.md`), позволяющий аргументированно выбирать архитектуру рабочего процесса под конкретные параметры команды и частоту релизов.
