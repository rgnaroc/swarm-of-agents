# 🐍 PYTHON И GOOGLE COLAB: ПОЛНОЕ РУКОВОДСТВО ДЛЯ РАБОТЫ С AI-АГЕНТАМИ

## Введение

Этот материал предназначен для освоения базовых и продвинутых навыков работы с Python в среде Google Colab, необходимых для эффективной разработки и тестирования AI-агентов в рамках курса.

---

## ЧАСТЬ 1: ОСНОВЫ PYTHON ДЛЯ AI-РАЗРАБОТКИ

### 1.1 Базовый синтаксис

#### Переменные и типы данных

```python
# Основные типы данных
name = "Агент"              # строка (str)
version = 1.0               # число с плавающей точкой (float)
tasks_completed = 42        # целое число (int)
is_active = True            # булево значение (bool)
capabilities = ["search", "calculate", "reason"]  # список (list)
config = {"temperature": 0.7, "max_tokens": 1000} # словарь (dict)

# Проверка типа
print(type(name))           # <class 'str'>
print(isinstance(version, float))  # True
```

#### Работа со строками

```python
# f-строки (форматирование)
agent_name = "GPT-4"
task = "анализ данных"
prompt = f"Ты — {agent_name}. Твоя задача: {task}."
print(prompt)

# Методы строк
text = "  Hello World  "
print(text.strip())         # "Hello World"
print(text.lower())         # "  hello world  "
print(text.upper())         # "  HELLO WORLD  "
print(text.replace("World", "Agent"))  # "  Hello Agent  "

# Разбиение и объединение
tags = "ai,agent,llm,python"
tag_list = tags.split(",")  # ['ai', 'agent', 'llm', 'python']
joined = " | ".join(tag_list)  # "ai | agent | llm | python"
```

#### Списки и словари

```python
# Списки (lists)
tools = ["calculator", "search", "calendar"]
tools.append("email")       # Добавить элемент
tools.remove("search")      # Удалить элемент
first_tool = tools[0]       # Доступ по индексу
subset = tools[1:3]         # Срез [1, 3)
length = len(tools)         # Длина списка

# List comprehension (генератор списков)
numbers = [1, 2, 3, 4, 5]
squared = [x**2 for x in numbers]  # [1, 4, 9, 16, 25]
even = [x for x in numbers if x % 2 == 0]  # [2, 4]

# Словари (dicts)
agent_config = {
    "name": "Assistant",
    "model": "gpt-4o-mini",
    "temperature": 0.7,
    "tools": ["search", "calc"]
}

# Доступ к значениям
print(agent_config["name"])           # "Assistant"
print(agent_config.get("model"))      # "gpt-4o-mini"
print(agent_config.get("timeout", 30)) # 30 (значение по умолчанию)

# Перебор словаря
for key, value in agent_config.items():
    print(f"{key}: {value}")

# Добавление и удаление
agent_config["max_iterations"] = 10
del agent_config["temperature"]
```

#### Условия и циклы

```python
# Условные операторы
response_time = 150

if response_time < 100:
    status = "отлично"
elif response_time < 500:
    status = "нормально"
else:
    status = "медленно"

# Цикл for
messages = ["Привет", "Как дела?", "Что нового?"]
for i, msg in enumerate(messages):
    print(f"Сообщение {i+1}: {msg}")

# Цикл while
counter = 0
while counter < 5:
    print(f"Итерация {counter}")
    counter += 1

# break и continue
for i in range(10):
    if i == 3:
        continue    # Пропустить эту итерацию
    if i == 7:
        break       # Выйти из цикла
    print(i)
```

### 1.2 Функции

