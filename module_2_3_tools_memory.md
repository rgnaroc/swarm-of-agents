# 📗 МОДУЛЬ 2-3: ИНСТРУМЕНТЫ, ФУНКЦИИ И ПАМЯТЬ АГЕНТОВ
## Расширяем возможности AI-агента

---

## ЧАСТЬ 1: FUNCTION CALLING (ВЫЗОВ ФУНКЦИЙ)

### 1.1 Концепция

**Function Calling** — это способность LLM вызывать внешние функции. Вместо того чтобы просто отвечать текстом, модель может сказать: "Мне нужно выполнить функцию X с аргументами Y".

**Аналогия**: представь, что LLM — это директор компании. Он не делает всё сам, но знает, какие сотрудники (функции) за что отвечают, и даёт им задания.

```
Пользователь: "Сколько будет 245 * 178?"

LLM анализирует: "Это математика — у меня есть инструмент 'калькулятор'"
→ Вызывает calculator(a=245, b=178, operation="multiply")

Результат: 43610

LLM формирует ответ: "245 умножить на 178 равно 43 610"
```

### 1.2 Как описать функцию для LLM

OpenAI использует формат JSON Schema для описания функций:

```python
functions = [
    {
        "name": "get_weather",
        "description": "Получает текущую погоду в указанном городе",
        "parameters": {
            "type": "object",
            "properties": {
                "city": {
                    "type": "string",
                    "description": "Название города, например 'Москва' или 'London'"
                },
                "units": {
                    "type": "string",
                    "enum": ["celsius", "fahrenheit"],
                    "description": "Единицы измерения температуры"
                }
            },
            "required": ["city"]
        }
    }
]
```

**Структура описания:**
- `name` — имя функции (без пробелов)
- `description` — ЧЕМПИОНАТЬ! Описание должно быть таким, чтобы LLM поняла, КОГДА вызывать эту функцию
- `parameters` — JSON Schema параметров
- `required` — обязательные параметры

### 1.3 Полный пример Function Calling

```python
import os
import json
from openai import OpenAI
from dotenv import load_dotenv

load_dotenv()
client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

# ОПРЕДЕЛЯЕМ ФУНКЦИИ
functions = [
    {
        "name": "calculate",
        "description": "Выполняет математические вычисления",
        "parameters": {
            "type": "object",
            "properties": {
                "expression": {
                    "type": "string",
                    "description": "Математическое выражение, например '2 + 2' или '10 * 5'"
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
            "properties": {}
        }
    }
]

# РЕАЛИЗАЦИЯ ФУНКЦИЙ
def calculate(expression: str) -> str:
    """Безопасный калькулятор"""
    try:
        # Разрешённые символы
        allowed = set('0123456789+-*/.() ')
        if not all(c in allowed for c in expression):
            return "Ошибка: недопустимые символы"
        result = eval(expression)
        return str(result)
    except Exception as e:
        return f"Ошибка вычисления: {e}"

def get_current_time() -> str:
    """Текущее время"""
    from datetime import datetime
    return datetime.now().strftime("%Y-%m-%d %H:%M:%S")

# СЛОВАРЬ ДОСТУПНЫХ ФУНКЦИЙ
available_functions = {
    "calculate": calculate,
    "get_current_time": get_current_time
}

# ДИАЛОГ С АГЕНТОМ
def run_conversation(user_message: str):
    messages = [
        {"role": "system", "content": "Ты — полезный ассистент. Используй инструменты, когда нужно что-то вычислить или узнать время."},
        {"role": "user", "content": user_message}
    ]
    
    # Первый запрос — модель решает, нужны ли функции
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=messages,
        functions=functions,
        function_call="auto",  # auto / none / {"name": "..."}
        temperature=0.1
    )
    
    message = response.choices[0].message
    
    # Если модель хочет вызвать функцию
    if message.function_call:
        function_name = message.function_call.name
        function_args = json.loads(message.function_call.arguments)
        
        print(f"🤖 Агент вызывает функцию: {function_name}({function_args})")
        
        # Вызываем функцию
        function_to_call = available_functions[function_name]
        function_response = function_to_call(**function_args)
        
        # Добавляем в историю
        messages.append(message)  # запрос функции
        messages.append({
            "role": "function",
            "name": function_name,
            "content": function_response
        })
        
        # Повторный запрос для финального ответа
        second_response = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=messages,
            temperature=0.7
        )
        
        return second_response.choices[0].message.content
    
    return message.content

# Тестируем
print(run_conversation("Сколько будет 125 умножить на 37?"))
print(run_conversation("Который час?"))
print(run_conversation("Расскажи анекдот"))  # не требует функций
```

