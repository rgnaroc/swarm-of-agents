# 📙 МОДУЛЬ 4-5: ФРЕЙМВОРКИ И МУЛЬТИАГЕНТНЫЕ СИСТЕМЫ
## Профессиональные инструменты для создания агентов

---

## ЧАСТЬ 1: LANGCHAIN — ОСНОВЫ

### 1.1 Что такое LangChain?

LangChain — это фреймворк для создания приложений с LLM. Он предоставляет готовые компоненты: цепочки, агенты, память, инструменты.

**Аналогия**: если раньше мы писали агента "с нуля" (как строили дом из палок), LangChain — это конструктор LEGO с готовыми деталями.

```bash
pip install langchain langchain-openai langchain-community
```

### 1.2 Базовые компоненты

```python
import os
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_core.messages import HumanMessage, SystemMessage

load_dotenv()

# Создаём модель
llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,
    api_key=os.getenv("OPENAI_API_KEY")
)

# Простой запрос
messages = [
    SystemMessage(content="Ты — дружелюбный помощник."),
    HumanMessage(content="Объясни, что такое LangChain")
]

response = llm.invoke(messages)
print(response.content)
```

### 1.3 Prompt Templates (шаблоны промптов)

```python
from langchain_core.prompts import ChatPromptTemplate, PromptTemplate

# Простой шаблон
template = PromptTemplate.from_template(
    "Расскажи {count} фактов о {topic} на {language} языке"
)

prompt = template.format(count=3, topic="Python", language="русском")
print(prompt)
# Вывод: Расскажи 3 фактов о Python на русском языке

# Chat-шаблон с системным сообщением
chat_template = ChatPromptTemplate.from_messages([
    ("system", "Ты — {role}. Общайся в стиле {style}."),
    ("human", "{question}")
])

prompt = chat_template.format_messages(
    role="эксперт по программированию",
    style="простой и понятный",
    question="Что такое рекурсия?"
)

response = llm.invoke(prompt)
print(response.content)
```

### 1.4 Chains (цепочки)

**Chain** — это последовательность операций, где выход одного шага — вход следующего.

```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate

# Создаём цепочку: шаблон → LLM → парсер
prompt = ChatPromptTemplate.from_messages([
    ("system", "Ты — переводчик. Переведи текст на {language}."),
    ("human", "{text}")
])

# Парсер извлекает текст из ответа
parser = StrOutputParser()

# Цепочка (LCEL синтаксис)
chain = prompt | llm | parser

# Запуск
result = chain.invoke({
    "language": "английский",
    "text": "Привет, мир! Как дела?"
})
print(result)
```

**LCEL (LangChain Expression Language)** — оператор `|` объединяет компоненты, как пайплайн в Unix.

### 1.5 Sequential Chain (последовательная цепочка)

```python
from langchain_core.prompts import ChatPromptTemplate

# Шаг 1: Генерация идей
ideas_prompt = ChatPromptTemplate.from_template(
    "Придумай 3 идеи для стартапа в сфере {industry}"
)
ideas_chain = ideas_prompt | llm | StrOutputParser()

# Шаг 2: Оценка идей
eval_prompt = ChatPromptTemplate.from_template(
    "Оцени эти идеи стартапа с точки зрения реализуемости (1-10):\n{ideas}"
)
eval_chain = eval_prompt | llm | StrOutputParser()

# Общая цепочка
full_chain = ideas_chain | (lambda x: {"ideas": x}) | eval_chain

# Запуск
result = full_chain.invoke({"industry": "образования"})
print(result)
```

---

## ЧАСТЬ 2: LANGCHAIN AGENTS

### 2.1 Создание агента с инструментами