```python
# Базовая функция
def greet_agent(name, greeting="Привет"):
    """Приветствие агента"""
    return f"{greeting}, {name}!"

print(greet_agent("Alex"))           # "Привет, Alex!"
print(greet_agent("Alex", "Здравствуй"))  # "Здравствуй, Alex!"

# Функция с произвольным количеством аргументов
def sum_all(*args):
    return sum(args)

print(sum_all(1, 2, 3, 4))  # 10

# Функция с именованными аргументами
def create_agent(**kwargs):
    agent = {"type": "assistant"}
    agent.update(kwargs)
    return agent

agent = create_agent(name="Bot", model="gpt-4", tools=["search"])
print(agent)  # {'type': 'assistant', 'name': 'Bot', 'model': 'gpt-4', 'tools': ['search']}

# Lambda-функции (анонимные)
square = lambda x: x ** 2
print(square(5))  # 25

# Использование с map/filter
numbers = [1, 2, 3, 4, 5]
doubled = list(map(lambda x: x * 2, numbers))  # [2, 4, 6, 8, 10]
evens = list(filter(lambda x: x % 2 == 0, numbers))  # [2, 4]
```

### 1.3 Обработка ошибок

```python
# try-except блок
def safe_divide(a, b):
    try:
        result = a / b
        return result
    except ZeroDivisionError:
        return "Ошибка: деление на ноль"
    except TypeError:
        return "Ошибка: некорректный тип данных"
    except Exception as e:
        return f"Неизвестная ошибка: {e}"

print(safe_divide(10, 2))  # 5.0
print(safe_divide(10, 0))  # "Ошибка: деление на ноль"

# finally выполняется всегда
def read_file(filename):
    file = None
    try:
        file = open(filename, 'r')
        content = file.read()
        return content
    except FileNotFoundError:
        return "Файл не найден"
    finally:
        if file:
            file.close()

# Выброс исключения
def validate_age(age):
    if age < 0:
        raise ValueError("Возраст не может быть отрицательным")
    return age
```

### 1.4 Работа с JSON

```python
import json

# Python → JSON (сериализация)
data = {
    "agent": {
        "name": "Helper",
        "version": 1.0,
        "active": True,
        "tools": ["search", "calc"]
    }
}
json_string = json.dumps(data, indent=2, ensure_ascii=False)
print(json_string)

# JSON → Python (десериализация)
json_input = '''
{
    "task": "analyze",
    "priority": 1,
    "completed": false
}
'''
parsed = json.loads(json_input)
print(parsed["task"])  # "analyze"

# Чтение/запись JSON файлов
with open('config.json', 'w', encoding='utf-8') as f:
    json.dump(data, f, indent=2, ensure_ascii=False)

with open('config.json', 'r', encoding='utf-8') as f:
    loaded_data = json.load(f)
```

### 1.5 Классы и ООП

```python
# Базовый класс
class AIAgent:
    def __init__(self, name, model="gpt-4o-mini"):
        self.name = name
        self.model = model
        self.conversation_history = []
        self.tools = []
    
    def add_message(self, role, content):
        """Добавить сообщение в историю"""
        self.conversation_history.append({
            "role": role,
            "content": content
        })
    
    def add_tool(self, tool):
        """Добавить инструмент"""
        self.tools.append(tool)
    
    def get_context(self):
        """Получить контекст диалога"""
        return self.conversation_history
    
    def clear_history(self):
        """Очистить историю"""
        self.conversation_history = []
    
    def __str__(self):
        return f"AIAgent(name={self.name}, model={self.model})"

# Использование
agent = AIAgent("Assistant", "gpt-4")
agent.add_message("user", "Привет!")
agent.add_message("assistant", "Здравствуйте!")
print(agent)  # AIAgent(name=Assistant, model=gpt-4)
print(agent.get_context())

# Наследование
class ToolUsingAgent(AIAgent):
    def __init__(self, name, model="gpt-4o-mini"):
        super().__init__(name, model)
        self.tool_results = []
    
    def execute_tool(self, tool_name, **kwargs):
        """Выполнить инструмент"""
        # Здесь будет логика выполнения
        result = f"Результат выполнения {tool_name}"
        self.tool_results.append(result)
        return result
```

---

## ЧАСТЬ 2: БИБЛИОТЕКИ ДЛЯ РАБОТЫ С AI

### 2.1 Установка библиотек

```python
# В Google Colab используйте ! перед командой
!pip install openai
!pip install langchain
!pip install requests
!pip install beautifulsoup4

# Проверка установленных версий
import openai
print(openai.__version__)
```

### 2.2 Работа с OpenAI API