### 1.4 Параметр function_call

```python
# Автоматический выбор (модель решает сама)
function_call="auto"

# Запретить вызовы функций
function_call="none"

# Принудительно вызвать конкретную функцию
function_call={"name": "calculate"}
```

---

## ЧАСТЬ 2: СОЗДАНИЕ ИНСТРУМЕНТОВ (TOOLS)

### 2.1 Инструмент поиска в интернете

```python
import requests
from bs4 import BeautifulSoup

def web_search(query: str, num_results: int = 3) -> str:
    """Поиск через DuckDuckGo (без API-ключа!)"""
    try:
        url = "https://html.duckduckgo.com/html/"
        params = {"q": query}
        
        headers = {
            "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
        }
        
        response = requests.post(url, data=params, headers=headers, timeout=10)
        soup = BeautifulSoup(response.text, "html.parser")
        
        results = []
        for result in soup.select(".result")[:num_results]:
            title = result.select_one(".result__title")
            snippet = result.select_one(".result__snippet")
            if title and snippet:
                results.append(f"{title.get_text(strip=True)}: {snippet.get_text(strip=True)}")
        
        return "\n".join(results) if results else "Ничего не найдено"
    
    except Exception as e:
        return f"Ошибка поиска: {e}"

# Тест
print(web_search("Python AI agents tutorial 2024"))
```

### 2.2 Работа с файлами

```python
def read_file(filepath: str) -> str:
    """Чтение текстового файла"""
    try:
        with open(filepath, "r", encoding="utf-8") as f:
            content = f.read()
        return content[:3000]  # ограничиваем длину
    except Exception as e:
        return f"Ошибка чтения: {e}"

def write_file(filepath: str, content: str) -> str:
    """Запись в файл"""
    try:
        with open(filepath, "w", encoding="utf-8") as f:
            f.write(content)
        return f"Файл {filepath} успешно записан"
    except Exception as e:
        return f"Ошибка записи: {e}"

def list_directory(path: str = ".") -> str:
    """Список файлов в папке"""
    try:
        import os
        files = os.listdir(path)
        return "\n".join(files)
    except Exception as e:
        return f"Ошибка: {e}"
```

### 2.3 Полноценный агент с 5 инструментами