```python
from langchain_openai import ChatOpenAI
from langchain.agents import create_tool_calling_agent, AgentExecutor
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.tools import tool

# Определяем инструменты декоратором @tool
@tool
def multiply(a: int, b: int) -> int:
    """Умножает два числа."""
    return a * b

@tool
def add(a: int, b: int) -> int:
    """Складывает два числа."""
    return a + b

@tool
def get_weather(city: str) -> str:
    """Возвращает погоду (заглушка)."""
    weather_data = {
        "москва": "Солнечно, +22°C",
        "лондон": "Дождливо, +15°C",
        "токио": "Облачно, +26°C"
    }
    return weather_data.get(city.lower(), "Нет данных")

# Список инструментов
tools = [multiply, add, get_weather]

# Промпт для агента
prompt = ChatPromptTemplate.from_messages([
    ("system", "Ты — полезный ассистент с доступом к инструментам."),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}"),  # важно для агента!
])

# Создаём агента
llm = ChatOpenAI(model="gpt-4o-mini")
agent = create_tool_calling_agent(llm, tools, prompt)

# Executor запускает цикл мышление → действие → наблюдение
executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,  # показываем шаги
    max_iterations=10
)

# Запуск
result = executor.invoke({"input": "Сколько будет 25 умножить на 4 плюс 100?"})
print(result["output"])
```

### 2.2 Встроенные инструменты LangChain

```python
from langchain_community.tools import DuckDuckGoSearchRun, WikipediaQueryRun
from langchain_community.utilities import WikipediaAPIWrapper

# Поиск в интернете
search = DuckDuckGoSearchRun()
result = search.run("Python AI agents LangChain 2024")
print(result)

# Поиск в Википедии
wiki = WikipediaQueryRun(api_wrapper=WikipediaAPIWrapper(lang="ru"))
result = wiki.run("Искусственный интеллект")
print(result[:500])
```

### 2.3 ReAct агент

```python
from langchain.agents import create_react_agent, AgentExecutor
from langchain_core.prompts import PromptTemplate

# ReAct промпт (Reasoning + Acting)
react_template = """Ответь на вопрос, используя инструменты.

У тебя есть доступ к:
{tools}

Используй формат:
Thought: мои рассуждения
Action: название_инструмента
Action Input: входные данные
Observation: результат
... (повторяй пока не получишь ответ)
Final Answer: ответ пользователю

Начало!

Question: {input}
{agent_scratchpad}"""

prompt = PromptTemplate.from_template(react_template).partial(
    tools=", ".join([t.name for t in tools])
)

agent = create_react_agent(llm, tools, prompt)
executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

executor.invoke({"input": "Найди информацию о LangChain и объясни, что это"})
```

---

## ЧАСТЬ 3: LANGGRAPH — ГРАФОВЫЕ WORKFLOW

### 3.1 Концепция LangGraph

LangGraph позволяет создавать сложные workflow в виде графа: узлы (функции) и рёбра (переходы между ними).

**Зачем?** Для многошаговых процессов с условиями и циклами.

```bash
pip install langgraph
```

### 3.2 Простой граф

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict, Annotated
import operator

# Определяем состояние (данные, которые передаются между узлами)
class AgentState(TypedDict):
    input: str
    output: str
    count: int

# Узлы — обычные функции
def node_start(state: AgentState):
    print(f"🚀 Старт с входом: {state['input']}")
    return {"count": state.get("count", 0) + 1}

def node_process(state: AgentState):
    print(f"⚙️ Обработка (шаг {state['count']})")
    return {
        "output": f"Обработано: {state['input'].upper()}",
        "count": state["count"] + 1
    }

def node_end(state: AgentState):
    print(f"✅ Финал: {state['output']}")
    return {}

# Условный переход
def should_continue(state: AgentState):
    if state["count"] < 3:
        return "process"  # возвращаемся к обработке
    return "end"  # идём к финалу

# Создаём граф
builder = StateGraph(AgentState)

# Добавляем узлы
builder.add_node("start", node_start)
builder.add_node("process", node_process)
builder.add_node("end", node_end)

# Добавляем рёбра
builder.set_entry_point("start")
builder.add_edge("start", "process")

# Условное ребро из process
builder.add_conditional_edges(
    "process",
    should_continue,
    {"process": "process", "end": "end"}
)

builder.add_edge("end", END)

