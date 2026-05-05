# 📗 МОДУЛЬ 8: БОНУС — ПРОДВИНУТЫЕ ТЕМЫ
## Для тех, кто хочет большего

---

## ЧАСТЬ 1: FINE-TUNING МОДЕЛЕЙ

### 1.1 LoRA и QLoRA

Fine-tuning позволяет адаптировать модель под конкретную задачу без огромных затрат.

```python
# Пример с PEFT (Parameter-Efficient Fine-Tuning)
from peft import LoraConfig, get_peft_model
from transformers import AutoModelForCausalLM, AutoTokenizer, TrainingArguments
from trl import SFTTrainer

# Загружаем базовую модель
model = AutoModelForCausalLM.from_pretrained("microsoft/DialoGPT-medium")
tokenizer = AutoTokenizer.from_pretrained("microsoft/DialoGPT-medium")

# LoRA конфигурация
lora_config = LoraConfig(
    r=16,               # rank (размер адаптеров)
    lora_alpha=32,      # масштабирование
    target_modules=["q_proj", "v_proj"],  # какие слои адаптировать
    lora_dropout=0.05,
    bias="none",
    task_type="CAUSAL_LM"
)

# Применяем адаптеры
model = get_peft_model(model, lora_config)
model.print_trainable_parameters()
# Вывод: trainable params: 2M || all params: 344M || trainable%: 0.58%
```

### 1.2 Подготовка датасета

```python
# Формат данных для обучения
import json

training_data = [
    {
        "instruction": "Определи тональность текста",
        "input": "Этот продукт просто невероятный!",
        "output": "Позитивная"
    },
    {
        "instruction": "Определи тональность текста",
        "input": "Полный провал, никому не рекомендую",
        "output": "Негативная"
    }
]

# Сохраняем
with open("training.json", "w", encoding="utf-8") as f:
    json.dump(training_data, f, ensure_ascii=False, indent=2)
```

---

## ЧАСТЬ 2: МУЛЬТИМОДАЛЬНЫЕ АГЕНТЫ

### 2.1 Работа с изображениями

```python
from openai import OpenAI
import base64

client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

def image_to_base64(image_path):
    with open(image_path, "rb") as f:
        return base64.b64encode(f.read()).decode()

def analyze_image(image_path: str, question: str):
    """Агент анализирует изображение"""
    base64_image = image_to_base64(image_path)
    
    response = client.chat.completions.create(
        model="gpt-4o-mini",  # поддерживает vision
        messages=[
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": question},
                    {
                        "type": "image_url",
                        "image_url": {
                            "url": f"data:image/jpeg;base64,{base64_image}"
                        }
                    }
                ]
            }
        ],
        max_tokens=500
    )
    
    return response.choices[0].message.content

# Использование
result = analyze_image("screenshot.png", "Что изображено на картинке? Опиши детали.")
print(result)
```

### 2.2 Агент с генерацией изображений

```python
from openai import OpenAI

client = OpenAI()

def generate_image_tool(prompt: str) -> str:
    """Генерирует изображение по описанию"""
    response = client.images.generate(
        model="dall-e-3",
        prompt=prompt,
        size="1024x1024",
        quality="standard",
        n=1
    )
    return response.data[0].url

# Использование
url = generate_image_tool("A futuristic robot helping a human programmer")
print(f"Изображение: {url}")
```

---

## ЧАСТЬ 3: MCP (MODEL CONTEXT PROTOCOL)

### 3.1 Что такое MCP?

MCP (Model Context Protocol) от Anthropic — стандарт для подключения инструментов к LLM. Это как USB для AI-агентов.

```python
# Пример MCP-сервера
from mcp.server import Server
from mcp.types import Tool, TextContent

app = Server("my-agent-tools")

@app.list_tools()
async def list_tools():
    return [
        Tool(
            name="calculate",
            description="Калькулятор",
            inputSchema={"type": "object", "properties": {"expr": {"type": "string"}}}
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict):
    if name == "calculate":
        result = eval(arguments["expr"])
        return [TextContent(type="text", text=str(result))]
```

---

## ЧАСТЬ 4: САМООБУЧАЮЩИЕСЯ АГЕНТЫ

### 4.1 Обучение с подкреплением (RL)

```python
# Простой пример RL для агента
class LearningAgent:
    def __init__(self):
        self.success_history = []  # успешные действия
        self.failure_history = []  # неуспешные
        self.prompt_variants = []  # варианты промптов
    
    def learn_from_feedback(self, task: str, result: str, success: bool):
        """Учимся на обратной связи"""
        entry = {"task": task, "result": result}
        
        if success:
            self.success_history.append(entry)
        else:
            self.failure_history.append(entry)
    
    def get_improved_prompt(self, base_prompt: str) -> str:
        """Улучшаем промпт на основе истории"""
        # Анализируем успешные примеры
        patterns = self._extract_patterns(self.success_history)
        
        improved = f"""{base_prompt}

Учти следующие паттерны успешных решений:
{patterns}"""
        
        return improved
    
    def _extract_patterns(self, history):
        # Упрощённо — собираем общие элементы
        return "\n".join([f"- {h['task'][:50]}..." for h in history[-5:]])
```

### 4.2 Self-Reflection