```python
import os
import json
import requests
from datetime import datetime
from openai import OpenAI
from dotenv import load_dotenv

load_dotenv()
client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

class ToolAgent:
    def __init__(self):
        self.tools = [
            {
                "type": "function",
                "function": {
                    "name": "search_web",
                    "description": "Ищет информацию в интернете",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "query": {"type": "string", "description": "Поисковый запрос"}
                        },
                        "required": ["query"]
                    }
                }
            },
            {
                "type": "function",
                "function": {
                    "name": "calculator",
                    "description": "Вычисляет математические выражения",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "expression": {"type": "string", "description": "Например: 2+2 или 100/5"}
                        },
                        "required": ["expression"]
                    }
                }
            },
            {
                "type": "function",
                "function": {
                    "name": "get_time",
                    "description": "Узнаёт текущее время",
                    "parameters": {"type": "object", "properties": {}}
                }
            },
            {
                "type": "function",
                "function": {
                    "name": "read_txt_file",
                    "description": "Читает текстовый файл",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "path": {"type": "string", "description": "Путь к файлу"}
                        },
                        "required": ["path"]
                    }
                }
            },
            {
                "type": "function",
                "function": {
                    "name": "write_txt_file",
                    "description": "Создаёт или перезаписывает текстовый файл",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "path": {"type": "string", "description": "Куда сохранить"},
                            "content": {"type": "string", "description": "Что записать"}
                        },
                        "required": ["path", "content"]
                    }
                }
            }
        ]
        
        self.functions = {
            "search_web": self.search_web,
            "calculator": self.calculator,
            "get_time": self.get_time,
            "read_txt_file": self.read_txt_file,
            "write_txt_file": self.write_txt_file
        }
    
    def search_web(self, query: str) -> str:
        try:
            url = "https://html.duckduckgo.com/html/"
            headers = {"User-Agent": "Mozilla/5.0"}
            response = requests.post(url, data={"q": query}, headers=headers, timeout=10)
            from bs4 import BeautifulSoup
            soup = BeautifulSoup(response.text, "html.parser")
            results = []
            for r in soup.select(".result")[:3]:
                title = r.select_one(".result__title")
                if title:
                    results.append(title.get_text(strip=True))
            return "; ".join(results) if results else "Ничего не найдено"
        except Exception as e:
            return f"Ошибка: {e}"
    
    def calculator(self, expression: str) -> str:
        allowed = set('0123456789+-*/.() ')
        if not all(c in allowed for c in expression):
            return "Ошибка: недопустимые символы"
        try:
            return str(eval(expression))
        except:
            return "Ошибка вычисления"
    
    def get_time(self) -> str:
        return datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    
    def read_txt_file(self, path: str) -> str:
        try:
            with open(path, "r", encoding="utf-8") as f:
                return f.read()[:2000]
        except Exception as e:
            return f"Ошибка: {e}"
    
    def write_txt_file(self, path: str, content: str) -> str:
        try:
            with open(path, "w", encoding="utf-8") as f:
                f.write(content)
            return f"Файл {path} сохранён"
        except Exception as e:
            return f"Ошибка: {e}"
    
    def run(self, user_input: str) -> str:
        messages = [
            {"role": "system", "content": "Ты — умный ассистент с доступом к инструментам. Используй их для поиска информации, вычислений, работы с файлами и проверки времени."},
            {"role": "user", "content": user_input}
        ]
        
        # Цикл для множественных вызовов
        for _ in range(5):  # максимум 5 итераций
            response = client.chat.completions.create(
                model="gpt-4o-mini",
                messages=messages,
                tools=self.tools,
                tool_choice="auto",
                temperature=0.2
            )
            
            message = response.choices[0].message
            
            # Если нет вызовов — возвращаем ответ
            if not message.tool_calls:
                return message.content
            
            # Иначе обрабатываем вызовы
            messages.append(message)
            
            for tool_call in message.tool_calls:
                function_name = tool_call.function.name
                function_args = json.loads(tool_call.function.arguments)
                
                print(f"🔧 Вызов: {function_name}({function_args})")
                
                function = self.functions[function_name]
                result = function(**function_args)
                
                messages.append({
                    "role": "tool",
                    "tool_call_id": tool_call.id,
                    "name": function_name,
                    "content": result
                })
        
        return "Слишком много итераций"

# Использование
agent = ToolAgent()
print(agent.run("Найди последние новости про Python и запиши их в файл news.txt"))
print(agent.run("Сколько будет 135 * 47 + 892?"))
```

---

## ЧАСТЬ 3: ПАМЯТЬ АГЕНТА

### 3.1 Зачем агенту память?

**Без памяти:**
- Агент не помнит предыдущие диалоги
- Не может учиться на прошлом опыте
- Тратит токены на повторный контекст

**С памятью:**
- Помнит предпочтения пользователя
- Использует прошлые данные
- Учится на ошибках

### 3.2 Виды памяти

```
┌─────────────────────────────────────────┐
│         ВИДЫ ПАМЯТИ АГЕНТА              │
├─────────────────────────────────────────┤
│                                         │
│  📝 Краткосрочная (Short-term)          │
│     └─ Контекстное окно (текущий чат)  │
│     └─ Sliding window                   │
│                                         │
│  💾 Долгосрочная (Long-term)            │
│     └─ Векторная база данных            │
│     └─ SQL/NoSQL базы                   │
│                                         │
│  🧠 Процедурная (Procedural)            │
│     └─ Навыки и шаблоны поведения       │
│     └─ Условные рефлексы                │
│                                         │
└─────────────────────────────────────────┘
```

### 3.3 Векторные представления (Embeddings)

**Концепция**: превращаем текст в массив чисел (вектор), где похожие тексты имеют близкие векторы.

**Аналогия**: каждый текст — точка в пространстве. Близкие по смыслу тексты — близкие точки.

```python
from openai import OpenAI
import numpy as np

client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

def get_embedding(text: str) -> list:
    """Получаем векторное представление текста"""
    response = client.embeddings.create(
        model="text-embedding-3-small",
        input=text
    )
    return response.data[0].embedding

# Пример
words = ["король", "королева", "мужчина", "женщина", "яблоко"]
embeddings = [get_embedding(w) for w in words]

# Функция косинусной близости
def cosine_similarity(a, b):
    return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))

# Сравниваем
print(f"король ~ королева: {cosine_similarity(embeddings[0], embeddings[1]):.3f}")
print(f"король ~ яблоко: {cosine_similarity(embeddings[0], embeddings[4]):.3f}")
# Вывод: король ближе к королеве, чем к яблоку!
```

