# 🐧 Linux для работы с LLM и агентами

## Введение

Этот модуль предназначен для быстрого освоения Linux с фокусом на задачи работы с большими языковыми моделями (LLM) и AI-агентами. Вы научитесь всем необходимым командам и концепциям для эффективной разработки, развертывания и управления AI-системами в среде Linux.

---

## Содержание

1. [Основы работы с терминалом](#1-основы-работы-с-терминалом)
2. [Файловая система и навигация](#2-файловая-система-и-навигация)
3. [Работа с файлами и текстом](#3-работа-с-файлами-и-текстом)
4. [Управление процессами](#4-управление-процессами)
5. [Пакетные менеджеры и установка ПО](#5-пакетные-менеджеры-и-установка-по)
6. [Работа с Python и виртуальными окружениями](#6-работа-s-python-и-виртуальными-окружениями)
7. [Git и управление версиями](#7-git-и-управление-версиями)
8. [Сеть и удаленный доступ](#8-сеть-и-удаленный-доступ)
9. [Docker и контейнеризация](#9-docker-и-контейнеризация)
10. [Мониторинг ресурсов](#10-мониторинг-ресурсов)
11. [Автоматизация и скрипты](#11-автоматизация-и-скрипты)
12. [Практические задачи для LLM/AI](#12-практические-задачи-для-llmai)

---

## 1. Основы работы с терминалом

### Открытие терминала
- **GUI**: Ctrl+Alt+T (Ubuntu/Debian) или поиск "Terminal"
- **SSH**: `ssh user@hostname` для удаленного доступа

### Основные команды

```bash
# Показать текущую директорию
pwd

# Очистить экран терминала
clear

# Показать историю команд
history

# Повторить команду из истории по номеру
!номер

# Получить справку по команде
man команда          # Полная документация
команда --help       # Краткая справка
```

### Горячие клавиши терминала

| Клавиши | Действие |
|---------|----------|
| Ctrl+C | Прервать текущий процесс |
| Ctrl+Z | Приостановить процесс |
| Ctrl+D | Завершить ввод / выйти из оболочки |
| Tab | Автодополнение команд и путей |
| Ctrl+R | Поиск по истории команд |
| Ctrl+A | Перейти в начало строки |
| Ctrl+E | Перейти в конец строки |
| Ctrl+U | Удалить от курсора до начала строки |
| Ctrl+K | Удалить от курсора до конца строки |

---

## 2. Файловая система и навигация

### Структура файловой системы Linux

```
/
├── bin/          # Основные исполняемые файлы
├── home/         # Домашние директории пользователей
├── usr/          # Программы и библиотеки
├── etc/          # Конфигурационные файлы
├── var/          # Переменные данные (логи, кэш)
├── tmp/          # Временные файлы
├── opt/          # Дополнительное ПО
└── root/         # Домашняя директория суперпользователя
```

### Команды навигации

```bash
# Перейти в домашнюю директорию
cd

# Перейти в конкретную директорию
cd /path/to/directory

# Вернуться на уровень выше
cd ..

# Вернуться в предыдущую директорию
cd -

# Показать содержимое директории
ls                    # Краткий список
ls -l                 # Подробный список
ls -la                # Все файлы включая скрытые
ls -lh                # С размерами в человекочитаемом формате
ls -lt                # Отсортировать по времени изменения
```

### Работа с путями

```bash
# Абсолютный путь (от корня)
/home/user/projects/llm_agent

# Относительный путь (от текущей директории)
./projects/llm_agent
../other_project

# Тильда обозначает домашнюю директорию
~/projects/llm_agent
```

---

## 3. Работа с файлами и текстом

### Создание и удаление файлов

```bash
# Создать пустой файл
touch filename.txt

# Создать директорию
mkdir directory_name

# Создать директорию с родительскими папками
mkdir -p parent/child/grandchild

# Удалить файл
rm filename.txt

# Удалить директорию (пустую)
rmdir directory_name

# Удалить директорию со всем содержимым (ОСТОРОЖНО!)
rm -rf directory_name
```

### Копирование и перемещение

```bash
# Копировать файл
cp source.txt destination.txt

# Копировать директорию рекурсивно
cp -r source_dir/ destination_dir/

# Переместить/переименовать файл
mv old_name.txt new_name.txt
mv file.txt /new/location/
```

### Просмотр содержимого файлов

```bash
# Вывести содержимое файла
cat filename.txt

# Вывести с номерами строк
cat -n filename.txt

# Постраничный просмотр (листать пробелом, выход q)
less filename.txt
more filename.txt

# Первые N строк
head -n 20 filename.txt

# Последние N строк
tail -n 20 filename.txt

# Следить за обновлениями файла (логи)
tail -f logfile.log
```

### Редактирование файлов

#### nano (простой редактор для начинающих)

```bash
nano filename.txt
```

**Горячие клавиши nano:**
- Ctrl+O: Сохранить
- Ctrl+X: Выйти
- Ctrl+K: Вырезать строку
- Ctrl+U: Вставить
- Ctrl+W: Поиск

#### vim (продвинутый редактор)

```bash
vim filename.txt
```

**Режимы vim:**
- Нормальный режим (Esc): навигация и команды
- Режим вставки (i): редактирование текста
- Командный режим (:): сохранение, выход и др.

**Основные команды vim:**
```
i           # Войти в режим вставки
Esc         # Выйти в нормальный режим
:w          # Сохранить
:q          # Выйти
:q!         # Выйти без сохранения
:wq         # Сохранить и выйти
dd          # Удалить строку
yy          # Копировать строку
p           # Вставить
/search     # Найти текст
n           # Следующее совпадение
```

### Поиск файлов

```bash
# Найти файл по имени
find /path -name "filename.txt"

# Найти файлы по расширению
find . -name "*.py"

# Найти файлы измененные за последние N дней
find . -mtime -7

# Найти файлы больше определенного размера
find . -size +100M
```

### Текстовые утилиты для обработки данных

```bash
# Поиск текста в файлах
grep "pattern" filename.txt
grep -r "pattern" ./directory        # Рекурсивно
grep -i "pattern" filename.txt       # Без учета регистра
grep -v "pattern" filename.txt       # Исключая паттерн
grep -n "pattern" filename.txt       # С номерами строк

# Конвейеры (pipes) - передача вывода одной команды в другую
cat logfile.txt | grep "ERROR" | head -n 10

# Сортировка
sort filename.txt
sort -n filename.txt                 # Числовая сортировка
sort -r filename.txt                 # Обратный порядок

# Подсчет строк, слов, символов
wc filename.txt
wc -l filename.txt                   # Только строки

# Уникальные строки
uniq filename.txt

# Замена текста
sed 's/old/new/g' filename.txt       # Заменить все вхождения
sed -i 's/old/new/g' filename.txt    # Заменить в файле

# Извлечение колонок из текста
cut -d',' -f1 data.csv               # Первая колонка, разделитель запятая
```

---

## 4. Управление процессами

### Просмотр процессов

```bash
# Показать запущенные процессы
ps aux

# Динамический просмотр процессов
top
htop                          # Более удобная версия (требует установки)

# Показать процессы по имени
pgrep python
ps aux | grep python

# Показать дерево процессов
pstree
```

### Запуск и управление процессами

```bash
# Запустить процесс в фоне
python train_model.py &

# Запустить процесс игнорируя разрыв соединения
nohup python train_model.py &

# Использовать screen для сессий
screen -S training_session
python train_model.py
# Ctrl+A, D для открепления
# screen -r training_session для возвращения

# Использовать tmux (альтернатива screen)
tmux new -s training_session
# Ctrl+B, D для открепления
# tmux attach -t training_session для возвращения
```

### Остановка процессов

```bash
# Завершить процесс по PID
kill PID

# Принудительно завершить процесс
kill -9 PID

# Завершить процесс по имени
pkill process_name
killall process_name
```

### Приоритеты процессов

```bash
# Запустить с низким приоритетом (хорошо для фоновых задач)
nice -n 10 python background_task.py

# Изменить приоритет запущенного процесса
renice +5 -p PID
```

---

## 5. Пакетные менеджеры и установка ПО

### Ubuntu/Debian (apt)

```bash
# Обновить список пакетов
sudo apt update

# Обновить установленные пакеты
sudo apt upgrade

# Установить пакет
sudo apt install package_name

# Удалить пакет
sudo apt remove package_name

# Найти пакет
apt search keyword

# Показать информацию о пакете
apt show package_name

# Установить несколько пакетов
sudo apt install git python3-pip docker.io curl wget
```

### CentOS/RHEL/Fedora (dnf/yum)

```bash
sudo dnf install package_name
sudo dnf remove package_name
sudo dnf update
```

### Arch Linux (pacman)

```bash
sudo pacman -S package_name
sudo pacman -R package_name
sudo pacman -Syu
```

### Универсальные методы установки

#### Установка из исходников

```bash
git clone https://github.com/project/repo.git
cd repo
./configure
make
sudo make install
```

#### Установка через pip (Python)

```bash
# Установить пакет
pip install package_name

# Установить конкретную версию
pip install package_name==1.2.3

# Установить из requirements.txt
pip install -r requirements.txt

# Обновить pip
pip install --upgrade pip
```

#### Установка через conda

```bash
# Создать окружение
conda create -n llm_env python=3.10

# Активировать окружение
conda activate llm_env

# Установить пакет
conda install package_name

# Установить из канала conda-forge
conda install -c conda-forge package_name
```

---

## 6. Работа с Python и виртуальными окружениями

### Установка Python

```bash
# Ubuntu/Debian
sudo apt install python3 python3-pip python3-venv

# Проверка версии
python3 --version
pip3 --version
```

### Виртуальные окружения

#### venv (стандартная библиотека)

```bash
# Создать виртуальное окружение
python3 -m venv venv

# Активировать (Linux/Mac)
source venv/bin/activate

# Деактивировать
deactivate

# Установить пакеты в виртуальное окружение
pip install torch transformers langchain
```

#### virtualenv

```bash
# Установить
pip install virtualenv

# Создать окружение
virtualenv venv

# Активировать
source venv/bin/activate
```

#### conda

```bash
# Создать окружение с конкретным Python
conda create -n llm python=3.10

# Активировать
conda activate llm

# Экспорт окружения
conda env export > environment.yml

# Импорт окружения
conda env create -f environment.yml
```

### Управление зависимостями

```bash
# Создать requirements.txt
pip freeze > requirements.txt

# Установить из requirements.txt
pip install -r requirements.txt

# Пример requirements.txt для LLM проекта
# torch>=2.0.0
# transformers>=4.30.0
# langchain>=0.1.0
# openai>=1.0.0
# python-dotenv>=1.0.0
```

### Запуск Python скриптов

```bash
# Запустить скрипт
python script.py

# Запустить с аргументами
python script.py --model llama2 --epochs 10

# Запустить в фоне
python script.py &

# Запустить с перенаправлением вывода
python script.py > output.log 2>&1

# Запустить в виртуальном окружении
source venv/bin/activate && python script.py
```

---

## 7. Git и управление версиями

### Базовые команды Git

```bash
# Инициализировать репозиторий
git init

# Клонировать репозиторий
git clone https://github.com/username/repo.git

# Проверить статус
git status

# Добавить файлы
git add filename.txt
git add .                    # Добавить все

# Сделать коммит
git commit -m "Описание изменений"

# Просмотреть историю
git log
git log --oneline            # Кратко

# Создать ветку
git branch branch_name

# Переключиться на ветку
git checkout branch_name

# Создать и переключиться
git checkout -b new_branch

# Слить ветки
git merge branch_name

# Отправить изменения
git push origin main

# Получить изменения
git pull origin main
```

### Работа с удаленными репозиториями

```bash
# Добавить удаленный репозиторий
git remote add origin https://github.com/username/repo.git

# Посмотреть удаленные репозитории
git remote -v

# Клонировать конкретную ветку
git clone -b branch_name https://github.com/username/repo.git
```

### Отмена изменений

```bash
# Отменить изменения в файле
git checkout -- filename.txt

# Отменить коммит (оставить изменения)
git reset --soft HEAD~1

# Отменить коммит (удалить изменения)
git reset --hard HEAD~1

# Отменить последний запушенный коммит
git revert HEAD
```

### SSH ключи для GitHub/GitLab

```bash
# Создать SSH ключ
ssh-keygen -t ed25519 -C "your_email@example.com"

# Скопировать публичный ключ
cat ~/.ssh/id_ed25519.pub

# Добавить ключ в SSH агент
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

---

## 8. Сеть и удаленный доступ

### Проверка сети

```bash
# Проверить подключение
ping google.com

# Проверить доступность порта
nc -zv hostname port
telnet hostname port

# Показать сетевые интерфейсы
ip addr
ifconfig                     # Устаревшая, но еще используется

# Показать открытые порты
ss -tulpn
netstat -tulpn               # Устаревшая
```

### SSH - удаленный доступ

```bash
# Подключиться к удаленному серверу
ssh user@hostname

# Подключиться на конкретный порт
ssh -p 2222 user@hostname

# Подключиться с использованием ключа
ssh -i ~/.ssh/key.pem user@hostname

# Копировать файл на сервер
scp file.txt user@hostname:/path/to/destination

# Копировать директорию рекурсивно
scp -r directory/ user@hostname:/path/to/destination

# Копировать с сервера
scp user@hostname:/path/to/file.txt ./local_path/

# Проброс портов (например, для доступа к Jupyter)
ssh -L 8888:localhost:8888 user@hostname
```

### Конфигурация SSH

```bash
# Файл конфигурации ~/.ssh/config
Host myserver
    HostName 192.168.1.100
    User myuser
    IdentityFile ~/.ssh/mykey.pem
    Port 2222

# Теперь можно подключаться просто:
ssh myserver
```

### Wget и Curl - загрузка файлов

```bash
# Скачать файл
wget https://example.com/file.zip
curl -O https://example.com/file.zip

# Скачать с распаковкой
wget https://example.com/file.tar.gz
tar -xzf file.tar.gz

# Получить информацию о URL
curl -I https://example.com

# Отправить POST запрос
curl -X POST -d "key=value" https://api.example.com/endpoint

# Скачать модель HuggingFace
wget https://huggingface.co/model/path/resolve/main/model.safetensors
```

---

## 9. Docker и контейнеризация

### Установка Docker

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install docker.io docker-compose

# Добавить пользователя в группу docker (чтобы не использовать sudo)
sudo usermod -aG docker $USER
# Перезайти в систему

# Проверить установку
docker --version
docker run hello-world
```

### Основные команды Docker

```bash
# Скачать образ
docker pull ubuntu:22.04
docker pull pytorch/pytorch:2.0.1-cuda11.7-cudnn8-runtime

# Показать загруженные образы
docker images

# Запустить контейнер
docker run -it ubuntu:22.04 bash
docker run -d nginx                 # В фоне

# Запустить с пробросом порта
docker run -p 8080:80 nginx

# Запустить с монтированием тома
docker run -v $(pwd):/workspace -it python:3.10 bash

# Запустить с переменными окружения
docker run -e API_KEY=secret -e MODEL_NAME=llama2 myimage

# Показать запущенные контейнеры
docker ps
docker ps -a                        # Все контейнеры

# Остановить контейнер
docker stop container_id

# Удалить контейнер
docker rm container_id

# Удалить образ
docker rmi image_id

# Войти в запущенный контейнер
docker exec -it container_id bash

# Показать логи контейнера
docker logs container_id
docker logs -f container_id         # Следить в реальном времени
```

### Создание своего образа (Dockerfile)

```dockerfile
# Пример Dockerfile для LLM проекта
FROM python:3.10-slim

WORKDIR /app

# Установить системные зависимости
RUN apt-get update && apt-get install -y \
    git \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Копировать зависимости
COPY requirements.txt .

# Установить Python пакеты
RUN pip install --no-cache-dir -r requirements.txt

# Копировать код проекта
COPY . .

# Команда запуска
CMD ["python", "main.py"]
```

```bash
# Собрать образ
docker build -t my-llm-app .

# Запустить собранный образ
docker run -it my-llm-app

# Запустить с GPU (требуется nvidia-container-toolkit)
docker run --gpus all -it my-llm-app
```

### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  llm-app:
    build: .
    ports:
      - "8000:8000"
    volumes:
      - .:/app
      - model_cache:/root/.cache/huggingface
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]

  redis:
    image: redis:alpine
    ports:
      - "6379:6379"

volumes:
  model_cache:
```

```bash
# Запустить все сервисы
docker-compose up

# Запустить в фоне
docker-compose up -d

# Остановить все сервисы
docker-compose down

# Просмотреть логи
docker-compose logs -f
```

---

## 10. Мониторинг ресурсов

### Использование памяти и CPU

```bash
# Общая информация о системе
uname -a
hostnamectl

# Информация о CPU
lscpu
cat /proc/cpuinfo

# Информация о памяти
free -h
cat /proc/meminfo

# Информация о дисках
df -h
du -sh directory_name
du -h --max-depth=1 directory_name

# Динамический мониторинг
top
htop                          # Более удобный аналог
watch -n 1 free -h            # Обновлять каждую секунду
```

### Мониторинг дискового пространства

```bash
# Найти большие файлы
find / -type f -size +1G 2>/dev/null

# Найти самые большие директории
du -ah /home | sort -rh | head -n 20

# Очистить кэш apt
sudo apt clean

# Очистить старые ядра
sudo apt autoremove

# Очистить журналы
sudo journalctl --vacuum-time=1d
```

### Мониторинг GPU (NVIDIA)

```bash
# Установить утилиты NVIDIA
sudo apt install nvidia-utils-525

# Показать информацию о GPU
nvidia-smi

# Непрерывный мониторинг
watch -n 1 nvidia-smi

# Детальная информация
nvidia-smi -q
```

### Логирование

```bash
# Системные логи
journalctl
journalctl -xe                  # С подробностями
journalctl -f                   # Следить в реальном времени
journalctl -u service_name      # Логи конкретного сервиса

# Логи приложений
/var/log/syslog
/var/log/auth.log               # Логи аутентификации
dmesg | less                    # Логи ядра
```

---

## 11. Автоматизация и скрипты

### Bash скрипты

```bash
#!/bin/bash
# Пример скрипта для запуска обучения модели

# Переменные
MODEL_NAME="llama2-7b"
EPOCHS=10
BATCH_SIZE=32

# Создание директорий
mkdir -p models/${MODEL_NAME}
mkdir -p logs

# Запуск обучения
echo "Запуск обучения ${MODEL_NAME}..."
python train.py \
    --model ${MODEL_NAME} \
    --epochs ${EPOCHS} \
    --batch_size ${BATCH_SIZE} \
    > logs/train_$(date +%Y%m%d_%H%M%S).log 2>&1

# Проверка успешности
if [ $? -eq 0 ]; then
    echo "Обучение завершено успешно!"
    # Копирование модели в архив
    cp -r models/${MODEL_NAME} /backup/models/
else
    echo "Ошибка при обучении!"
    exit 1
fi
```

```bash
# Сделать скрипт исполняемым
chmod +x script.sh

# Запустить скрипт
./script.sh
bash script.sh
```

### Переменные окружения

```bash
# Установить переменную
export API_KEY="your_secret_key"
export MODEL_PATH="/models/llama2"

# Посмотреть все переменные
env
printenv

# Посмотреть конкретную переменную
echo $API_KEY

# Сохранить в .bashrc для постоянного использования
echo 'export API_KEY="your_secret_key"' >> ~/.bashrc
source ~/.bashrc

# Использовать в команде
API_KEY=secret python app.py
```

### Планировщик задач (cron)

```bash
# Редактировать crontab
crontab -e

# Примеры расписаний:
# Запуск каждый день в 3:00
0 3 * * * /path/to/script.sh

# Запуск каждый час
0 * * * * /path/to/script.sh

# Запуск каждые 5 минут
*/5 * * * * /path/to/script.sh

# Запуск каждый понедельник в 9:00
0 9 * * 1 /path/to/script.sh

# Перезапуск сервиса каждую ночь
0 0 * * * sudo systemctl restart my-service
```

### Алиасы (сокращения команд)

```bash
# Добавить в ~/.bashrc
alias ll='ls -la'
alias gs='git status'
alias gp='git push'
alias dockerps='docker ps -a'
alias htop='htop -o PERCENT_CPU'

# Применить изменения
source ~/.bashrc
```

---

## 12. Практические задачи для LLM/AI

### Задача 1: Настройка рабочего окружения

```bash
# Создать структуру проекта
mkdir -p ~/projects/llm_agent/{src,data,models,logs,notebooks}
cd ~/projects/llm_agent

# Создать виртуальное окружение
python3 -m venv venv
source venv/bin/activate

# Установить основные пакеты
pip install torch transformers langchain openai python-dotenv jupyter

# Создать .env файл для секретов
cat > .env << EOF
OPENAI_API_KEY=your_key_here
HUGGINGFACE_TOKEN=your_token_here
MODEL_PATH=./models
EOF

# Добавить .env в .gitignore
echo ".env" >> .gitignore
```

### Задача 2: Запуск обучения в фоне

```bash
# Использовать screen для длительной сессии
screen -S training

# Внутри screen запустить обучение
python train.py --config config.yaml

# Открепиться: Ctrl+A, D

# Позже вернуться
screen -r training

# Альтернатива с nohup
nohup python train.py > training.log 2>&1 &

# Следить за прогрессом
tail -f training.log
```

### Задача 3: Мониторинг использования GPU

```bash
# Создать скрипт мониторинга
cat > monitor_gpu.sh << 'EOF'
#!/bin/bash
while true; do
    clear
    echo "=== GPU Status ==="
    nvidia-smi
    echo ""
    echo "=== Memory Usage ==="
    free -h
    echo ""
    echo "=== Top Processes ==="
    ps aux --sort=-%mem | head -n 10
    sleep 5
done
EOF

chmod +x monitor_gpu.sh
./monitor_gpu.sh
```

### Задача 4: Автоматическое резервное копирование моделей

```bash
# Скрипт бэкапа
cat > backup_models.sh << 'EOF'
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backup/models_${DATE}"
SOURCE_DIR="$HOME/projects/llm_agent/models"

mkdir -p $BACKUP_DIR
rsync -av --progress $SOURCE_DIR/ $BACKUP_DIR/

# Сжать архив
tar -czf ${BACKUP_DIR}.tar.gz $BACKUP_DIR

# Удалить оригинал
rm -rf $BACKUP_DIR

# Удалить старые бэкапы ( старше 7 дней)
find /backup -name "models_*.tar.gz" -mtime +7 -delete

echo "Backup completed: ${BACKUP_DIR}.tar.gz"
EOF

chmod +x backup_models.sh

# Добавить в cron для ежедневного выполнения
echo "0 2 * * * $HOME/backup_models.sh" | crontab -
```

### Задача 5: Развертывание API с Docker

```bash
# Dockerfile для FastAPI приложения
cat > Dockerfile << 'EOF'
FROM python:3.10-slim

WORKDIR /app

RUN apt-get update && apt-get install -y git curl

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
EOF

# Сборка и запуск
docker build -t llm-api .
docker run -d -p 8000:8000 --name llm-container llm-api

# Проверка
curl http://localhost:8000/docs
```

### Задача 6: Обработка логов для анализа

```bash
# Найти все ошибки в логах
grep -i "error" logs/*.log | head -n 50

# Посчитать количество запросов по эндпоинтам
cat access.log | cut -d' ' -f7 | sort | uniq -c | sort -rn

# Найти медленные запросы (>1с)
awk '$NF > 1 {print}' access.log

# Извлечь временные метки ошибок
grep "ERROR" logs/app.log | cut -d' ' -f1-4

# Создать отчет
cat logs/app.log | grep "ERROR" | wc -l > error_count.txt
```

### Задача 7: Управление несколькими моделями

```bash
# Скрипт для переключения между моделями
cat > switch_model.sh << 'EOF'
#!/bin/bash

MODELS_DIR="$HOME/models"
ACTIVE_LINK="$MODELS_DIR/active"

if [ -z "$1" ]; then
    echo "Использование: $0 <model_name>"
    echo "Доступные модели:"
    ls -1 $MODELS_DIR
    exit 1
fi

MODEL_NAME=$1

if [ ! -d "$MODELS_DIR/$MODEL_NAME" ]; then
    echo "Модель $MODEL_NAME не найдена!"
    exit 1
fi

# Удалить старую ссылку
rm -f $ACTIVE_LINK

# Создать новую ссылку
ln -s "$MODELS_DIR/$MODEL_NAME" $ACTIVE_LINK

echo "Активная модель переключена на: $MODEL_NAME"
ls -l $ACTIVE_LINK
EOF

chmod +x switch_model.sh

# Использование
./switch_model.sh llama2-7b
./switch_model.sh mistral-7b
```

---

## Дополнительные ресурсы

### Полезные команды для ежедневной работы

```bash
# Быстрый поиск команды в истории
Ctrl+R, начать печатать

# Быстрое редактирование длинного пути
Esc, .  (вставляет последний аргумент предыдущей команды)

# Повторить последнюю команду с sudo
sudo !!

# Распаковать любой архив
extract() {
    if [ -f $1 ] ; then
        case $1 in
            *.tar.bz2)   tar xjf $1     ;;
            *.tar.gz)    tar xzf $1     ;;
            *.bz2)       bunzip2 $1     ;;
            *.rar)       unrar e $1     ;;
            *.gz)        gunzip $1      ;;
            *.tar)       tar xf $1      ;;
            *.tbz2)      tar xjf $1     ;;
            *.tgz)       tar xzf $1     ;;
            *.zip)       unzip $1       ;;
            *)           echo "'$1' cannot be extracted" ;;
        esac
    else
        echo "'$1' is not a valid file"
    fi
}
```

### Рекомендованные пакеты для LLM разработки

```bash
# Системные утилиты
sudo apt install -y \
    git \
    curl \
    wget \
    htop \
    tmux \
    jq \              # Для работы с JSON
    tree \            # Визуализация структуры директорий
    ripgrep \         # Быстрый grep
    fd-find \         # Быстрый find
    bat \             # Улучшенный cat
    ncdu              # Анализ дискового пространства

# Python инструменты
pip install \
    jupyterlab \
    ipython \
    black \           # Форматирование кода
    flake8 \          # Линтер
    pytest \          # Тестирование
    rich \            # Красивый вывод в терминале
    tqdm              # Прогресс бары
```

### Шпаргалка по правам доступа

```bash
# Посмотреть права
ls -l

# Пример: -rwxr-xr--
# Первый символ: тип файла (- файл, d директория, l ссылка)
# Далее 9 символов: права для владельца, группы, остальных
# r=4, w=2, x=1

# Изменить права
chmod 755 script.sh      # rwxr-xr-x
chmod +x script.sh       # Добавить исполнение
chmod -w file.txt        # Убрать запись

# Изменить владельца
sudo chown user:group file.txt

# Изменить владельца рекурсивно
sudo chown -R user:group directory/
```

---

## Заключение

Этот модуль покрыл основные аспекты работы с Linux, необходимые для эффективной разработки и развертывания LLM и AI-агентов. Ключевые навыки:

✅ Навигация и работа с файловой системой  
✅ Управление процессами и фоновые задачи  
✅ Работа с Python и виртуальными окружениями  
✅ Git и контроль версий  
✅ Docker и контейнеризация  
✅ Мониторинг ресурсов (CPU, GPU, память)  
✅ Автоматизация через скрипты и cron  
✅ Удаленный доступ через SSH  

**Следующие шаги:**
1. Практикуйте команды ежедневно
2. Создайте свои скрипты для автоматизации рутинных задач
3. Освойте Docker для воспроизводимости окружения
4. Настройте мониторинг для ваших AI-приложений
5. Изучите продвинутые темы: Kubernetes, CI/CD, облачные платформы

Удачи в изучении Linux и разработке AI-систем! 🚀
