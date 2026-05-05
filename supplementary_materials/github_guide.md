# Полное руководство по Git и GitHub для работы с AI-агентами

## Содержание

1. [Введение](#введение)
2. [Основы Git](#основы-git)
3. [Работа с GitHub](#работа-с-github)
4. [Рабочий процесс для курса](#рабочий-процесс-для-курса)
5. [Продвинутые техники](#продвинутые-техники)
6. [Решение распространенных проблем](#решение-распространенных-проблем)
7. [Best Practices для работы с кодом агентов](#best-practices-для-работы-с-кодом-агентов)
8. [Шпаргалка по командам](#шпаргалка-по-командам)

---

## Введение

### Зачем нужен Git и GitHub в курсе по AI-агентам?

При работе с AI-агентами вы будете:
- **Хранить код агентов** (промпты, функции, конфигурации)
- **Отслеживать изменения** в логике агентов
- **Сотрудничать** с другими разработчиками
- **Делиться** своими наработками
- **Возвращаться** к рабочим версиям при ошибках
- **Управлять версиями** промптов и конфигураций

### Что такое Git и GitHub?

**Git** — система контроля версий (VCS), которая:
- Сохраняет историю изменений вашего кода
- Позволяет работать над разными функциями параллельно
- Дает возможность откатиться к любой предыдущей версии

**GitHub** — веб-платформа для хостинга Git-репозиториев:
- Централизованное хранение кода
- Инструменты для совместной работы (Pull Requests, Issues)
- Интеграция с CI/CD, документацией и другими сервисами

---

## Основы Git

### Установка Git

#### Windows
1. Скачайте установщик с [git-scm.com](https://git-scm.com/download/win)
2. Запустите установщик, оставьте настройки по умолчанию
3. Проверьте установку:
```bash
git --version
```

#### macOS
```bash
# Через Homebrew (рекомендуется)
brew install git

# Или через Xcode Command Line Tools
xcode-select --install
```

#### Linux (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install git
```

### Первая настройка

После установки настройте свое имя и email:

```bash
git config --global user.name "Ваше Имя"
git config --global user.email "your.email@example.com"
```

Проверка настроек:
```bash
git config --list
```

**Важно:** Используйте тот же email, что и в аккаунте GitHub!

### Основные концепции Git

#### Репозиторий (Repository)
Хранилище вашего проекта со всей историей изменений.

#### Коммит (Commit)
Снимок изменений в определенный момент времени. Каждый коммит имеет:
- Уникальный хэш (например, `a1b2c3d`)
- Автора
- Дату создания
- Сообщение описывающее изменения

#### Ветка (Branch)
Параллельная линия разработки. Позволяет работать над новыми функциями, не затрагивая основную версию.

#### Стадия (Staging Area)
Промежуточная зона, куда вы добавляете файлы перед коммитом.

### Создание репозитория

#### Инициализация нового репозитория

```bash
# Создайте папку для проекта
mkdir my-ai-agent
cd my-ai-agent

# Инициализируйте Git-репозиторий
git init
```

После этой команды создается скрытая папка `.git` со всей структурой репозитория.

#### Клонирование существующего репозитория

```bash
# Клонировать по HTTPS
git clone https://github.com/username/repository-name.git

# Клонировать по SSH (рекомендуется)
git clone git@github.com:username/repository-name.git

# Клонировать в конкретную папку
git clone https://github.com/username/repository-name.git my-project-folder
```

### Базовый рабочий цикл

#### 1. Проверка статуса

```bash
git status
```

Показывает:
- Какие файлы изменены
- Какие файлы добавлены в стадию
- Какие файлы не отслеживаются

#### 2. Добавление файлов в стадию

```bash
# Добавить конкретный файл
git add filename.py

# Добавить все файлы в текущей папке
git add .

# Добавить все файлы с определенным расширением
git add *.py

# Добавить все измененные файлы (кроме новых)
git add -u
```

#### 3. Создание коммита

```bash
git commit -m "Описание изменений"
```

**Хорошие сообщения коммитов:**
- ✅ `Add user authentication to agent`
- ✅ `Fix bug in prompt template parsing`
- ✅ `Update API key configuration`

**Плохие сообщения коммитов:**
- ❌ `fix`
- ❌ `update`
- ❌ `changes`

#### 4. Просмотр истории

```bash
# Показать последние коммиты
git log

# Показать последние 5 коммитов
git log -n 5

# Показать коммиты в одну строку
git log --oneline

# Показать коммиты с графиком веток
git log --graph --oneline --all
```

### Работа с ветками

#### Создание и переключение веток

```bash
# Создать новую ветку
git branch feature-name

# Переключиться на ветку
git checkout feature-name

# Создать и переключиться одной командой (рекомендуется)
git checkout -b feature-name

# Современный синтаксис (Git 2.23+)
git switch -b feature-name
```

#### Просмотр веток

```bash
# Показать все локальные ветки
git branch

# Показать все ветки (локальные и удаленные)
git branch -a

# Показать текущую ветку
git branch --show-current
```

#### Слияние веток (Merge)

```bash
# Переключиться на основную ветку
git checkout main

# Влить изменения из другой ветки
git merge feature-name
```

#### Удаление веток

```bash
# Удалить локальную ветку
git branch -d feature-name

# Принудительно удалить (если не слита)
git branch -D feature-name
```

### Отмена изменений

#### Отмена изменений в файле до стадии

```bash
# Отменить изменения в конкретном файле
git checkout -- filename.py

# Современный синтаксис
git restore filename.py
```

#### Удаление файла из стадии

```bash
git reset HEAD filename.py

# Современный синтаксис
git restore --staged filename.py
```

#### Отмена последнего коммита

```bash
# Отменить коммит, сохранить изменения в рабочей директории
git reset --soft HEAD~1

# Отменить коммит и изменения
git reset --hard HEAD~1
```

**⚠️ Осторожно:** `--hard` безвозвратно удаляет изменения!

#### Изменение последнего коммита

```bash
# Добавить забытые файлы и изменить сообщение
git add forgotten_file.py
git commit --amend -m "Новое описание коммита"
```

---

## Работа с GitHub

### Регистрация и настройка аккаунта

1. Перейдите на [github.com](https://github.com)
2. Нажмите "Sign up"
3. Введите email, придумайте пароль и имя пользователя
4. Подтвердите email
5. Пройдите капчу

### Настройка аутентификации

#### Вариант 1: HTTPS с токеном (рекомендуется для начинающих)

1. Зайдите в Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Нажмите "Generate new token (classic)"
3. Выберите срок действия и права (минимум `repo`)
4. Скопируйте токен (**сохраните его, больше не увидите!**)
5. При клонировании используйте токен вместо пароля

#### Вариант 2: SSH-ключи (более безопасно и удобно)

**Генерация SSH-ключа:**

```bash
# Создать новый SSH-ключ
ssh-keygen -t ed25519 -C "your.email@example.com"

# Или для старых систем
ssh-keygen -t rsa -b 4096 -C "your.email@example.com"
```

Нажимайте Enter для принятия настроек по умолчанию.

**Добавление ключа в ssh-agent:**

```bash
# Запустить агент
eval "$(ssh-agent -s)"

# Добавить ключ
ssh-add ~/.ssh/id_ed25519
```

**Добавление ключа в GitHub:**

1. Скопируйте публичный ключ:
```bash
cat ~/.ssh/id_ed25519.pub
```

2. В GitHub: Settings → SSH and GPG keys → New SSH key
3. Вставьте ключ и дайте название
4. Нажмите "Add SSH key"

**Проверка подключения:**

```bash
ssh -T git@github.com
```

Должны увидеть приветствие от GitHub.

### Основные операции с GitHub

#### Push (отправка изменений)

```bash
# Отправить локальную ветку на сервер
git push origin branch-name

# Отправить и установить связь с удаленной веткой
git push -u origin branch-name

# Отправить все ветки
git push --all origin
```

#### Pull (получение изменений)

```bash
# Получить и слить изменения с сервера
git pull origin main

# Только получить изменения (без слияния)
git fetch origin
```

#### Fork (ответвление репозитория)

Fork создает копию чужого репозитория в вашем аккаунте:

1. Откройте репозиторий на GitHub
2. Нажмите кнопку "Fork" в правом верхнем углу
3. Выберите аккаунт для форка
4. Клонируйте свой форк:
```bash
git clone https://github.com/YOUR_USERNAME/repository-name.git
```

#### Pull Request (запрос на слияние)

PR позволяет предложить свои изменения в основной репозиторий:

1. Создайте форк репозитория
2. Создайте новую ветку для своих изменений
3. Внесите изменения и закоммитьте их
4. Отправьте ветку на сервер:
```bash
git push origin your-feature-branch
```
5. На GitHub нажмите "New Pull Request"
6. Выберите базовую ветку и вашу ветку
7. Добавьте описание изменений
8. Отправьте на ревью

### Работа с Issues

Issues используются для отслеживания задач, багов и предложений:

- **Создание Issue:** Нажмите "New Issue" в репозитории
- **Шаблоны:** Многие проекты используют шаблоны для разных типов задач
- **Метки (Labels):** Категоризируют issues (bug, enhancement, question)
- **Assignees:** Назначают ответственных
- **Связь с кодом:** В коммитах упоминайте `#номер_issue` для автоматической связи

Пример:
```bash
git commit -m "Fix agent timeout issue #42"
```

### GitHub Pages

Хостинг статических сайтов прямо из репозитория:

1. Создайте репозиторий с именем `username.github.io`
2. Добавьте файлы сайта (HTML, CSS, JS)
3. В Settings → Pages выберите ветку и папку
4. Сайт доступен по адресу `https://username.github.io`

### GitHub Actions

Автоматизация процессов (CI/CD, тесты, деплой):

Пример workflow для тестирования Python-кода:

```yaml
# .github/workflows/test.yml
name: Run Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.10'
    
    - name: Install dependencies
      run: |
        pip install -r requirements.txt
        pip install pytest
    
    - name: Run tests
      run: pytest
```

---

## Рабочий процесс для курса

### Типичный сценарий работы над проектом агента

#### Шаг 1: Подготовка

```bash
# Клонируйте репозиторий курса
git clone git@github.com:course-repo/ai-agents-course.git
cd ai-agents-course

# Создайте свою ветку для домашнего задания
git checkout -b homework-1-yourname
```

#### Шаг 2: Разработка

```bash
# Создайте файлы агента
touch agents/my_agent.py
touch prompts/my_agent_prompts.md

# Редактируйте файлы в любимом редакторе

# Периодически проверяйте статус
git status

# Добавляйте готовые части
git add agents/my_agent.py
git add prompts/my_agent_prompts.md

# Коммитьте с понятными сообщениями
git commit -m "Add basic agent structure with function calling"
git commit -m "Implement prompt templates for customer support"
```

#### Шаг 3: Синхронизация с основным репозиторием

```bash
# Добавьте upstream (основной репозиторий)
git remote add upstream https://github.com/course-repo/ai-agents-course.git

# Получите изменения из основного репозитория
git fetch upstream

# Слейте изменения в свою ветку
git merge upstream/main homework-1-yourname

# Или используйте rebase для более чистой истории
git rebase upstream/main
```

#### Шаг 4: Отправка работы

```bash
# Отправьте свою ветку на GitHub
git push -u origin homework-1-yourname

# Создайте Pull Request через веб-интерфейс
```

### Структура репозитория для проекта агента

Рекомендуемая структура:

```
my-ai-agent/
├── .gitignore
├── README.md
├── requirements.txt
├── config/
│   ├── __init__.py
│   └── settings.py
├── agents/
│   ├── __init__.py
│   ├── base_agent.py
│   └── customer_support_agent.py
├── prompts/
│   ├── system_prompts.md
│   └── user_templates.md
├── tools/
│   ├── __init__.py
│   ├── search_tool.py
│   └── calculator_tool.py
├── tests/
│   ├── __init__.py
│   ├── test_agent.py
│   └── test_tools.py
├── notebooks/
│   └── experimentation.ipynb
└── docs/
    └── api_reference.md
```

### .gitignore для Python-проектов

Создайте файл `.gitignore` в корне проекта:

```gitignore
# Byte-compiled / optimized / DLL files
__pycache__/
*.py[cod]
*$py.class

# C extensions
*.so

# Distribution / packaging
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg

# PyInstaller
*.manifest
*.spec

# Installer logs
pip-log.txt
pip-delete-this-directory.txt

# Unit test / coverage reports
htmlcov/
.tox/
.nox/
.coverage
.coverage.*
.cache
nosetests.xml
coverage.xml
*.cover
*.py,cover
.hypothesis/
.pytest_cache/

# Translations
*.mo
*.pot

# Environments
.env
.venv
env/
venv/
ENV/
env.bak/
venv.bak/

# IDE
.idea/
.vscode/
*.swp
*.swo
*~

# Jupyter Notebook
.ipynb_checkpoints

# pyenv
.python-version

# Secrets
.env.local
.env.production
secrets.json
*.key
api_keys.txt

# OS
.DS_Store
Thumbs.db

# Logs
*.log
logs/

# Data files (большие файлы)
data/*.csv
data/*.json
*.pkl
```

---

## Продвинутые техники

### Rebase vs Merge

#### Merge
Сохраняет полную историю слияний:

```bash
git checkout main
git merge feature-branch
```

**Плюсы:**
- Полная история всех операций
- Легко понять, когда и что было слито

**Минусы:**
- История может стать запутанной
- Много лишних коммитов слияния

#### Rebase
Переписывает историю, делая её линейной:

```bash
git checkout feature-branch
git rebase main
```

**Плюсы:**
- Чистая линейная история
- Легче читать `git log`

**Минусы:**
- Переписывает историю (опасно для общих веток)
- Может вызвать конфликты

**Золотое правило:** Никогда не делайте rebase веток, которые уже отправлены на сервер и используются другими людьми!

### Разрешение конфликтов слияния

Конфликты возникают, когда два человека изменили одни и те же строки.

#### Шаг 1: Обнаружение конфликта

```bash
git merge feature-branch
# Auto-merging file.py
# CONFLICT (content): Merge conflict in file.py
# Automatic merge failed; fix conflicts and then commit the result.
```

#### Шаг 2: Просмотр конфликта

Откройте файл в редакторе. Конфликт выглядит так:

```python
def process_user_input(text):
<<<<<<< HEAD
    # Основная ветка: проверка на спам
    if is_spam(text):
        return "Spam detected"
=======
    # Фича-ветка: проверка на токсичность
    if is_toxic(text):
        return "Toxic content"
>>>>>>> feature-branch
    
    return process_text(text)
```

#### Шаг 3: Разрешение

Выберите нужный вариант или объедините оба:

```python
def process_user_input(text):
    # Проверка на спам и токсичность
    if is_spam(text):
        return "Spam detected"
    if is_toxic(text):
        return "Toxic content"
    
    return process_text(text)
```

#### Шаг 4: Завершение

```bash
# Добавьте разрешенный файл
git add file.py

# Завершите слияние
git commit -m "Resolve merge conflict in process_user_input"

# Или при использовании rebase
git rebase --continue
```

### Stash (временное сохранение)

Когда нужно переключиться на другую задачу, не коммитя незавершенные изменения:

```bash
# Сохранить изменения
git stash

# Сохранить с сообщением
git stash save "Work in progress on agent memory"

# Посмотреть список stash
git stash list

# Применить последний stash
git stash apply

# Применить конкретный stash
git stash apply stash@{2}

# Применить и удалить из списка
git stash pop

# Удалить stash
git stash drop stash@{1}

# Очистить все stash
git stash clear
```

### Cherry-pick

Перенос конкретного коммита из одной ветки в другую:

```bash
# Найти хэш коммита
git log --oneline feature-branch

# Перенести коммит в текущую ветку
git cherry-pick abc1234

# Перенести несколько коммитов
git cherry-pick abc1234^..abc1234
```

### Teardown и очистка

```bash
# Удалить слитые локальные ветки
git branch --merged | grep -v "\*\|main\|master" | xargs git branch -d

# Удалить удаленные ветки, которых нет на сервере
git fetch --prune

# Полная очистка
git gc --prune=now
```

### Поиск в истории

```bash
# Найти коммиты по тексту в сообщении
git log --grep="bug fix"

# Найти коммиты по автору
git log --author="John"

# Найти коммиты, изменившие конкретный файл
git log -- filename.py

# Найти, кто изменил конкретную строку
git blame filename.py

# Поиск по содержимому коммита
git log -S "function_name"
```

---

## Решение распространенных проблем

### Проблема 1: "Please tell me who you are"

**Ошибка:**
```
fatal: unable to auto-detect email address
```

**Решение:**
```bash
git config --global user.name "Ваше Имя"
git config --global user.email "your.email@example.com"
```

### Проблема 2: Забыл добавить файлы в коммит

**Решение:**
```bash
# Добавить забытые файлы
git add forgotten_file.py

# Изменить последний коммит
git commit --amend --no-edit
```

### Проблема 3: Отправить не в ту ветку

**Решение:**
```bash
# Отменить последний push (если никто еще не забрал)
git push origin --delete branch-name

# Создать правильную ветку
git checkout -b correct-branch

# Перенести коммиты
git cherry-pick <commit-hash>

# Отправить в правильную ветку
git push -u origin correct-branch
```

### Проблема 4: Конфликт при pull

**Решение:**
```bash
# Отменить слияние
git merge --abort

# Или использовать rebase вместо merge
git pull --rebase origin main
```

### Проблема 5: Случайно закоммитил секретные данные

**Решение (если коммит только что создан):**
```bash
# Удалить файл из коммита
git reset --soft HEAD~1
git rm --cached secret_file.json
echo "secret_file.json" >> .gitignore
git commit -m "Remove secrets and add to gitignore"
git push --force
```

**Если коммитов много — используйте BFG Repo-Cleaner:**
```bash
# Скачать BFG: https://rtyley.github.io/bfg-repo-cleaner/
java -jar bfg.jar --delete-files secret_file.json my-repo.git
```

**⚠️ Важно:** После удаления секретов обязательно:
1. Смените все скомпрометированные ключи
2. Предупредите команду
3. Проверьте историю на наличие других секретов

### Проблема 6: "Changes not staged for commit"

**Ошибка возникает, когда файлы изменены, но не добавлены в стадию.**

**Решение:**
```bash
# Посмотреть, что изменено
git status
git diff

# Добавить изменения
git add .

# Или откатить изменения
git checkout -- filename.py
```

### Проблема 7: Detached HEAD state

**Вы оказались в состоянии "detached HEAD", когда 체크아웃нули конкретный коммит вместо ветки.**

**Решение:**
```bash
# Если нужно сохранить изменения
git checkout -b new-branch-name

# Если можно потерять изменения
git checkout main
```

---

## Best Practices для работы с кодом агентов

### 1. Коммитьте часто и маленькими порциями

✅ Хорошо:
```bash
git add prompts/system_prompt.md
git commit -m "Add system prompt for customer support agent"

git add tools/calculator.py
git commit -m "Implement basic calculator tool"

git add agents/base_agent.py
git commit -m "Create base agent class with LLM integration"
```

❌ Плохо:
```bash
git add .
git commit -m "update everything"
```

### 2. Пишите информативные сообщения коммитов

Используйте формат:
```
<type>(<scope>): <subject>

<body>

<footer>
```

Типы коммитов:
- `feat`: новая функция
- `fix`: исправление бага
- `docs`: изменение документации
- `style`: форматирование кода
- `refactor`: рефакторинг без изменения функционала
- `test`: добавление тестов
- `chore`: обновление зависимостей, настройка сборки

Пример:
```
feat(agent): add memory module for conversation history

Implement short-term memory using Redis cache.
Memory persists for 24 hours and stores last 50 messages.

Closes #123
```

### 3. Используйте ветки для каждой фичи

✅ Правильный подход:
```bash
git checkout -b feature/add-search-tool
# работа над поиском...
git push -u origin feature/add-search-tool
# PR и мерж

git checkout -b fix/prompt-injection-vulnerability
# работа над безопасностью...
```

### 4. Храните секреты отдельно

Никогда не коммитьте API ключи и пароли!

✅ Правильно:
```python
# config/settings.py
import os
from dotenv import load_dotenv

load_dotenv()

OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")
DATABASE_URL = os.getenv("DATABASE_URL")
```

```bash
# .env (добавлен в .gitignore)
OPENAI_API_KEY=sk-...
DATABASE_URL=postgresql://...
```

❌ Неправильно:
```python
# Никогда так не делайте!
OPENAI_API_KEY = "sk-proj-abc123..."
```

### 5. Тестируйте перед коммитом

```bash
# Запустите тесты
pytest

# Проверьте линтером
flake8 agents/

# Проверьте форматирование
black --check agents/

# Только если всё прошло успешно
git add .
git commit -m "feat: add new agent feature"
```

### 6. Документируйте изменения в промптах

При изменении промптов создавайте отдельную ветку и подробно описывайте:

```bash
git checkout -b prompt-improvement/customer-support-v2
git add prompts/customer_support.md
git commit -m "improve(prompts): enhance customer support agent empathy

- Add emotional recognition phase
- Include de-escalation techniques
- Update examples for complex scenarios

A/B test results: +15% satisfaction rate"
```

### 7. Используйте теги для версий

```bash
# Создать тег для релиза
git tag -a v1.0.0 -m "Initial release of customer support agent"

# Отправить теги на сервер
git push origin --tags

# Посмотреть все теги
git tag -l
```

### 8. Регулярно синхронизируйтесь с main

```bash
# Каждое утро или перед началом новой задачи
git checkout main
git pull upstream main
git checkout your-branch
git rebase main
```

---

## Шпаргалка по командам

### Базовые команды

| Команда | Описание |
|---------|----------|
| `git init` | Инициализировать репозиторий |
| `git clone <url>` | Клонировать репозиторий |
| `git status` | Показать статус файлов |
| `git add <file>` | Добавить файл в стадию |
| `git commit -m "msg"` | Создать коммит |
| `git push` | Отправить изменения на сервер |
| `git pull` | Получить и слить изменения |
| `git log` | Показать историю коммитов |

### Ветки

| Команда | Описание |
|---------|----------|
| `git branch` | Список веток |
| `git branch <name>` | Создать ветку |
| `git checkout <name>` | Переключиться на ветку |
| `git checkout -b <name>` | Создать и переключиться |
| `git merge <branch>` | Слить ветку |
| `git branch -d <name>` | Удалить ветку |

### Отмена изменений

| Команда | Описание |
|---------|----------|
| `git restore <file>` | Отменить изменения в файле |
| `git restore --staged <file>` | Убрать файл из стадии |
| `git reset --soft HEAD~1` | Отменить последний коммит |
| `git commit --amend` | Изменить последний коммит |
| `git revert <commit>` | Создать коммит, отменяющий указанный |

### Работа с удаленным репозиторием

| Команда | Описание |
|---------|----------|
| `git remote -v` | Показать удаленные репозитории |
| `git remote add <name> <url>` | Добавить удаленный репозиторий |
| `git fetch <remote>` | Получить изменения без слияния |
| `git push <remote> <branch>` | Отправить ветку |
| `git pull <remote> <branch>` | Получить и слить |

### Полезные утилиты

| Команда | Описание |
|---------|----------|
| `git stash` | Временно сохранить изменения |
| `git cherry-pick <commit>` | Перенести конкретный коммит |
| `git rebase <branch>` | Перебазировать ветку |
| `git blame <file>` | Показать, кто изменил каждую строку |
| `git diff` | Показать различия между версиями |
| `git tag` | Работа с тегами |

### Настройка и информация

| Команда | Описание |
|---------|----------|
| `git config --list` | Показать настройки |
| `git config user.name` | Показать имя пользователя |
| `git log --oneline --graph` | Красивый лог с графом |
| `git show <commit>` | Показать детали коммита |
| `git reflog` | История всех действий |

---

## Дополнительные ресурсы

### Официальная документация
- [Git Documentation](https://git-scm.com/doc)
- [GitHub Docs](https://docs.github.com)
- [GitHub Learning Lab](https://lab.github.com)

### Интерактивные обучающие ресурсы
- [Learn Git Branching](https://learngitbranching.js.org) — визуальное обучение веткам
- [Oh My Git!](https://ohmygit.org) — игра для изучения Git
- [Git Immersion](https://gitimmersion.com) — пошаговое руководство

### Книги
- "Pro Git" by Scott Chacon — бесплатная книга на [git-scm.com/book](https://git-scm.com/book/en/v2)
- "Git Pocket Guide" by Richard E. Silverman

### Видеокурсы
- [Git and GitHub for Poets](https://www.youtube.com/playlist?list=PLRqwX-V7Uu6ZF9C0YMKuns9sLDzK6zoiV) — для начинающих
- [Advanced Git](https://www.atlassian.com/git/tutorials) — от Atlassian

### Шпаргалки
- [GitHub Git Cheat Sheet](https://training.github.com/downloads/github-git-cheat-sheet/)
- [Atlassian Git Cheat Sheet](https://www.atlassian.com/dam/jcr:e7e22f25-bba2-4ef1-a19f-13f89baa73c4/Swansea_Git-cheatsheet_04.pdf)

### Инструменты
- [GitKraken](https://www.gitkraken.com) — GUI клиент для Git
- [SourceTree](https://www.sourcetreeapp.com) — бесплатный GUI клиент
- [GitHub Desktop](https://desktop.github.com) — официальный десктопный клиент
- [Lazygit](https://github.com/jesseduffield/lazygit) — терминальный UI

---

## Практическое задание для закрепления

### Задание 1: Базовый рабочий процесс

1. Создайте новый репозиторий на GitHub под названием `my-first-agent`
2. Клонируйте его локально
3. Создайте структуру проекта:
   ```
   my-first-agent/
   ├── README.md
   ├── requirements.txt
   └── agent.py
   ```
4. Сделайте несколько коммитов с понятными сообщениями
5. Отправьте изменения на GitHub

### Задание 2: Работа с ветками

1. Создайте ветку `feature/add-memory`
2. Добавьте файл `memory.py` с базовой реализацией памяти
3. Сделайте коммит
4. Переключитесь на `main`
5. Создайте ветку `fix/typo`
6. Исправьте опечатку в `README.md`
7. Вернитесь на `main` и слейте `fix/typo`
8. Отправьте обе ветки на сервер

### Задание 3: Pull Request

1. Найдите открытый проект с домашними заданиями курса
2. Создайте форк
3. Выполните задание в своей ветке
4. Создайте Pull Request с подробным описанием
5. Попросите одногруппника сделать code review

### Задание 4: Разрешение конфликтов

1. Создайте две ветки от `main`
2. В обеих ветках измените один и тот же файл в одном месте
3. Попробуйте слить ветки
4. Разрешите конфликт вручную
5. Завершите слияние

---

## Заключение

Умение работать с Git и GitHub — критически важный навык для любого разработчика, особенно в области AI-агентов, где:
- Код быстро меняется и экспериментируется
- Важна возможность отката к рабочим версиям
- Часто требуется сотрудничество с другими разработчиками
- Необходимо управлять версиями промптов и конфигураций

Регулярная практика и следование best practices помогут вам эффективно использовать эти инструменты в учебном проекте и будущей карьере.

**Помните:**
- Коммитьте часто и маленькими порциями
- Пишите понятные сообщения коммитов
- Используйте ветки для изоляции изменений
- Никогда не коммитьте секреты
- Регулярно синхронизируйтесь с основной веткой
- Делайте backup важных изменений

Удачи в изучении Git и создании потрясающих AI-агентов! 🚀