# Компилируем и запускаем
graph = builder.compile()
result = graph.invoke({"input": "hello world"})
print(result)
```

### 3.3 Граф с LLM

```python
from langgraph.graph import StateGraph
from langchain_openai import ChatOpenAI
from typing import TypedDict

class State(TypedDict):
    question: str
    context: str
    answer: str
    needs_search: bool

llm = ChatOpenAI(model="gpt-4o-mini")

# Узел: анализ вопроса
def analyze(state: State):
    prompt = f"Нужен ли поиск в интернете для ответа? Ответь только 'да' или 'нет'.\nВопрос: {state['question']}"
    response = llm.invoke(prompt)
    needs_search = "да" in response.content.lower()
    return {"needs_search": needs_search}

# Узел: поиск (заглушка)
def search(state: State):
    if state["needs_search"]:
        # Здесь был бы реальный поиск
        return {"context": "Найденная информация по запросу"}
    return {"context": ""}

# Узел: генерация ответа
def generate(state: State):
    context = state.get("context", "")
    prompt = f"Контекст: {context}\nВопрос: {state['question']}\nОтвет:"
    response = llm.invoke(prompt)
    return {"answer": response.content}

# Строим граф
builder = StateGraph(State)
builder.add_node("analyze", analyze)
builder.add_node("search", search)
builder.add_node("generate", generate)

builder.set_entry_point("analyze")
builder.add_edge("analyze", "search")
builder.add_edge("search", "generate")
builder.add_edge("generate", END)

graph = builder.compile()
result = graph.invoke({"question": "Какая погода в Москве?"})
print(result["answer"])
```

---

## ЧАСТЬ 4: CREWAI — КОМАНДЫ АГЕНТОВ

### 4.1 Философия CrewAI

CrewAI — фреймворк для создания команд агентов (crew = команда). Каждый агент имеет роль, цель и набор инструментов.

```bash
pip install crewai crewai-tools
```

### 4.2 Создание команды аналитиков

```python
import os
from crewai import Agent, Task, Crew
from crewai_tools import ScrapeWebsiteTool, SerperDevTool

# Инструменты
search_tool = SerperDevTool(api_key=os.getenv("SERPER_API_KEY"))
scrape_tool = ScrapeWebsiteTool()

# Агент 1: Исследователь
researcher = Agent(
    role="Исследователь рынка",
    goal="Найти актуальную информацию о заданной теме",
    backstory="Опытный аналитик с 10-летним стажем. Умеет находить скрытые инсайты.",
    tools=[search_tool, scrape_tool],
    verbose=True,
    allow_delegation=False,
    llm="gpt-4o-mini"
)

# Агент 2: Аналитик
analyst = Agent(
    role="Бизнес-аналитик",
    goal="Проанализировать данные и выявить тренды",
    backstory="Эксперт по бизнес-анализу. Строит прогнозы на основе данных.",
    tools=[search_tool],
    verbose=True,
    allow_delegation=False,
    llm="gpt-4o-mini"
)

# Агент 3: Писатель
writer = Agent(
    role="Контент-писатель",
    goal="Написать понятный и структурированный отчёт",
    backstory="Профессиональный копирайтер. Превращает сложные данные в читаемый текст.",
    tools=[],
    verbose=True,
    allow_delegation=False,
    llm="gpt-4o-mini"
)

# Задачи
task_research = Task(
    description="Исследуй рынок AI-агентов в 2024 году. Найди: ключевые игроки, объём рынка, тренды.",
    expected_output="Структурированный список фактов о рынке AI-агентов",
    agent=researcher
)

task_analyze = Task(
    description="Проанализируй данные исследователя. Выдели 3 главных тренда и дай прогноз на 2025 год.",
    expected_output="Аналитический отчёт с трендами и прогнозом",
    agent=analyst,
    context=[task_research]  # зависит от первой задачи
)

task_write = Task(
    description="Напиши финальный отчёт на основе анализа. Максимум 2 страницы, простой язык.",
    expected_output="Готовый отчёт в markdown формате",
    agent=writer,
    context=[task_analyze]
)