```python
class ReflectiveAgent:
    """Агент, который анализирует свои ошибки"""
    
    def __init__(self):
        self.llm = ChatOpenAI(model="gpt-4o-mini")
    
    def run_with_reflection(self, task: str, max_attempts: int = 3):
        """Выполняет задачу с самооценкой"""
        
        for attempt in range(max_attempts):
            # Выполняем
            result = self._execute(task)
            
            # Оцениваем
            reflection = self._reflect(task, result)
            
            if reflection["is_correct"]:
                return result
            
            # Корректируем
            task = self._correct(task, reflection["feedback"])
        
        return result  # лучшее, что получилось
    
    def _reflect(self, task: str, result: str) -> dict:
        """Самоанализ"""
        prompt = f"""Оцени результат выполнения задачи:
        
Задача: {task}
Результат: {result}

Ответь в формате:
- Правильно: да/нет
- Проблемы: если есть
- Как улучшить: конкретные советы"""
        
        response = self.llm.invoke(prompt)
        content = response.content
        
        return {
            "is_correct": "да" in content.lower() and "нет" not in content.lower().split("правильно:")[-1][:20],
            "feedback": content
        }
    
    def _correct(self, task: str, feedback: str) -> str:
        """Исправляем задачу на основе фидбека"""
        return f"{task}\n\nУчти: {feedback}"
    
    def _execute(self, task: str) -> str:
        response = self.llm.invoke(task)
        return response.content
```

---

## ЧАСТЬ 5: ЛОКАЛЬНЫЕ МОДЕЛИ

### 5.1 Ollama — запуск локальных моделей

```bash
# Установка Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Запуск модели
ollama run llama3.2

# API-запрос к локальной модели
curl http://localhost:11434/api/generate -d '{
    "model": "llama3.2",
    "prompt": "Привет! Как дела?"
}'
```

```python
# Python-клиент для Ollama
import requests

class LocalLLM:
    def __init__(self, model="llama3.2"):
        self.model = model
        self.url = "http://localhost:11434/api/generate"
    
    def generate(self, prompt: str) -> str:
        response = requests.post(self.url, json={
            "model": self.model,
            "prompt": prompt,
            "stream": False
        })
        return response.json()["response"]

# Использование
llm = LocalLLM()
print(llm.generate("Напиши функцию для сортировки списка"))
```

### 5.2 Интеграция с LangChain

```python
from langchain_ollama import OllamaLLM

# Используем локальную модель в LangChain
llm = OllamaLLM(model="llama3.2")

result = llm.invoke("Объясни, что такое AI-агент")
print(result)
```

---

## ЧАСТЬ 6: ОПТИМИЗАЦИЯ СТОИМОСТИ

### 6.1 Управление токенами

```python
class TokenOptimizer:
    """Оптимизирует расход токенов"""
    
    def __init__(self):
        self.cache = {}  # кэш ответов
    
    def cached_invoke(self, prompt: str, llm):
        """Кэшируем повторяющиеся запросы"""
        # Простой хэш
        key = hash(prompt) % 1000000
        
        if key in self.cache:
            print("♻️ Кэш хит!")
            return self.cache[key]
        
        result = llm.invoke(prompt)
        self.cache[key] = result
        
        # Ограничиваем размер кэша
        if len(self.cache) > 1000:
            self.cache.pop(next(iter(self.cache)))
        
        return result
    
    def compress_context(self, messages: list, max_tokens: int = 2000) -> list:
        """Сжимаем контекст, если он слишком длинный"""
        total = sum(len(str(m)) for m in messages)
        
        if total <= max_tokens * 4:  # примерно 4 символа на токен
            return messages
        
        # Оставляем system и последние сообщения
        system_msgs = [m for m in messages if m.get("role") == "system"]
        recent = messages[-5:]  # последние 5
        
        return system_msgs + recent
```

### 6.2 Fallback-модели

```python
class FallbackLLM:
    """Автоматический переход на резервную модель"""
    
    def __init__(self):
        self.primary = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))
        self.fallback = OpenAI(
            api_key=os.getenv("GROQ_API_KEY"),
            base_url="https://api.groq.com/openai/v1"
        )
    
    def invoke(self, messages, primary_model="gpt-4o-mini", fallback_model="llama-3.2-90b"):
        try:
            # Пробуем основную модель
            response = self.primary.chat.completions.create(
                model=primary_model,
                messages=messages
            )
            return response.choices[0].message.content
        
        except Exception as e:
            print(f"⚠️ Primary failed: {e}. Switching to fallback...")
            
            # Переходим на резервную
            response = self.fallback.chat.completions.create(
                model=fallback_model,
                messages=messages
            )
            return response.choices[0].message.content
```

---

## 📋 ЗАДАНИЯ БОНУС-МОДУЛЯ

### Задание 1: Локальный агент (⭐⭐)
Создай агента, который работает полностью на локальной модели (Ollama) без API OpenAI.

### Задание 2: Мультимодальный агент (⭐⭐⭐)
Создай агента, который:
- Принимает изображения
- Описывает их
- Генерирует новые изображения по описанию

### Задание 3: Самообучающийся агент (⭐⭐⭐)
Создай агента, который:
- Запоминает успешные и неуспешные действия
- Улучшает промпты на основе истории
- Повышает точность с каждым использованием

---

## 🔗 ПОЛЕЗНЫЕ ССЫЛКИ

- [Ollama](https://ollama.com/) — локальные модели
- [Groq](https://groq.com/) — быстрый inference
- [Hugging Face](https://huggingface.co/) — модели и датасеты
- [Unsloth](https://unsloth.ai/) — быстрый fine-tuning
- [LLM Pricing](https://www.botgenuity.com/tools/llm-pricing) — сравнение цен

---

**Продолжай экспериментировать и создавать!** 🚀