```python
from openai import OpenAI
import os

# Инициализация клиента
client = OpenAI(api_key="your-api-key-here")

# Простой запрос
response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {"role": "system", "content": "Ты полезный ассистент."},
        {"role": "user", "content": "Объясни, что такое AI-агент?"}
    ],
    temperature=0.7,
    max_tokens=500
)

print(response.choices[0].message.content)

# Работа с историей диалога
messages = [
    {"role": "system", "content": "Ты — эксперт по программированию."}
]

def chat(user_input):
    messages.append({"role": "user", "content": user_input})
    
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=messages,
        temperature=0.7
    )
    
    assistant_message = response.choices[0].message.content
    messages.append({"role": "assistant", "content": assistant_message})
    
    return assistant_message

# Пример использования
print(chat("Как создать функцию в Python?"))
print(chat("А как добавить аннотации типов?"))
```

### 2.3 Обработка функций (Function Calling)

```python
import json

# Определение доступных функций
functions = [
    {
        "name": "calculate",
        "description": "Выполняет математические вычисления",
        "parameters": {
            "type": "object",
            "properties": {
                "expression": {
                    "type": "string",
                    "description": "Математическое выражение, например '2 + 2'"
                }
            },
            "required": ["expression"]
        }
    },
    {
        "name": "get_current_time",
        "description": "Возвращает текущее время",
        "parameters": {
            "type": "object",
            "properties": {},
            "required": []
        }
    }
]

# Реализация функций
def calculate(expression: str) -> str:
    allowed = set('0123456789+-*/.() ')
    if not all(c in allowed for c in expression):
        return "Ошибка: недопустимые символы"
    try:
        result = eval(expression)
        return str(result)
    except Exception as e:
        return f"Ошибка вычисления: {e}"

def get_current_time() -> str:
    from datetime import datetime
    return datetime.now().strftime("%Y-%m-%d %H:%M:%S")

# Словарь функций
available_functions = {
    "calculate": calculate,
    "get_current_time": get_current_time
}

# Обработка вызова функции
def process_with_function_call(user_message):
    messages = [
        {"role": "system", "content": "Используй инструменты когда нужно."},
        {"role": "user", "content": user_message}
    ]
    
    # Первый запрос
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=messages,
        functions=functions,
        function_call="auto"
    )
    
    message = response.choices[0].message
    
    # Если модель хочет вызвать функцию
    if message.function_call:
        function_name = message.function_call.name
        function_args = json.loads(message.function_call.arguments)
        
        # Вызов функции
        function_to_call = available_functions[function_name]
        function_response = function_to_call(**function_args)
        
        # Добавляем результат в диалог
        messages.append(message)
        messages.append({
            "role": "function",
            "name": function_name,
            "content": function_response
        })
        
        # Получаем финальный ответ
        final_response = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=messages
        )
        
        return final_response.choices[0].message.content
    
    return message.content
```

---

## ЧАСТЬ 3: GOOGLE COLAB — ПОЛНОЕ РУКОВОДСТВО

### 3.1 Начало работы

#### Что такое Google Colab?

Google Colab (Colaboratory) — это бесплатная облачная среда для работы с Python, которая:
- Не требует установки ПО (работает в браузере)
- Предоставляет бесплатный доступ к GPU и TPU
- Интегрируется с Google Drive
- Идеально подходит для экспериментов с AI/ML

#### Создание первого ноутбука