# Создаём команду
crew = Crew(
    agents=[researcher, analyst, writer],
    tasks=[task_research, task_analyze, task_write],
    verbose=True,
    process="sequential"  # последовательное выполнение
)

# Запуск
result = crew.kickoff()
print(result)
```

### 4.3 Процессы в CrewAI

| Процесс | Описание | Когда использовать |
|---------|----------|-------------------|
| `sequential` | Задачи выполняются по порядку | Линейные workflow |
| `hierarchical` | Есть менеджер, который делегирует | Сложные проекты |
| `parallel` | Задачи выполняются параллельно | Независимые задачи |

---

## ЧАСТЬ 5: AUTOGEN — МУЛЬТИАГЕНТНЫЕ ДИАЛОГИ

### 5.1 Концепция AutoGen

AutoGen (Microsoft) — фреймворк для создания агентов, которые общаются друг с другом.

```bash
pip install pyautogen
```

### 5.2 Два агента в диалоге

```python
import autogen

# Конфигурация LLM
config_list = [{
    "model": "gpt-4o-mini",
    "api_key": os.getenv("OPENAI_API_KEY")
}]

# Агент-помощник (ИИ)
assistant = autogen.AssistantAgent(
    name="assistant",
    llm_config={"config_list": config_list},
    system_message="Ты — опытный программист. Помогай писать и отлаживать код."
)

# Агент-пользователь (прокси)
user_proxy = autogen.UserProxyAgent(
    name="user_proxy",
    human_input_mode="NEVER",  # не ждём человека
    max_consecutive_auto_reply=5,
    code_execution_config={
        "work_dir": "coding",
        "use_docker": False  # для простоты
    }
)

# Запускаем диалог
user_proxy.initiate_chat(
    assistant,
    message="Напиши функцию на Python для сортировки списка словарей по ключу"
)
```

### 5.3 GroupChat — групповой чат

```python
import autogen

config_list = [{"model": "gpt-4o-mini", "api_key": os.getenv("OPENAI_API_KEY")}]

# Создаём агентов
pm = autogen.AssistantAgent(
    name="ProductManager",
    llm_config={"config_list": config_list},
    system_message="Ты — продакт-менеджер. Определяй требования и приоритеты."
)

dev = autogen.AssistantAgent(
    name="Developer",
    llm_config={"config_list": config_list},
    system_message="Ты — разработчик. Пишешь чистый, эффективный код."
)

tester = autogen.AssistantAgent(
    name="Tester",
    llm_config={"config_list": config_list},
    system_message="Ты — тестировщик. Ищешь баги и пишешь тесты."
)

user_proxy = autogen.UserProxyAgent(
    name="User",
    human_input_mode="NEVER",
    max_consecutive_auto_reply=10,
    code_execution_config={"work_dir": "project", "use_docker": False}
)

# Групповой чат
groupchat = autogen.GroupChat(
    agents=[user_proxy, pm, dev, tester],
    messages=[],
    max_round=12
)

manager = autogen.GroupChatManager(
    groupchat=groupchat,
    llm_config={"config_list": config_list}
)