### 3.4 Векторная база данных ChromaDB

```bash
pip install chromadb
```

```python
import chromadb
from chromadb.utils.embedding_functions import OpenAIEmbeddingFunction

# Инициализация
client = chromadb.Client()

# Создаём коллекцию с OpenAI embeddings
embedding_function = OpenAIEmbeddingFunction(
    api_key=os.getenv("OPENAI_API_KEY"),
    model_name="text-embedding-3-small"
)

collection = client.create_collection(
    name="my_knowledge",
    embedding_function=embedding_function
)

# Добавляем документы
documents = [
    "Python — язык программирования общего назначения",
    "JavaScript используется для веб-разработки",
    "Docker контейнеризирует приложения",
    "Kubernetes оркестрирует контейнеры",
    "React — библиотека для UI",
]

collection.add(
    documents=documents,
    ids=["doc1", "doc2", "doc3", "doc4", "doc5"],
    metadatas=[
        {"category": "language"},
        {"category": "language"},
        {"category": "devops"},
        {"category": "devops"},
        {"category": "frontend"}
    ]
)

# Поиск
results = collection.query(
    query_texts=["Чем Docker отличается от Kubernetes?"],
    n_results=2
)

print(results["documents"])  # найдёт doc3 и doc4!
```

### 3.5 RAG (Retrieval-Augmented Generation)

**RAG** — это когда агент ищет релевантную информацию в базе знаний и использует её для ответа.

```python
class RAGAgent:
    def __init__(self):
        self.client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))
        
        # Инициализируем ChromaDB
        chroma_client = chromadb.Client()
        
        # Получаем или создаём коллекцию
        try:
            self.collection = chroma_client.get_collection("knowledge_base")
        except:
            self.collection = chroma_client.create_collection("knowledge_base")
    
    def add_documents(self, texts: list, sources: list = None):
        """Добавляем документы в базу знаний"""
        ids = [f"doc_{i}" for i in range(len(texts))]
        
        # Генерируем embeddings через OpenAI
        embeddings = []
        for text in texts:
            response = self.client.embeddings.create(
                model="text-embedding-3-small",
                input=text
            )
            embeddings.append(response.data[0].embedding)
        
        self.collection.add(
            documents=texts,
            ids=ids,
            embeddings=embeddings,
            metadatas=[{"source": s} for s in (sources or ["unknown"] * len(texts))]
        )
    
    def ask(self, question: str, top_k: int = 3) -> str:
        """Отвечаем на вопрос с использованием RAG"""
        
        # 1. Ищем релевантные документы
        results = self.collection.query(
            query_embeddings=[self._get_embedding(question)],
            n_results=top_k
        )
        
        context = "\n\n".join(results["documents"][0])
        sources = results["metadatas"][0]
        
        # 2. Формируем промпт с контекстом
        prompt = f"""Ответь на вопрос, используя ТОЛЬКО предоставленный контекст.
Если ответа нет в контексте — скажи "Я не знаю".

Контекст:
{context}

Вопрос: {question}

Дай краткий и точный ответ."""
        
        # 3. Получаем ответ
        response = self.client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[
                {"role": "system", "content": "Ты — полезный ассистент. Отвечай только на основе предоставленного контекста."},
                {"role": "user", "content": prompt}
            ],
            temperature=0.1
        )
        
        answer = response.choices[0].message.content
        
        # Добавляем источники
        source_info = "\n\nИсточники: " + ", ".join([s["source"] for s in sources])
        return answer + source_info
    
    def _get_embedding(self, text: str) -> list:
        response = self.client.embeddings.create(
            model="text-embedding-3-small",
            input=text
        )
        return response.data[0].embedding

# Использование
rag = RAGAgent()

# Загружаем знания
knowledge = [
    "Наша компания работает с 2015 года. Штаб-квартира в Москве.",
    "Продукт Alpha — облачное решение для аналитики данных.",
    "Цена продукта Alpha: от 5000 руб/месяц за базовый тариф.",
    "Поддержка работает 24/7 через чат и email: support@company.ru",
]

rag.add_documents(knowledge, sources=["about", "products", "pricing", "support"])

# Спрашиваем
print(rag.ask("Сколько стоит продукт Alpha?"))
print(rag.ask("Как связаться с поддержкой?"))
print(rag.ask("Какая погода в Москве?"))  # Должен сказать "не знаю"
```

---