1. Перейдите на [colab.research.google.com](https://colab.research.google.com)
2. Нажмите "Новый блокнот" (New Notebook)
3. Дайте название файлу (Файл → Переименовать)

### 3.2 Ячейки и их типы

```
┌─────────────────────────────────────┐
│  [▶] Код                            │  ← Ячейка кода
│  ─────────────────────────────────  │
│  print("Hello, Colab!")             │
│                                     │
│  Hello, Colab!                      │  ← Вывод
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│  ## Заголовок                       │  ← Текстовая ячейка (Markdown)
│  Текст с **форматированием**        │
└─────────────────────────────────────┘
```

#### Горячие клавиши

| Действие | Windows/Linux | Mac |
|----------|---------------|-----|
| Выполнить ячейку | Ctrl+Enter | Cmd+Enter |
| Выполнить и перейти к следующей | Shift+Enter | Shift+Enter |
| Добавить ячейку кода выше | Ctrl+M A | Cmd+M A |
| Добавить ячейку кода ниже | Ctrl+M B | Cmd+M B |
| Удалить ячейку | Ctrl+M D | Cmd+M D |
| Преобразовать в Markdown | Ctrl+M M | Cmd+M M |
| Преобразовать в код | Ctrl+M Y | Cmd+M Y |

### 3.3 Магические команды Colab

```python
# Время выполнения ячейки
%%time
result = sum(range(1000000))
print(result)

# Время выполнения каждой строки
%%timeit
x = [i**2 for i in range(1000)]

# Информация о системе
!python --version
!pip --version

# Список файлов в директории
!ls -la

# Проверка переменных окружения
%env

# Измерение памяти
%memit result = [i**2 for i in range(1000000)]

# Профилирование кода
%prun sum([i**2 for i in range(1000)])
```

### 3.4 Установка и управление пакетами

```python
# Установка пакетов
!pip install openai langchain requests

# Установка конкретной версии
!pip install openai==1.12.0

# Обновление пакетов
!pip install --upgrade openai

# uninstall пакетов
!pip uninstall package-name

# Список установленных пакетов
!pip list

# Сохранение зависимостей
!pip freeze > requirements.txt
```

### 3.5 Работа с файлами

```python
# Создание файла
with open('test.txt', 'w') as f:
    f.write('Привет, Colab!')

# Чтение файла
with open('test.txt', 'r') as f:
    content = f.read()
    print(content)

# Проверка существования файла
import os
if os.path.exists('test.txt'):
    print("Файл существует")

# Удаление файла
os.remove('test.txt')

# Работа с директориями
os.makedirs('data/subfolder', exist_ok=True)
os.chdir('data')
current_dir = os.getcwd()
print(current_dir)
```

### 3.6 Монтирование Google Drive

```python
from google.colab import drive
drive.mount('/content/drive')

# После монтирования файлы доступны по пути:
# /content/drive/MyDrive/...

# Копирование файлов из Drive
!cp /content/drive/MyDrive/data/config.json ./

# Сохранение результатов в Drive
with open('/content/drive/MyDrive/results/output.txt', 'w') as f:
    f.write('Результаты работы агента')
```

### 3.7 Загрузка и скачивание файлов

```python
# Загрузка файлов через интерфейс
from google.colab import files

# Загрузить файл с компьютера
uploaded = files.upload()

for filename in uploaded.keys():
    print(f'Загружен файл: {filename}')

# Скачивание файла на компьютер
with open('output.json', 'w') as f:
    f.write('{"result": "success"}')

files.download('output.json')
```

### 3.8 Использование GPU/TPU

```python
# Проверка доступности GPU
import tensorflow as tf
print("GPU доступен:", tf.config.list_physical_devices('GPU'))

# Или через torch
import torch
print("CUDA доступен:", torch.cuda.is_available())
print("Версия CUDA:", torch.version.cuda)

# Переключение на GPU:
# Меню → Среда выполнения → Изменить тип среды выполнения → GPU

# Информация о GPU
!nvidia-smi
```

---

## ЧАСТЬ 4: ПРАКТИЧЕСКИЕ ПРИМЕРЫ ДЛЯ КУРСА

### 4.1 Шаблон для работы с AI-агентом

```python
# ╔══════════════════════════════════════════════════════════╗
# ║   ШАБЛОН: БАЗОВЫЙ AI-АГЕНТ В GOOGLE COLAB               ║
# ╚══════════════════════════════════════════════════════════╝

# Шаг 1: Установка зависимостей
!pip install openai

# Шаг 2: Импорт библиотек
from openai import OpenAI
import json
import os

# Шаг 3: Настройка API ключа (безопасное хранение)
# Вариант 1: Через secrets Colab (рекомендуется)
from google.colab import userdata
API_KEY = userdata.get('OPENAI_API_KEY')

# Вариант 2: Прямое указание (не рекомендуется для продакшена)
# API_KEY = "sk-..."

client = OpenAI(api_key=API_KEY)

# Шаг 4: Определение инструментов агента
tools = [
    {
        "type": "function",
        "function": {
            "name": "calculate",
            "description": "Выполняет математические вычисления",
            "parameters": {
                "type": "object",
                "properties": {
                    "expression": {
                        "type": "string",
                        "description": "Математическое выражение"
                    }
                },
                "required": ["expression"]
            }
        }
    }
]

# Шаг 5: Реализация функций
def execute_calculate(expression: str) -> str:
    try:
        allowed = set('0123456789+-*/.() ')
        if not all(c in allowed for c in expression):
            return "Ошибка: недопустимые символы"
        result = eval(expression)
        return str(result)
    except Exception as e:
        return f"Ошибка: {e}"

function_map = {
    "calculate": execute_calculate
}

# Шаг 6: Основной цикл агента
class SimpleAgent:
    def __init__(self, system_prompt="Ты полезный ассистент."):
        self.messages = [
            {"role": "system", "content": system_prompt}
        ]
        self.client = client
        self.tools = tools
        self.function_map = function_map
    
    def chat(self, user_input):
        self.messages.append({"role": "user", "content": user_input})
        
        response = self.client.chat.completions.create(
            model="gpt-4o-mini",
            messages=self.messages,
            tools=self.tools,
            tool_choice="auto"
        )
        
        message = response.choices[0].message
        
        # Обработка вызова инструментов
        while message.tool_calls:
            self.messages.append(message)
            
            for tool_call in message.tool_calls:
                function_name = tool_call.function.name
                function_args = json.loads(tool_call.function.arguments)
                
                function_result = self.function_map[function_name](**function_args)
                
                self.messages.append({
                    "role": "tool",
                    "tool_call_id": tool_call.id,
                    "name": function_name,
                    "content": function_result
                })
            
            response = self.client.chat.completions.create(
                model="gpt-4o-mini",
                messages=self.messages,
                tools=self.tools
            )
            message = response.choices[0].message
        
        self.messages.append(message)
        return message.content

# Шаг 7: Использование
agent = SimpleAgent()
response = agent.chat("Сколько будет 123 * 456?")
print(response)
```

### 4.2 Шаблон для тестирования промптов

```python
# ╔══════════════════════════════════════════════════════════╗
# ║   ШАБЛОН: ТЕСТИРОВАНИЕ ПРОМПТОВ                         ║
# ╚══════════════════════════════════════════════════════════╝

from openai import OpenAI
from google.colab import userdata
import pandas as pd

client = OpenAI(api_key=userdata.get('OPENAI_API_KEY'))

# Набор тестовых запросов
test_cases = [
    {"input": "Привет!", "expected_type": "greeting"},
    {"input": "Реши: 2 + 2 * 2", "expected_type": "calculation"},
    {"input": "Какая погода в Москве?", "expected_type": "info_request"},
]

# Системные промпты для тестирования
system_prompts = {
    "friendly": "Ты дружелюбный помощник. Отвечай кратко и позитивно.",
    "professional": "Ты профессиональный ассистент. Отвечай точно и формально.",
    "creative": "Ты креативный помощник. Используй аналогии и метафоры."
}

# Функция тестирования
def test_prompt(system_prompt, test_case):
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": test_case["input"]}
        ],
        temperature=0.7
    )
    return response.choices[0].message.content

# Запуск тестов
results = []
for prompt_name, prompt_text in system_prompts.items():
    for test in test_cases:
        response = test_prompt(prompt_text, test)
        results.append({
            "prompt_style": prompt_name,
            "input": test["input"],
            "expected_type": test["expected_type"],
            "response": response
        })

# Отображение результатов
df = pd.DataFrame(results)
print(df.to_string())

# Сохранение результатов
df.to_csv('prompt_test_results.csv', index=False, encoding='utf-8-sig')
print("\nРезультаты сохранены в prompt_test_results.csv")
```

### 4.3 Шаблон для многошаговых задач

```python
# ╔══════════════════════════════════════════════════════════╗
# ║   ШАБЛОН: МНОГОШАГОВЫЕ ЗАДАЧИ (CHAIN OF THOUGHT)        ║
# ╚══════════════════════════════════════════════════════════╝

from openai import OpenAI
from google.colab import userdata
import json

client = OpenAI(api_key=userdata.get('OPENAI_API_KEY'))

class ChainOfThoughtAgent:
    def __init__(self):
        self.messages = []
    
    def solve_step_by_step(self, problem):
        """Решение задачи по шагам с явным мышлением"""
        
        # Шаг 1: Анализ задачи
        analysis_prompt = f"""
        Проанализируй следующую задачу. Определи:
        1. Что требуется сделать?
        2. Какие данные нужны?
        3. Какие шаги необходимо выполнить?
        
        Задача: {problem}
        
        Ответь в формате JSON:
        {{
            "goal": "...",
            "required_data": [...],
            "steps": ["...", "..."]
        }}
        """
        
        response = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{"role": "user", "content": analysis_prompt}],
            response_format={"type": "json_object"}
        )
        
        plan = json.loads(response.choices[0].message.content)
        print("📋 План решения:")
        print(json.dumps(plan, indent=2, ensure_ascii=False))
        
        # Шаг 2: Выполнение по шагам
        results = []
        for i, step in enumerate(plan["steps"], 1):
            print(f"\n🔹 Шаг {i}: {step}")
            
            step_prompt = f"""
            Выполни шаг {i} из плана: {step}
            
            Полный план: {plan}
            Предыдущие результаты: {results}
            
            Предоставь конкретный результат этого шага.
            """
            
            response = client.chat.completions.create(
                model="gpt-4o-mini",
                messages=[{"role": "user", "content": step_prompt}]
            )
            
            step_result = response.choices[0].message.content
            results.append(step_result)
            print(f"✅ Результат: {step_result[:100]}...")
        
        # Шаг 3: Синтез финального ответа
        synthesis_prompt = f"""
        На основе выполненного плана и полученных результатов,
        предоставь окончательное решение задачи.
        
        Исходная задача: {problem}
        План: {plan}
        Результаты шагов: {results}
        
        Дай полный, структурированный ответ.
        """
        
        response = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{"role": "user", "content": synthesis_prompt}]
        )
        
        final_answer = response.choices[0].message.content
        print("\n🎯 Финальный ответ:")
        print(final_answer)
        
        return {
            "plan": plan,
            "step_results": results,
            "final_answer": final_answer
        }

# Пример использования
agent = ChainOfThoughtAgent()
result = agent.solve_step_by_step(
    "Разработай план создания простого чат-бота для техподдержки"
)
```

### 4.4 Шаблон для работы с памятью агента

```python
# ╔══════════════════════════════════════════════════════════╗
# ║   ШАБЛОН: АГЕНТ С ПАМЯТЬЮ                               ║
# ╚══════════════════════════════════════════════════════════╝

from openai import OpenAI
from google.colab import userdata
import json
from datetime import datetime

client = OpenAI(api_key=userdata.get('OPENAI_API_KEY'))

class AgentWithMemory:
    def __init__(self, max_history=10):
        self.max_history = max_history
        self.short_term_memory = []  # Последние сообщения
        self.long_term_memory = {}   # Важные факты о пользователе
        self.system_prompt = "Ты полезный ассистент с памятью."
    
    def add_to_short_term(self, role, content):
        """Добавление в краткосрочную память"""
        self.short_term_memory.append({
            "role": role,
            "content": content,
            "timestamp": datetime.now().isoformat()
        })
        
        # Ограничение размера
        if len(self.short_term_memory) > self.max_history:
            self.short_term_memory = self.short_term_memory[-self.max_history:]
    
    def extract_facts(self, conversation):
        """Извлечение важных фактов для долгосрочной памяти"""
        prompt = f"""
        Проанализируй диалог и извлеки важные факты о пользователе.
        Факты должны быть полезны для будущих взаимодействий.
        
        Диалог:
        {json.dumps(conversation, ensure_ascii=False)}
        
        Верни факты в формате JSON:
        {{
            "facts": [
                {{"key": "имя", "value": "..."}},
                {{"key": "предпочтения", "value": "..."}}
            ]
        }}
        
        Если фактов нет, верни пустой список.
        """
        
        response = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{"role": "user", "content": prompt}],
            response_format={"type": "json_object"}
        )
        
        facts_data = json.loads(response.choices[0].message.content)
        
        for fact in facts_data.get("facts", []):
            self.long_term_memory[fact["key"]] = fact["value"]
    
    def get_context_with_memory(self):
        """Формирование контекста с учётом памяти"""
        context_messages = [{"role": "system", "content": self.system_prompt}]
        
        # Добавляем известные факты
        if self.long_term_memory:
            facts_text = "\n".join(
                f"- {k}: {v}" for k, v in self.long_term_memory.items()
            )
            context_messages.append({
                "role": "system",
                "content": f"Известные факты о пользователе:\n{facts_text}"
            })
        
        # Добавляем историю диалога
        context_messages.extend(self.short_term_memory)
        
        return context_messages
    
    def chat(self, user_input):
        # Добавляем сообщение пользователя
        self.add_to_short_term("user", user_input)
        
        # Получаем контекст
        messages = self.get_context_with_memory()
        
        # Запрос к модели
        response = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=messages,
            temperature=0.7
        )
        
        assistant_response = response.choices[0].message.content
        
        # Добавляем ответ ассистента
        self.add_to_short_term("assistant", assistant_response)
        
        # Периодически извлекаем факты
        if len(self.short_term_memory) % 5 == 0:
            self.extract_facts(self.short_term_memory)
        
        return assistant_response
    
    def show_memory(self):
        """Показать состояние памяти"""
        print("🧠 Краткосрочная память:")
        for msg in self.short_term_memory:
            print(f"  [{msg['role']}] {msg['content'][:50]}...")
        
        print("\n💾 Долгосрочная память:")
        for key, value in self.long_term_memory.items():
            print(f"  {key}: {value}")

# Пример использования
agent = AgentWithMemory()

print(agent.chat("Привет! Меня зовут Александр."))
print(agent.chat("Я работаю программистом на Python."))
print(agent.chat("Люблю использовать библиотеку LangChain."))
print(agent.chat("Напомни, как меня зовут и чем я занимаюсь?"))

agent.show_memory()
```

---

## ЧАСТЬ 5: ЛУЧШИЕ ПРАКТИКИ И СОВЕТЫ

### 5.1 Безопасность API ключей

```python
# ✅ ПРАВИЛЬНО: Использование Google Colab Secrets
from google.colab import userdata
API_KEY = userdata.get('OPENAI_API_KEY')

# Как настроить:
# 1. Нажмите на иконку 🔑 (Secrets) в левой панели
# 2. Добавьте новый secret с именем OPENAI_API_KEY
# 3. Вставьте ваш API ключ

# ❌ НЕПРАВИЛЬНО: Хардкодинг ключа в коде
API_KEY = "sk-proj-abc123..."  # Никогда не делайте так!

# ❌ НЕПРАВИЛЬНО: Вывод ключа в лог
print(API_KEY)  # Никогда не печатайте ключи!
```

### 5.2 Оптимизация затрат

```python
# Использование более дешёвых моделей для простых задач
def smart_model_selection(task_complexity):
    if task_complexity == "simple":
        return "gpt-4o-mini"      # ~$0.15/1M токенов
    elif task_complexity == "medium":
        return "gpt-4o"           # ~$2.50/1M токенов
    else:
        return "gpt-4-turbo"      # ~$10/1M токенов

# Ограничение длины ответа
response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=messages,
    max_tokens=500  # Ограничиваем максимальную длину
)

# Подсчёт токенов
def count_tokens(text):
    import tiktoken
    encoder = tiktoken.get_encoding("cl100k_base")
    return len(encoder.encode(text))

print(f"Токенов в ответе: {count_tokens(response.choices[0].message.content)}")
```

### 5.3 Обработка ошибок и retry-логика

```python
import time
from openai import RateLimitError, APIConnectionError

def robust_api_call(func, max_retries=3, backoff_factor=2):
    """Вызов API с повторными попытками при ошибках"""
    
    for attempt in range(max_retries):
        try:
            return func()
        except RateLimitError as e:
            wait_time = backoff_factor ** attempt
            print(f"Rate limit. Ждём {wait_time} секунд...")
            time.sleep(wait_time)
        except APIConnectionError as e:
            wait_time = backoff_factor ** attempt
            print(f"Ошибка соединения. Ждём {wait_time} секунд...")
            time.sleep(wait_time)
        except Exception as e:
            print(f"Неожиданная ошибка: {e}")
            raise
    
    raise Exception("Превышено количество попыток")

# Пример использования
def make_request():
    return client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": "Привет!"}]
    )

response = robust_api_call(make_request)
```

### 5.4 Структурирование проекта в Colab

```
Рекомендуемая структура ноутбука:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📝 Ячейка 1: Заголовок и описание проекта
📝 Ячейка 2: Содержание (навигация)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔧 Ячейка 3: Установка зависимостей (!pip install)
🔧 Ячейка 4: Импорт библиотек
🔧 Ячейка 5: Настройка конфигурации и секретов
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📦 Ячейка 6: Определение классов и функций
📦 Ячейка 7: Дополнительные утилиты
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🧪 Ячейка 8: Тестирование компонентов
🧪 Ячейка 9: Примеры использования
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 Ячейка 10: Визуализация результатов
📊 Ячейка 11: Выводы и следующие шаги
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 5.5 Сохранение и экспорт результатов

```python
# Сохранение всего состояния
import pickle

# Сохранение объектов
with open('agent_state.pkl', 'wb') as f:
    pickle.dump({
        'memory': agent.short_term_memory,
        'facts': agent.long_term_memory
    }, f)

# Загрузка состояния
with open('agent_state.pkl', 'rb') as f:
    state = pickle.load(f)

# Экспорт в различные форматы
import json
import csv

# JSON
with open('results.json', 'w', encoding='utf-8') as f:
    json.dump(results, f, ensure_ascii=False, indent=2)

# CSV
with open('results.csv', 'w', encoding='utf-8-sig', newline='') as f:
    writer = csv.DictWriter(f, fieldnames=results[0].keys())
    writer.writeheader()
    writer.writerows(results)

# Markdown отчёт
def generate_markdown_report(data):
    report = "# Отчёт о работе агента\n\n"
    report += f"## Статистика\n\n"
    report += f"- Всего запросов: {len(data)}\n"
    # ... добавить больше статистики
    return report

with open('report.md', 'w', encoding='utf-8') as f:
    f.write(generate_markdown_report(results))
```

---

## ЧАСТЬ 6: ДОПОЛНИТЕЛЬНЫЕ РЕСУРСЫ

### 6.1 Полезные ссылки

- [Официальная документация Python](https://docs.python.org/3/)
- [Google Colab Documentation](https://research.google.com/colaboratory/faq.html)
- [OpenAI API Reference](https://platform.openai.com/docs/api-reference)
- [LangChain Documentation](https://python.langchain.com/docs/get_started/introduction)

### 6.2 Рекомендуемые практики обучения

1. **Начните с малого**: Освойте базовый синтаксис Python перед работой с AI
2. **Практикуйтесь ежедневно**: Решайте небольшие задачи каждый день
3. **Копируйте и модифицируйте**: Берите рабочие примеры и изменяйте их
4. **Читайте документацию**: Учитесь находить ответы в официальной документации
5. **Экспериментируйте в Colab**: Используйте бесплатные ресурсы для тестов

### 6.3 Чеклист готовности к курсу

- [ ] Установлен Python 3.10+ или есть аккаунт Google Colab
- [ ] Получен API ключ OpenAI (или альтернативного провайдера)
- [ ] Освоены базовые типы данных Python (str, int, list, dict)
- [ ] Понимаете, как создавать и вызывать функции
- [ ] Знаете, как работать с JSON
- [ ] Умеете устанавливать пакеты через pip
- [ ] Можете запустить простой запрос к OpenAI API
- [ ] Понимаете разницу между AI-ассистентом и AI-агентом

---

## ЗАКЛЮЧЕНИЕ

Этот материал покрывает необходимые основы Python и Google Colab для успешной работы с AI-агентами в рамках курса. Рекомендуется:

1. Проработать все примеры кода в Google Colab
2. Модифицировать примеры под свои задачи
3. Создать собственный шаблон для будущих проектов
4. Сохранить этот файл как справочный материал

**Удачи в изучении AI-агентов!** 🚀

---

*Документ подготовлен для курса "Разработка приложений с использованием LLM"*
*Лицензия: The Unlicense (общественное достояние)*