# Запуск
user_proxy.initiate_chat(
    manager,
    message="Создайте простое TODO-приложение на Python с CLI-интерфейсом"
)
```

---

## ЧАСТЬ 6: МУЛЬТИАГЕНТНЫЕ ПАТТЕРНЫ

### 6.1 Supervisor Pattern

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict, List

class MultiAgentState(TypedDict):
    task: str
    result_planning: str
    result_coding: str
    result_testing: str
    final_result: str

# Рабочие агенты
def planner(state: MultiAgentState):
    prompt = f"Создай план выполнения: {state['task']}"
    response = llm.invoke(prompt)
    return {"result_planning": response.content}

def coder(state: MultiAgentState):
    prompt = f"Напиши код по плану:\n{state['result_planning']}"
    response = llm.invoke(prompt)
    return {"result_coding": response.content}

def tester(state: MultiAgentState):
    prompt = f"Протестируй этот код:\n{state['result_coding']}"
    response = llm.invoke(prompt)
    return {"result_testing": response.content}

# Супервизор (координатор)
def supervisor(state: MultiAgentState):
    prompt = f"""Объедини результаты в финальный ответ:
    
План: {state['result_planning']}
Код: {state['result_coding']}
Тесты: {state['result_testing']}"""
    
    response = llm.invoke(prompt)
    return {"final_result": response.content}

# Строим граф
builder = StateGraph(MultiAgentState)
builder.add_node("planner", planner)
builder.add_node("coder", coder)
builder.add_node("tester", tester)
builder.add_node("supervisor", supervisor)

# Последовательное выполнение
builder.set_entry_point("planner")
builder.add_edge("planner", "coder")
builder.add_edge("coder", "tester")
builder.add_edge("tester", "supervisor")
builder.add_edge("supervisor", END)

graph = builder.compile()
result = graph.invoke({"task": "Создать функцию для парсинга CSV"})
print(result["final_result"])
```

### 6.2 Human-in-the-Loop

```python
from langgraph.graph import StateGraph
from langgraph.checkpoint.memory import MemorySaver

class State(TypedDict):
    code: str
    approved: bool

def generate_code(state: State):
    return {"code": "def hello():\n    print('Hello!')"}

def human_review(state: State):
    print(f"Сгенерирован код:\n{state['code']}")
    # Здесь человек проверяет и подтверждает
    return {"approved": True}  # или False для повторной генерации

def deploy(state: State):
    print("Код развёрнут!")
    return {}

# Строим граф с checkpoint
builder = StateGraph(State)
builder.add_node("generate", generate_code)
builder.add_node("review", human_review)
builder.add_node("deploy", deploy)

builder.set_entry_point("generate")
builder.add_edge("generate", "review")

# Условный переход
builder.add_conditional_edges(
    "review",
    lambda s: "deploy" if s["approved"] else "generate"
)
builder.add_edge("deploy", END)

# Добавляем память для возможности прерывания
memory = MemorySaver()
graph = builder.compile(checkpointer=memory, interrupt_before=["review"])

# Запуск
result = graph.invoke({"code": "", "approved": False}, config={"thread_id": "1"})
```

---

## 📋 ПРАКТИЧЕСКИЕ ЗАДАНИЯ МОДУЛЯ 4-5

### Задание 1: LangChain цепочка (⭐ Базовое)
Создай цепочку, которая:
- Принимает тему статьи
- Генерирует план (3 раздела)
- Пишет текст для каждого раздела
- Объединяет в финальную статью

### Задание 2: CrewAI команда (⭐⭐ Среднее)
Создай команду для разработки:
- Аналитик: собирает требования
- Архитектор: проектирует структуру
- Разработчик: пишет код
Результат: работающий Python-скрипт.

### Задание 3: LangGraph workflow с циклом (⭐⭐⭐ Продвинутое)
Создай граф, который:
- Генерирует код
- Тестирует его (запускает)
- Если есть ошибки — исправляет (цикл до 3 раз)
- Возвращает рабочий код

---

## 🔍 ПРОВЕРОЧНЫЕ ВОПРОСЫ

1. В чём разница между LangChain и LangGraph?
2. Как CrewAI управляет командами агентов?
3. Что такое ReAct и почему он важен?
4. Как AutoGen организует диалог между агентами?
5. Когда использовать Human-in-the-Loop?
6. Что такое checkpoint в LangGraph?

---

## 📚 РЕСУРСЫ

```bash
# Установка всех библиотек
pip install langchain langchain-openai langchain-community \
    langgraph crewai pyautogen
```

- [LangChain Docs](https://python.langchain.com/)
- [LangGraph Tutorials](https://langchain-ai.github.io/langgraph/)
- [CrewAI Docs](https://docs.crewai.com/)
- [AutoGen Docs](https://microsoft.github.io/autogen/)

---

**Следующий модуль**: Интеграция с внешним миром — браузеры, базы данных, мессенджеры! 🚀