## ЧАСТЬ 4: УПРАВЛЕНИЕ ИСТОРИЕЙ ДИАЛОГА

### 4.1 Summarization (сжатие истории)

```python
class ConversationSummarizer:
    def __init__(self):
        self.client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))
        self.messages = []
        self.summary = ""
    
    def add_message(self, role: str, content: str):
        self.messages.append({"role": role, "content": content})
        
        # Если история длинная — сжимаем
        if len(str(self.messages)) > 3000:
            self._summarize()
    
    def _summarize(self):
        prompt = f"""Сжати следующий диалог в 2-3 предложения, сохраняя ключевые факты:
        
{self.messages}"""
        
        response = self.client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{"role": "user", "content": prompt}]
        )
        
        self.summary = response.choices[0].message.content
        self.messages = []  # очищаем
        print(f"📝 Создано резюме: {self.summary[:100]}...")
    
    def get_context(self) -> list:
        """Возвращает контекст для LLM"""
        context = []
        if self.summary:
            context.append({
                "role": "system", 
                "content": f"Резюме предыдущего диалога: {self.summary}"
            })
        context.extend(self.messages)
        return context
```

### 4.2 SQLite хранилище диалогов

```python
import sqlite3
from datetime import datetime

class PersistentMemory:
    def __init__(self, db_path: str = "conversations.db"):
        self.conn = sqlite3.connect(db_path)
        self._init_db()
    
    def _init_db(self):
        self.conn.execute("""
            CREATE TABLE IF NOT EXISTS messages (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                session_id TEXT,
                role TEXT,
                content TEXT,
                timestamp TEXT,
                metadata TEXT
            )
        """)
        self.conn.commit()
    
    def save_message(self, session_id: str, role: str, content: str):
        self.conn.execute(
            "INSERT INTO messages (session_id, role, content, timestamp) VALUES (?, ?, ?, ?)",
            (session_id, role, content, datetime.now().isoformat())
        )
        self.conn.commit()
    
    def get_history(self, session_id: str, limit: int = 20) -> list:
        cursor = self.conn.execute(
            "SELECT role, content FROM messages WHERE session_id = ? ORDER BY id DESC LIMIT ?",
            (session_id, limit)
        )
        return [{"role": r, "content": c} for r, c in cursor.fetchall()[::-1]]

# Использование
memory = PersistentMemory()
memory.save_message("user_123", "user", "Привет!")
memory.save_message("user_123", "assistant", "Здравствуй! Чем помочь?")
print(memory.get_history("user_123"))
```

---

## 📋 ПРАКТИЧЕСКИЕ ЗАДАНИЯ МОДУЛЯ 2-3

### Задание 1: Агент-калькулятор (⭐ Базовое)
Создай агента с инструментами:
- Калькулятор (+, -, *, /, **, sqrt)
- Перевод валют (фиксированный курс)
- Конвертер единиц (км→мили, кг→фунты)

### Задание 2: RAG-система для PDF (⭐⭐ Среднее)
Создай систему, которая:
- Загружает PDF-документы
- Разбивает на чанки (куски)
- Сохраняет в ChromaDB
- Отвечает на вопросы по содержимому

Подсказка: используй PyPDF2 или pdfplumber для чтения PDF.

### Задание 3: Персональный агент с долгосрочной памятью (⭐⭐⭐ Продвинутое)
Создай агента, который:
- Запоминает факты о пользователе (имя, предпочтения)
- Хранит информацию в SQLite
- Использует прошлые данные в новых диалогах
- Создаёт резюме долгих разговоров

---

## 🔍 ПРОВЕРОЧНЫЕ ВОПРОСЫ

1. Что такое Function Calling и зачем он нужен?
2. Как LLM понимает, какую функцию вызвать?
3. Что такое embedding и как он связан с семантическим поиском?
4. В чём суть RAG-подхода?
5. Какие виды памяти у агента и зачем каждая?
6. Почему важно ограничивать вызовы функций (max iterations)?

---

## 📚 РЕСУРСЫ

### Библиотеки для установки:
```bash
pip install openai chromadb requests beautifulsoup4 pypdf2 sqlite3
```

### Документация:
- [OpenAI Function Calling](https://platform.openai.com/docs/guides/function-calling)
- [ChromaDB Docs](https://docs.trychroma.com/)
- [Embeddings Guide](https://platform.openai.com/docs/guides/embeddings)

---

**Следующий модуль**: Фреймворки LangChain, LangGraph, CrewAI — создаём сложные workflow! 🚀
