# 🚀 МОДУЛЬ 8: PRODUCTION И РАЗВЁРТЫВАНИЕ AI-АГЕНТОВ

## Цель модуля
Превратить прототип AI-агента в надёжное, масштабируемое production-приложение, готовое к работе с реальными пользователями.

**Длительность:** 2 недели (16-20 часов)

---

## 📖 СОДЕРЖАНИЕ МОДУЛЯ

1. [Архитектура production-систем](#1-архитектура-production-систем)
2. [Оптимизация моделей и стоимости](#2-оптимизация-моделей-и-стоимости)
3. [Создание интерфейсов и API](#3-создание-интерфейсов-и-api)
4. [Контейнеризация и деплой](#4-контейнеризация-и-деплой)
5. [Тестирование и мониторинг](#5-тестирование-и-мониторинг)
6. [Практическая работа](#6-практическая-работа)
7. [Финальный проект модуля](#7-финальный-проект-модуля)

---

## 1. АРХИТЕКТУРА PRODUCTION-СИСТЕМ

### 1.1 Микросервисы vs Монолит

#### Монолитная архитектура
**Преимущества:**
- Простота разработки и тестирования
- Легче деплой (одно приложение)
- Меньше накладных расходов на коммуникацию
- Подходит для стартапов и небольших проектов

**Недостатки:**
- Сложнее масштабировать отдельные компоненты
- Отказ одного компонента влияет на всю систему
- Труднее обновлять отдельные части

```python
# Пример монолитной структуры
"""
app/
├── main.py              # Точка входа
├── agent/
│   ├── core.py          # Логика агента
│   ├── tools.py         # Инструменты
│   └── memory.py        # Память
├── api/
│   └── routes.py        # API эндпоинты
├── database/
│   └── models.py        # Модели БД
└── config.py            # Конфигурация
"""
```

#### Микросервисная архитектура
**Преимущества:**
- Независимое масштабирование компонентов
- Отказоустойчивость (один сервис упал — другие работают)
- Разные технологии для разных сервисов
- Легче поддерживать и обновлять

**Недостатки:**
- Сложность оркестрации
- Накладные расходы на межсервисную коммуникацию
- Требует DevOps-компетенций

```python
# Пример микросервисной структуры
"""
Сервисы:
├── Agent Service        # Основная логика агента
├── API Gateway          # Маршрутизация запросов
├── Memory Service       # Управление памятью и контекстом
├── Tools Service        # Выполнение инструментов
├── Monitoring Service   # Логирование и метрики
└── Queue Service        # Очереди задач (Celery + Redis)
"""
```

**Рекомендация:** Начинайте с монолита, переходите на микросервисы при росте нагрузки (>1000 пользователей).

---

### 1.2 Очереди задач

Для обработки длительных операций (запросы к LLM, веб-скрапинг, анализ документов) используйте очереди задач.

#### Celery + Redis/RabbitMQ

**Установка:**
```bash
pip install celery[redis] redis
```

**Конфигурация Celery:**
```python
# celery_config.py
from celery import Celery

celery_app = Celery(
    'ai_agent',
    broker='redis://localhost:6379/0',
    backend='redis://localhost:6379/0'
)

celery_app.conf.update(
    task_serializer='json',
    accept_content=['json'],
    result_serializer='json',
    timezone='UTC',
    enable_utc=True,
    task_track_started=True,
    task_time_limit=300,  # 5 минут максимум
    task_soft_time_limit=240,  # Мягкий лимит 4 минуты
)
```

**Создание задачи:**
```python
# tasks.py
from celery_config import celery_app
import logging

logger = logging.getLogger(__name__)

@celery_app.task(bind=True, max_retries=3)
def process_agent_query(self, user_id: str, query: str, context: dict):
    """
    Асинхронная обработка запроса к агенту
    """
    try:
        logger.info(f"Processing query for user {user_id}")
        
        # Импортируем здесь, чтобы избежать циклических зависимостей
        from agent.core import Agent
        
        agent = Agent()
        response = agent.process(query, context)
        
        # Сохраняем результат в БД
        save_result(user_id, response)
        
        return {
            'status': 'success',
            'response': response,
            'tokens_used': response.get('tokens', 0)
        }
        
    except Exception as exc:
        logger.error(f"Error processing query: {exc}")
        # Повторная попытка через экспоненциальную задержку
        raise self.retry(exc=exc, countdown=2 ** self.request.retries)
```

**Запуск воркера:**
```bash
# Терминал 1: Запуск Redis
redis-server

# Терминал 2: Запуск Celery воркера
celery -A tasks worker --loglevel=info --concurrency=4

# Терминал 3: Запуск Flower для мониторинга (опционально)
celery -A tasks flower --port=5555
```

**Вызов задачи из API:**
```python
# api/routes.py
from fastapi import APIRouter, BackgroundTasks
from tasks import process_agent_query
import uuid

router = APIRouter()

@router.post("/query")
async def submit_query(user_id: str, query: str):
    """
    Отправка запроса в очередь
    """
    task_id = str(uuid.uuid4())
    
    # Асинхронный запуск задачи
    task = process_agent_query.delay(user_id, query, {})
    
    return {
        'task_id': task.id,
        'status': 'queued',
        'message': 'Запрос принят в обработку'
    }

@router.get("/task/{task_id}")
async def get_task_status(task_id: str):
    """
    Проверка статуса задачи
    """
    task = process_agent_query.AsyncResult(task_id)
    
    if task.state == 'PENDING':
        return {'status': 'pending', 'result': None}
    elif task.state == 'SUCCESS':
        return {'status': 'completed', 'result': task.result}
    elif task.state == 'FAILURE':
        return {'status': 'failed', 'error': str(task.info)}
    else:
        return {'status': task.state, 'result': None}
```

---

### 1.3 Кэширование с Redis

Кэширование ответов снижает стоимость API-вызовов и ускоряет ответы.

**Установка:**
```bash
pip install redis
docker run -d -p 6379:6379 redis:alpine
```

**Реализация кэша:**
```python
# cache.py
import redis
import json
import hashlib
from typing import Optional, Any
from datetime import timedelta

class ResponseCache:
    def __init__(self, host='localhost', port=6379, db=0):
        self.redis_client = redis.Redis(host=host, port=port, db=db, decode_responses=True)
        self.default_ttl = 3600  # 1 час
    
    def _generate_key(self, query: str, context: dict) -> str:
        """Генерация уникального ключа для запроса"""
        content = f"{query}:{json.dumps(context, sort_keys=True)}"
        hash_value = hashlib.md5(content.encode()).hexdigest()
        return f"agent:cache:{hash_value}"
    
    def get(self, query: str, context: dict) -> Optional[Any]:
        """Получение ответа из кэша"""
        key = self._generate_key(query, context)
        cached = self.redis_client.get(key)
        
        if cached:
            return json.loads(cached)
        return None
    
    def set(self, query: str, context: dict, response: Any, ttl: int = None):
        """Сохранение ответа в кэш"""
        key = self._generate_key(query, context)
        ttl = ttl or self.default_ttl
        
        self.redis_client.setex(
            key,
            ttl,
            json.dumps(response, ensure_ascii=False)
        )
    
    def invalidate(self, pattern: str = "*"):
        """Очистка кэша по паттерну"""
        keys = self.redis_client.keys(f"agent:cache:{pattern}")
        if keys:
            self.redis_client.delete(*keys)

# Использование
cache = ResponseCache()

def get_cached_or_generate(query: str, context: dict, generate_func):
    """
    Паттерн Cache-Aside
    """
    # Проверяем кэш
    cached_response = cache.get(query, context)
    if cached_response:
        print("✓ Ответ найден в кэше")
        return cached_response, True
    
    # Генерируем новый ответ
    print("⚡ Генерация нового ответа")
    response = generate_func(query, context)
    
    # Сохраняем в кэш
    cache.set(query, context, response)
    
    return response, False
```

---

### 1.4 Логирование и мониторинг

#### Структурированное логирование с Loguru

**Установка:**
```bash
pip install loguru
```

**Настройка:**
```python
# logging_config.py
from loguru import logger
import sys
from datetime import datetime

# Настройка форматирования
log_format = (
    "<green>{time:YYYY-MM-DD HH:mm:ss}</green> | "
    "<level>{level: <8}</level> | "
    "<cyan>{name}</cyan>:<cyan>{function}</cyan>:<cyan>{line}</cyan> | "
    "<level>{message}</level>"
)

# Добавление handlers
logger.remove()  # Удалить стандартный handler

# Консоль
logger.add(
    sys.stdout,
    format=log_format,
    level="INFO",
    colorize=True
)

# Файл с ротацией
logger.add(
    "logs/app_{time:YYYY-MM-DD}.log",
    format=log_format,
    level="DEBUG",
    rotation="00:00",
    retention="7 days",
    compression="zip"
)

# Файл для ошибок
logger.add(
    "logs/error_{time:YYYY-MM-DD}.log",
    format=log_format,
    level="ERROR",
    rotation="50 MB",
    retention="30 days"
)

# Использование
logger.info("Агент запущен")
logger.debug(f"Параметры запроса: {params}")
logger.warning("Токенов осталось мало: {tokens}", tokens=100)
logger.error("Ошибка при вызове инструмента: {error}", error=exc)
```

#### Мониторинг с Sentry

**Установка:**
```bash
pip install sentry-sdk[fastapi]
```

**Настройка:**
```python
# sentry_config.py
import sentry_sdk
from sentry_sdk.integrations.fastapi import FastApiIntegration
from sentry_sdk.integrations.logging import LoggingIntegration

sentry_sdk.init(
    dsn="YOUR_SENTRY_DSN",
    integrations=[
        FastApiIntegration(),
        LoggingIntegration(
            level=logging.INFO,
            event_level=logging.ERROR
        ),
    ],
    traces_sample_rate=1.0,  # 100% трассировка
    profiles_sample_rate=1.0,  # Профилирование
    environment="production",
    release="1.0.0"
)

# Добавление контекста
from sentry_sdk import set_tag, set_user, add_breadcrumb

set_tag("agent_type", "customer_support")
set_user({"id": user_id, "username": username})

add_breadcrumb(
    category="agent_action",
    message="Выполнен поиск в базе знаний",
    level="info",
    data={"tool": "knowledge_base", "query": query}
)
```

---

## 2. ОПТИМИЗАЦИЯ МОДЕЛЕЙ И СТОИМОСТИ

### 2.1 Оптимизация токенов

#### Стратегии экономии токенов

**1. Сжатие контекста:**
```python
def compress_context(messages: list, max_tokens: int = 2000) -> list:
    """
    Сжатие истории диалога до заданного количества токенов
    """
    if not messages:
        return messages
    
    # Оставляем системное сообщение
    system_message = [m for m in messages if m['role'] == 'system']
    
    # Оставляем последние N сообщений
    other_messages = [m for m in messages if m['role'] != 'system']
    
    # Если слишком много сообщений, суммаризируем старые
    if len(other_messages) > 10:
        old_messages = other_messages[:-10]
        recent_messages = other_messages[-10:]
        
        # Создаём саммари старых сообщений
        summary_text = "\n".join([f"{m['role']}: {m['content']}" for m in old_messages])
        
        summary_prompt = f"""
        Кратко суммаризируй следующую историю диалога (основные факты и решения):
        {summary_text}
        
        Саммари (2-3 предложения):
        """
        
        # Получаем саммари от LLM
        summary_response = llm.generate(summary_prompt, max_tokens=200)
        
        compressed_messages = system_message + [
            {'role': 'assistant', 'content': f"Контекст предыдущего диалога: {summary_response}"},
            *recent_messages
        ]
        
        return compressed_messages
    
    return messages
```

**2. Оптимизация промптов:**
```python
# ❌ Плохо: многословный промпт
BAD_PROMPT = """
Ты очень полезный ассистент, который всегда старается помочь пользователю 
решить его задачу максимально качественно и подробно. Пожалуйста, отвечай 
на вопросы пользователя как можно более развернуто и детально, приводя 
примеры и объяснения там, где это необходимо...
"""

# ✅ Хорошо: лаконичный промпт
GOOD_PROMPT = """
Роль: Эксперт по технической поддержке.
Задача: Решать проблемы пользователей четко и по делу.
Формат: Краткий ответ + шаги решения.
Ограничение: Максимум 150 слов.
"""
```

**3. Выбор модели под задачу:**
```python
MODEL_ROUTING = {
    'simple_qa': 'gpt-3.5-turbo',      # Простые вопросы
    'complex_reasoning': 'gpt-4-turbo', # Сложные рассуждения
    'code_generation': 'gpt-4-turbo',   # Генерация кода
    'summarization': 'gpt-3.5-turbo',   # Саммари
    'creative_writing': 'gpt-4-turbo',  # Креатив
}

def select_model(task_type: str, budget_tier: str = 'standard') -> str:
    """
    Выбор модели на основе типа задачи и бюджета
    """
    if budget_tier == 'economy':
        return 'gpt-3.5-turbo'
    elif budget_tier == 'premium':
        return 'gpt-4-turbo'
    else:
        return MODEL_ROUTING.get(task_type, 'gpt-3.5-turbo')
```

---

### 2.2 Кэширование ответов LLM

Используйте специализированные решения для кэширования LLM-ответов:

**LLM Cache с semantic search:**
```python
from sentence_transformers import SentenceTransformer
import numpy as np
from sklearn.metrics.pairwise import cosine_similarity

class SemanticCache:
    def __init__(self, similarity_threshold=0.95):
        self.model = SentenceTransformer('all-MiniLM-L6-v2')
        self.cache = {}  # {embedding_hash: response}
        self.embeddings = []
        self.threshold = similarity_threshold
    
    def _get_embedding(self, text: str) -> np.ndarray:
        return self.model.encode([text])[0]
    
    def find_similar(self, query: str) -> Optional[dict]:
        """Поиск семантически похожего запроса в кэше"""
        if not self.cache:
            return None
        
        query_embedding = self._get_embedding(query)
        
        # Вычисляем схожесть со всеми закэшированными запросами
        similarities = []
        for i, emb in enumerate(self.embeddings):
            sim = cosine_similarity([query_embedding], [emb])[0][0]
            similarities.append((i, sim))
        
        # Находим лучший матч
        best_match = max(similarities, key=lambda x: x[1])
        
        if best_match[1] >= self.threshold:
            cache_key = list(self.cache.keys())[best_match[0]]
            return self.cache[cache_key]
        
        return None
    
    def add(self, query: str, response: dict):
        """Добавление в кэш"""
        embedding = self._get_embedding(query)
        self.cache[query] = response
        self.embeddings.append(embedding)
```

---

### 2.3 Fallback-модели (Отказоустойчивость)

Реализуйте цепочку fallback-моделей для повышения надёжности:

```python
from typing import List, Optional
import time

class ModelFallbackChain:
    def __init__(self, models: List[str]):
        self.models = models
        self.model_errors = {model: 0 for model in models}
        self.max_errors = 3
        self.cooldown_period = 300  # 5 минут
    
    def get_available_model(self) -> Optional[str]:
        """Получение доступной модели с учётом ошибок"""
        current_time = time.time()
        
        for model in self.models:
            if self.model_errors[model] < self.max_errors:
                return model
        
        return None
    
    def record_error(self, model: str):
        """Запись ошибки модели"""
        self.model_errors[model] += 1
        logger.warning(f"Модель {model} имеет {self.model_errors[model]} ошибок")
    
    def reset_errors(self, model: str):
        """Сброс счётчика ошибок после успешного запроса"""
        self.model_errors[model] = 0
    
    def generate_with_fallback(self, prompt: str, **kwargs):
        """Генерация с автоматическим fallback"""
        last_error = None
        
        for model in self.models:
            try:
                logger.info(f"Попытка использования модели: {model}")
                
                response = call_llm_api(model=model, prompt=prompt, **kwargs)
                
                # Успех — сбрасываем ошибки
                self.reset_errors(model)
                return response
                
            except Exception as e:
                logger.error(f"Ошибка модели {model}: {e}")
                self.record_error(model)
                last_error = e
                continue
        
        # Все модели не работали
        raise Exception(f"All models failed. Last error: {last_error}")

# Использование
fallback_chain = ModelFallbackChain([
    'gpt-4-turbo',
    'gpt-3.5-turbo',
    'claude-3-haiku',
    'gemini-pro'
])

response = fallback_chain.generate_with_fallback(prompt, max_tokens=500)
```

---

### 2.4 Self-hosting моделей

Для снижения затрат рассмотрите локальное хостирование открытых моделей.

#### Ollama

**Установка:**
```bash
# macOS/Linux
curl -fsSL https://ollama.com/install.sh | sh

# Запуск сервера
ollama serve

# Скачивание модели
ollama pull llama3:8b
ollama pull mistral:7b
```

**Интеграция с Python:**
```python
import requests

class OllamaClient:
    def __init__(self, base_url="http://localhost:11434"):
        self.base_url = base_url
    
    def generate(self, model: str, prompt: str, **kwargs) -> str:
        """Генерация текста с помощью локальной модели"""
        payload = {
            "model": model,
            "prompt": prompt,
            "stream": False,
            **kwargs
        }
        
        response = requests.post(
            f"{self.base_url}/api/generate",
            json=payload
        )
        
        if response.status_code == 200:
            return response.json()["response"]
        else:
            raise Exception(f"Ollama error: {response.text}")

# Использование
ollama = OllamaClient()
response = ollama.generate("llama3:8b", "Объясни квантовую физику просто")
```

#### vLLM (Высокая производительность)

**Установка и запуск:**
```bash
pip install vllm

# Запуск сервера
python -m vllm.entrypoints.api_server \
    --model meta-llama/Llama-2-7b-chat-hf \
    --host 0.0.0.0 \
    --port 8000
```

**Преимущества vLLM:**
- PagedAttention для эффективного использования памяти
- В 24 раза быстрее naive implementation
- Поддержка continuous batching
- High throughput serving

---

## 3. СОЗДАНИЕ ИНТЕРФЕЙСОВ И API

### 3.1 REST API с FastAPI

**Установка:**
```bash
pip install fastapi uvicorn[standard] pydantic
```

**Полноценный пример API:**
```python
# main.py
from fastapi import FastAPI, HTTPException, Depends, Security
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from pydantic import BaseModel, Field
from typing import Optional, List, Dict, Any
from contextlib import asynccontextmanager
import uvicorn

from agent.core import Agent
from cache import ResponseCache
from logging_config import logger

# Модели данных
class QueryRequest(BaseModel):
    query: str = Field(..., min_length=1, max_length=5000, description="Текст запроса")
    context: Optional[Dict[str, Any]] = Field(default={}, description="Дополнительный контекст")
    stream: bool = Field(default=False, description="Потоковый ответ")

class QueryResponse(BaseModel):
    response: str
    tokens_used: int
    execution_time: float
    sources: Optional[List[str]] = None

class HealthResponse(BaseModel):
    status: str
    version: str
    active_models: List[str]

# Безопасность
security = HTTPBearer()
cache = ResponseCache()

def verify_token(credentials: HTTPAuthorizationCredentials = Security(security)):
    """Проверка API токена"""
    valid_tokens = ["your-secret-token-1", "your-secret-token-2"]
    if credentials.credentials not in valid_tokens:
        raise HTTPException(status_code=401, detail="Invalid token")
    return credentials.credentials

# Lifecycle management
@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup
    logger.info("🚀 Запуск AI Agent API")
    app.state.agent = Agent()
    app.state.cache = cache
    yield
    # Shutdown
    logger.info("🛑 Остановка AI Agent API")
    del app.state.agent

# Создание приложения
app = FastAPI(
    title="AI Agent API",
    description="REST API для взаимодействия с AI-агентом",
    version="1.0.0",
    lifespan=lifespan
)

# Endpoints
@app.get("/health", response_model=HealthResponse)
async def health_check():
    """Проверка здоровья сервиса"""
    return HealthResponse(
        status="healthy",
        version="1.0.0",
        active_models=["gpt-4-turbo", "gpt-3.5-turbo"]
    )

@app.post("/query", response_model=QueryResponse)
async def process_query(
    request: QueryRequest,
    token: str = Depends(verify_token)
):
    """
    Обработка запроса к AI-агенту
    
    - **query**: Текст запроса пользователя
    - **context**: Дополнительный контекст (опционально)
    - **stream**: Потоковый режим (опционально)
    """
    import time
    start_time = time.time()
    
    try:
        # Проверяем кэш
        cached_response, is_cached = cache.get(request.query, request.context), False
        
        if cached_response:
            logger.info(f"Ответ найден в кэше для запроса: {request.query[:50]}...")
            return QueryResponse(
                response=cached_response['response'],
                tokens_used=cached_response['tokens_used'],
                execution_time=time.time() - start_time,
                sources=cached_response.get('sources')
            )
        
        # Обрабатываем запрос
        agent = app.state.agent
        result = await agent.process_async(
            request.query,
            request.context,
            stream=request.stream
        )
        
        # Сохраняем в кэш
        cache.set(
            request.query,
            request.context,
            {
                'response': result['response'],
                'tokens_used': result['tokens_used'],
                'sources': result.get('sources')
            }
        )
        
        logger.info(f"Запрос обработан за {time.time() - start_time:.2f} сек")
        
        return QueryResponse(
            response=result['response'],
            tokens_used=result['tokens_used'],
            execution_time=time.time() - start_time,
            sources=result.get('sources')
        )
        
    except Exception as e:
        logger.error(f"Ошибка обработки запроса: {e}")
        raise HTTPException(status_code=500, detail=str(e))

@app.get("/metrics")
async def get_metrics():
    """Получение метрик сервиса"""
    return {
        "cache_size": len(cache.redis_client.keys("agent:cache:*")),
        "uptime": "2h 15m",
        "requests_processed": 1250,
        "average_response_time": "1.2s"
    }

if __name__ == "__main__":
    uvicorn.run(
        "main:app",
        host="0.0.0.0",
        port=8000,
        reload=True,
        log_level="info"
    )
```

**Запуск API:**
```bash
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

**Документация автоматически доступна:**
- Swagger UI: `http://localhost:8000/docs`
- ReDoc: `http://localhost:8000/redoc`

---

### 3.2 Веб-интерфейс с Streamlit

**Установка:**
```bash
pip install streamlit
```

**Создание интерфейса:**
```python
# app.py
import streamlit as st
import requests
import json
from datetime import datetime

# Конфигурация страницы
st.set_page_config(
    page_title="AI Агент",
    page_icon="🤖",
    layout="wide",
    initial_sidebar_state="expanded"
)

# Заголовок
st.title("🤖 AI Агент")
st.markdown("Интерактивный интерфейс для взаимодействия с AI-агентом")

# Боковая панель
with st.sidebar:
    st.header("Настройки")
    
    api_url = st.text_input(
        "API URL",
        value="http://localhost:8000",
        help="URL вашего API сервера"
    )
    
    api_token = st.text_input(
        "API Token",
        type="password",
        value="your-secret-token-1",
        help="Токен для авторизации"
    )
    
    st.divider()
    
    # Статус подключения
    try:
        health_response = requests.get(f"{api_url}/health", timeout=5)
        if health_response.status_code == 200:
            st.success("✅ API подключено")
            st.json(health_response.json())
        else:
            st.error("❌ Ошибка подключения")
    except:
        st.error("❌ API недоступно")
    
    st.divider()
    
    # История сессии
    if 'messages' not in st.session_state:
        st.session_state.messages = []
    
    if st.button("🗑️ Очистить историю"):
        st.session_state.messages = []
        st.rerun()

# Отображение истории чата
for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])
        if "sources" in message and message["sources"]:
            with st.expander("📚 Источники"):
                for source in message["sources"]:
                    st.markdown(f"- {source}")

# Поле ввода запроса
if prompt := st.chat_input("Введите ваш запрос..."):
    # Добавляем сообщение пользователя
    st.session_state.messages.append({"role": "user", "content": prompt})
    
    with st.chat_message("user"):
        st.markdown(prompt)
    
    # Отправляем запрос к API
    with st.chat_message("assistant"):
        message_placeholder = st.empty()
        message_placeholder.markdown("⏳ Думаю...")
        
        try:
            response = requests.post(
                f"{api_url}/query",
                json={
                    "query": prompt,
                    "context": {"history": st.session_state.messages[-5:]}
                },
                headers={"Authorization": f"Bearer {api_token}"},
                timeout=60
            )
            
            if response.status_code == 200:
                result = response.json()
                
                # Отображаем ответ
                message_placeholder.markdown(result["response"])
                
                # Показываем метрики
                col1, col2 = st.columns(2)
                with col1:
                    st.metric("Токенов использовано", result["tokens_used"])
                with col2:
                    st.metric("Время выполнения", f"{result['execution_time']:.2f} сек")
                
                # Источники
                if result.get("sources"):
                    with st.expander("📚 Источники"):
                        for source in result["sources"]:
                            st.markdown(f"- {source}")
                
                # Сохраняем в историю
                st.session_state.messages.append({
                    "role": "assistant",
                    "content": result["response"],
                    "sources": result.get("sources")
                })
                
            else:
                message_placeholder.error(f"Ошибка API: {response.text}")
                
        except Exception as e:
            message_placeholder.error(f"Ошибка подключения: {str(e)}")

# Футер
st.divider()
st.markdown(
    """
    <div style='text-align: center; color: gray;'>
        <small>AI Agent v1.0.0 | Powered by LLM</small>
    </div>
    """,
    unsafe_allow_html=True
)
```

**Запуск:**
```bash
streamlit run app.py --server.port 8501
```

---

### 3.3 Интерфейс с Gradio (альтернатива)

```python
import gradio as gr
import requests

API_URL = "http://localhost:8000"

def chat_with_agent(message, history):
    """Обработка сообщения через API"""
    try:
        response = requests.post(
            f"{API_URL}/query",
            json={"query": message},
            timeout=60
        )
        
        if response.status_code == 200:
            result = response.json()
            return result["response"]
        else:
            return f"Ошибка API: {response.text}"
    
    except Exception as e:
        return f"Ошибка подключения: {str(e)}"

# Создание интерфейса
demo = gr.ChatInterface(
    fn=chat_with_agent,
    title="🤖 AI Агент",
    description="Задавайте вопросы AI-агенту",
    examples=["Что такое машинное обучение?", "Напиши код на Python", "Объясни квантовую физику"],
    theme="soft"
)

if __name__ == "__main__":
    demo.launch(server_name="0.0.0.0", server_port=7860)
```

---

## 4. КОНТЕЙНЕРИЗАЦИЯ И ДЕПЛОЙ

### 4.1 Docker-контейнеризация

**Создание Dockerfile:**
```dockerfile
# Dockerfile
FROM python:3.11-slim

# Рабочая директория
WORKDIR /app

# Установка системных зависимостей
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Копирование requirements
COPY requirements.txt .

# Установка Python зависимостей
RUN pip install --no-cache-dir -r requirements.txt

# Копирование кода
COPY . .

# Создание директорий для логов и данных
RUN mkdir -p logs data

# Переменные окружения
ENV PYTHONUNBUFFERED=1
ENV LOG_LEVEL=INFO

# Порт приложения
EXPOSE 8000

# Команда запуска
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  # AI Agent API
  agent-api:
    build: .
    ports:
      - "8000:8000"
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - REDIS_URL=redis://redis:6379/0
      - DATABASE_URL=postgresql://user:password@db:5432/agentdb
    depends_on:
      - redis
      - db
    volumes:
      - ./logs:/app/logs
      - ./data:/app/data
    restart: unless-stopped
    networks:
      - agent-network

  # Redis для кэша и очередей
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    restart: unless-stopped
    networks:
      - agent-network
    command: redis-server --appendonly yes

  # PostgreSQL для хранения данных
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: agentdb
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
    restart: unless-stopped
    networks:
      - agent-network

  # Celery Worker
  celery-worker:
    build: .
    command: celery -A tasks worker --loglevel=info --concurrency=4
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - REDIS_URL=redis://redis:6379/0
      - DATABASE_URL=postgresql://user:password@db:5432/agentdb
    depends_on:
      - redis
      - db
    volumes:
      - ./logs:/app/logs
    restart: unless-stopped
    networks:
      - agent-network

  # Flower для мониторинга Celery
  flower:
    build: .
    command: celery -A tasks flower --port=5555
    ports:
      - "5555:5555"
    environment:
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - redis
    restart: unless-stopped
    networks:
      - agent-network

  # Streamlit UI
  streamlit-ui:
    build: .
    command: streamlit run app.py --server.port 8501 --server.address 0.0.0.0
    ports:
      - "8501:8501"
    environment:
      - API_URL=http://agent-api:8000
    depends_on:
      - agent-api
    restart: unless-stopped
    networks:
      - agent-network

volumes:
  redis-data:
  postgres-data:

networks:
  agent-network:
    driver: bridge
```

**.env файл:**
```bash
# .env
OPENAI_API_KEY=sk-your-api-key-here
DATABASE_URL=postgresql://user:password@db:5432/agentdb
REDIS_URL=redis://redis:6379/0
LOG_LEVEL=INFO
```

**Запуск всего стека:**
```bash
# Сборка и запуск
docker-compose up --build -d

# Просмотр логов
docker-compose logs -f

# Остановка
docker-compose down

# Остановка с удалением volumes
docker-compose down -v
```

---

### 4.2 Деплой на Railway

**Шаги:**

1. **Подготовка репозитория:**
```bash
git init
git add .
git commit -m "Initial commit"
git push origin main
```

2. **Создание файла railway.toml:**
```toml
# railway.toml
[build]
builder = "NIXPACKS"

[deploy]
startCommand = "uvicorn main:app --host 0.0.0.0 --port $PORT"
restartPolicyType = "ON_FAILURE"
restartPolicyMaxRetries = 10
```

3. **nixpacks.toml для настройки сборки:**
```toml
# nixpacks.toml
[phases.setup]
nixPkgs = ["python311"]

[phases.install]
cmds = ["pip install -r requirements.txt"]

[phases.build]
cmds = ["echo 'Build complete'"]

[start]
cmd = "uvicorn main:app --host 0.0.0.0 --port $PORT"
```

4. **Деплой через CLI:**
```bash
# Установка Railway CLI
npm install -g @railway/cli

# Логин
railway login

# Инициализация проекта
railway init

# Деплой
railway up
```

5. **Настройка переменных окружения:**
```bash
railway variables set OPENAI_API_KEY=sk-...
railway variables set REDIS_URL=your-redis-url
```

---

### 4.3 Деплой на Render

**render.yaml:**
```yaml
services:
  - type: web
    name: ai-agent-api
    env: python
    region: frankfurt
    plan: starter
    buildCommand: pip install -r requirements.txt
    startCommand: uvicorn main:app --host 0.0.0.0 --port $PORT
    envVars:
      - key: OPENAI_API_KEY
        sync: false
      - key: REDIS_URL
        fromService:
          name: ai-agent-redis
          type: redis
          property: connectionString
    
  - type: worker
    name: celery-worker
    env: python
    buildCommand: pip install -r requirements.txt
    startCommand: celery -A tasks worker --loglevel=info
    envVars:
      - key: OPENAI_API_KEY
        sync: false
      - key: REDIS_URL
        fromService:
          name: ai-agent-redis
          type: redis
          property: connectionString
  
  - type: redis
    name: ai-agent-redis
    ipAllowList: []
    plan: starter
    region: frankfurt
```

**Деплой:**
1. Запушьте код на GitHub
2. Подключите репозиторий в Render
3. Render автоматически прочитает render.yaml и развернёт сервисы

---

### 4.4 Деплой на AWS (ECS Fargate)

**terraform/main.tf (упрощённо):**
```hcl
provider "aws" {
  region = "eu-central-1"
}

resource "aws_ecs_cluster" "agent_cluster" {
  name = "ai-agent-cluster"
}

resource "aws_ecs_task_definition" "agent_task" {
  family                   = "ai-agent"
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  cpu                      = 1024
  memory                   = 2048
  execution_role_arn       = aws_iam_role.ecs_execution_role.arn

  container_definitions = jsonencode([{
    name  = "agent-api"
    image = "${aws_ecr_repository.agent_repo.repository_url}:latest"
    portMappings = [{
      containerPort = 8000
      protocol      = "tcp"
    }]
    environment = [
      { name = "OPENAI_API_KEY", value = var.openai_api_key }
    ]
    logConfiguration = {
      logDriver = "awslogs"
      options = {
        awslogs-group         = "/ecs/ai-agent"
        awslogs-region        = "eu-central-1"
        awslogs-stream-prefix = "agent"
      }
    }
  }])
}

resource "aws_ecs_service" "agent_service" {
  name            = "ai-agent-service"
  cluster         = aws_ecs_cluster.agent_cluster.id
  task_definition = aws_ecs_task_definition.agent_task.arn
  desired_count   = 2
  launch_type     = "FARGATE"
}
```

---

## 5. ТЕСТИРОВАНИЕ И МОНИТОРИНГ

### 5.1 Unit-тесты для инструментов

**pytest конфигурация:**
```bash
pip install pytest pytest-asyncio pytest-cov
```

**tests/test_tools.py:**
```python
import pytest
from unittest.mock import Mock, patch, AsyncMock
from agent.tools import WeatherTool, SearchTool, CalculatorTool

class TestCalculatorTool:
    @pytest.fixture
    def calculator(self):
        return CalculatorTool()
    
    def test_addition(self, calculator):
        result = calculator.execute("2 + 2")
        assert result == 4
    
    def test_subtraction(self, calculator):
        result = calculator.execute("10 - 3")
        assert result == 7
    
    def test_multiplication(self, calculator):
        result = calculator.execute("5 * 6")
        assert result == 30
    
    def test_division(self, calculator):
        result = calculator.execute("20 / 4")
        assert result == 5
    
    def test_division_by_zero(self, calculator):
        with pytest.raises(ValueError):
            calculator.execute("10 / 0")
    
    def test_invalid_expression(self, calculator):
        with pytest.raises(ValueError):
            calculator.execute("invalid")

class TestWeatherTool:
    @pytest.fixture
    def weather_tool(self):
        return WeatherTool(api_key="test-key")
    
    @pytest.mark.asyncio
    async def test_get_weather_success(self, weather_tool):
        with patch('requests.get') as mock_get:
            mock_get.return_value.json.return_value = {
                'main': {'temp': 20},
                'weather': [{'description': 'clear'}]
            }
            
            result = await weather_tool.execute("Moscow")
            
            assert 'temperature' in result
            assert result['temperature'] == 20
    
    @pytest.mark.asyncio
    async def test_get_weather_city_not_found(self, weather_tool):
        with patch('requests.get') as mock_get:
            mock_get.return_value.status_code = 404
            
            with pytest.raises(Exception):
                await weather_tool.execute("InvalidCity123")

class TestSearchTool:
    @pytest.fixture
    def search_tool(self):
        return SearchTool()
    
    @pytest.mark.asyncio
    async def test_search_returns_results(self, search_tool):
        with patch('duckduckgo_search.search') as mock_search:
            mock_search.return_value = [
                {'title': 'Test Result', 'href': 'https://example.com'}
            ]
            
            results = await search_tool.execute("Python programming")
            
            assert len(results) > 0
            assert 'title' in results[0]
```

**Запуск тестов:**
```bash
# Запустить все тесты
pytest

# Запустить с покрытием
pytest --cov=agent --cov-report=html

# Запустить конкретный файл
pytest tests/test_tools.py -v

# Запустить асинхронные тесты
pytest tests/ -v --asyncio-mode=auto
```

---

### 5.2 Evals для LLM

Использование фреймворка для оценки качества ответов LLM.

**Установка:**
```bash
pip install deepeval
```

**tests/test_llm_quality.py:**
```python
from deepeval import assert_test
from deepeval.test_case import LLMTestCase, LLMTestCaseParams
from deepeval.metrics import AnswerRelevancyMetric, FaithfulnessMetric, ContextualRelevancyMetric

def test_answer_relevancy():
    """Тест на релевантность ответа"""
    metric = AnswerRelevancyMetric(threshold=0.7)
    
    test_case = LLMTestCase(
        input="Как установить Python на Ubuntu?",
        actual_output="Для установки Python на Ubuntu выполните: sudo apt-get install python3",
        retrieval_context=["Ubuntu использует пакетный менеджер apt"]
    )
    
    assert_test(test_case, [metric])

def test_faithfulness():
    """Тест на достоверность (отсутствие галлюцинаций)"""
    metric = FaithfulnessMetric(threshold=0.8)
    
    test_case = LLMTestCase(
        input="Какая столица Франции?",
        actual_output="Столица Франции - Париж",
        retrieval_context=["Париж является столицей Франции с 987 года"]
    )
    
    assert_test(test_case, [metric])

def test_contextual_relevancy():
    """Тест на релевантность контекста"""
    metric = ContextualRelevancyMetric(threshold=0.7)
    
    test_case = LLMTestCase(
        input="Как работает трансформер?",
        actual_output="Трансформеры используют механизм внимания...",
        retrieval_context=[
            "Архитектура трансформера основана на attention mechanism",
            "Self-attention позволяет модели учитывать контекст"
        ]
    )
    
    assert_test(test_case, [metric])
```

**Запуск evals:**
```bash
deepeval test run tests/test_llm_quality.py
```

---

### 5.3 A/B тестирование промптов

```python
# ab_testing.py
import random
from typing import Dict, List
from dataclasses import dataclass
from datetime import datetime

@dataclass
class PromptVariant:
    name: str
    prompt_template: str
    traffic_percentage: float  # 0.0 to 1.0

@dataclass
class ABTestResult:
    variant_name: str
    total_requests: int
    success_count: int
    avg_response_time: float
    avg_tokens_used: float
    user_satisfaction_score: float  # 1-5

class PromptABTester:
    def __init__(self, variants: List[PromptVariant]):
        self.variants = variants
        self.results: Dict[str, ABTestResult] = {}
        
        # Инициализация результатов
        for variant in variants:
            self.results[variant.name] = ABTestResult(
                variant_name=variant.name,
                total_requests=0,
                success_count=0,
                avg_response_time=0.0,
                avg_tokens_used=0.0,
                user_satisfaction_score=0.0
            )
    
    def select_variant(self) -> PromptVariant:
        """Выбор варианта на основе распределения трафика"""
        rand = random.random()
        cumulative = 0.0
        
        for variant in self.variants:
            cumulative += variant.traffic_percentage
            if rand <= cumulative:
                return variant
        
        return self.variants[-1]  # Fallback
    
    def record_result(
        self,
        variant_name: str,
        success: bool,
        response_time: float,
        tokens_used: int,
        user_rating: int = None
    ):
        """Запись результата теста"""
        result = self.results[variant_name]
        
        result.total_requests += 1
        if success:
            result.success_count += 1
        
        # Обновление средних значений
        n = result.total_requests
        result.avg_response_time = (
            (result.avg_response_time * (n - 1) + response_time) / n
        )
        result.avg_tokens_used = (
            (result.avg_tokens_used * (n - 1) + tokens_used) / n
        )
        
        if user_rating:
            # Скользящее среднее для рейтинга
            result.user_satisfaction_score = (
                (result.user_satisfaction_score * (n - 1) + user_rating) / n
            )
    
    def get_winner(self) -> PromptVariant:
        """Определение победителя по комбинации метрик"""
        best_score = 0
        best_variant = None
        
        for variant in self.variants:
            result = self.results[variant.name]
            
            if result.total_requests == 0:
                continue
            
            # Композитный скор
            success_rate = result.success_count / result.total_requests
            score = (
                success_rate * 0.4 +
                (1 / (1 + result.avg_response_time)) * 0.3 +
                (result.user_satisfaction_score / 5) * 0.3
            )
            
            if score > best_score:
                best_score = score
                best_variant = variant
        
        return best_variant

# Пример использования
variants = [
    PromptVariant(
        name="verbose",
        prompt_template="Ты очень полезный ассистент... (подробный)",
        traffic_percentage=0.5
    ),
    PromptVariant(
        name="concise",
        prompt_template="Роль: Ассистент. Задача: Помочь. Формат: Кратко.",
        traffic_percentage=0.5
    )
]

ab_tester = PromptABTester(variants)

# В production
variant = ab_tester.select_variant()
# ... обработка запроса ...
ab_tester.record_result(
    variant_name=variant.name,
    success=True,
    response_time=1.5,
    tokens_used=250,
    user_rating=5
)
```

---

### 5.4 Нагрузочное тестирование

**Locust для нагрузочного тестирования:**
```python
# locustfile.py
from locust import HttpUser, task, between
import random

class AgentUser(HttpUser):
    wait_time = between(1, 3)  # Пауза между запросами 1-3 секунды
    
    queries = [
        "Что такое машинное обучение?",
        "Напиши функцию на Python для сортировки",
        "Объясни теорию относительности",
        "Как создать веб-сайт?",
        "Реши уравнение: x^2 + 5x + 6 = 0"
    ]
    
    @task(3)
    def simple_query(self):
        """Частые простые запросы"""
        self.client.post(
            "/query",
            json={
                "query": random.choice(self.queries[:3]),
                "context": {}
            },
            headers={"Authorization": "Bearer your-token"}
        )
    
    @task(1)
    def complex_query(self):
        """Редкие сложные запросы"""
        self.client.post(
            "/query",
            json={
                "query": random.choice(self.queries[3:]),
                "context": {"complexity": "high"}
            },
            headers={"Authorization": "Bearer your-token"}
        )
    
    @task(2)
    def health_check(self):
        """Проверка здоровья"""
        self.client.get("/health")
```

**Запуск Locust:**
```bash
# Локальный запуск с веб-интерфейсом
locust -f locustfile.py --host=http://localhost:8000

# Запуск без интерфейса (headless)
locust -f locustfile.py --host=http://localhost:8000 --headless -u 100 -r 10 --run-time 5m

# Параметры:
# -u 100: 100 одновременных пользователей
# -r 10: 10 пользователей в секунду hatch rate
# --run-time 5m: длительность теста
```

---

## 6. ПРАКТИЧЕСКАЯ РАБОТА

### Задание 1: Обёртка агента в FastAPI (2 часа)

**Цель:** Создать REST API для вашего AI-агента.

**Требования:**
1. Эндпоинт POST `/query` для обработки запросов
2. Эндпоинт GET `/health` для проверки здоровья
3. Аутентификация через API токен
4. Валидация входных данных с Pydantic
5. Логирование всех запросов

**Критерии успеха:**
- API отвечает за < 5 секунд
- Swagger документация доступна
- Все запросы логируются

---

### Задание 2: Streamlit интерфейс (2 часа)

**Цель:** Создать веб-интерфейс для взаимодействия с агентом.

**Требования:**
1. Чат-интерфейс с историей сообщений
2. Боковая панель с настройками
3. Индикатор статуса API
4. Отображение метрик (токены, время)
5. Кнопка очистки истории

**Критерии успеха:**
- Интерфейс работает локально
- История сохраняется в сессии
- Красивый UX/UI

---

### Задание 3: Docker контейнеризация (2 часа)

**Цель:** Упаковать приложение в Docker контейнеры.

**Требования:**
1. Dockerfile для основного приложения
2. docker-compose.yml с сервисами:
   - API (FastAPI)
   - Redis (кэш)
   - PostgreSQL (БД)
   - Celery Worker
3. Правильная настройка networking
4. Volumes для персистентности данных

**Критерии успеха:**
- `docker-compose up` запускает всё
- Сервисы видят друг друга
- Данные сохраняются после перезапуска

---

### Задание 4: Написание тестов (2 часа)

**Цель:** Покрыть код тестами.

**Требования:**
1. Unit-тесты для инструментов (минимум 5 тестов)
2. Integration тест для API эндпоинта
3. Evals для качества ответов LLM
4. Покрытие кода > 70%

**Критерии успеха:**
- Все тесты проходят
- CI/CD пайплайн зелёный
- Нет критических багов

---

### Задание 5: Деплой на облачную платформу (3 часа)

**Цель:** Задеплоить приложение на Railway/Render.

**Требования:**
1. Подготовка репозитория на GitHub
2. Настройка переменных окружения
3. Деплой через CLI или веб-интерфейс
4. Проверка работы в production

**Критерии успеха:**
- Приложение доступно по публичному URL
- API отвечает на запросы
- Логи доступны для просмотра

---

## 7. ФИНАЛЬНЫЙ ПРОЕКТ МОДУЛЯ

### Production-ready AI Агент

**Задача:** Создать полноценное production-приложение с AI-агентом.

**Требования:**

#### Архитектура
- [ ] Микросервисная или монолитная архитектура (обосновать выбор)
- [ ] Очереди задач (Celery + Redis)
- [ ] Кэширование ответов
- [ ] База данных для хранения истории

#### Функциональность
- [ ] REST API с FastAPI
- [ ] Веб-интерфейс (Streamlit/Gradio)
- [ ] Минимум 3 инструмента у агента
- [ ] Память и контекст диалога

#### Infrastructure
- [ ] Docker контейнеризация
- [ ] docker-compose для локальной разработки
- [ ] Деплой на облачную платформу
- [ ] Переменные окружения через .env

#### Качество
- [ ] Unit-тесты (покрытие > 70%)
- [ ] Evals для LLM ответов
- [ ] Логирование с Loguru
- [ ] Мониторинг ошибок (Sentry)

#### Документация
- [ ] README с инструкцией по запуску
- [ ] API документация (Swagger)
- [ ] Описание архитектуры

**Примеры идей для проекта:**
1. **Customer Support Bot** — автоматическая поддержка клиентов с интеграцией в базу знаний
2. **Research Assistant** — агент для исследования тем с поиском в интернете и генерацией отчётов
3. **Code Review Bot** — анализ кода, поиск багов, рекомендации по улучшению
4. **Personal Finance Advisor** — анализ расходов, рекомендации по бюджету, прогнозы
5. **Content Creator** — генерация статей, постов для соцсетей с SEO оптимизацией

**Критерии оценки:**
- ✅ Работоспособность (50%)
- ✅ Код качество (20%)
- ✅ Документация (15%)
- ✅ Тесты (10%)
- ✅ Деплой (5%)

---

## 📚 ДОПОЛНИТЕЛЬНЫЕ РЕСУРСЫ

### Книги и статьи
- "Designing Machine Learning Systems" — Chip Huyen
- "Building LLM Applications for Production" — DeepLearning.AI
- "The Twelve-Factor App" — https://12factor.net/

### Документация
- [FastAPI Docs](https://fastapi.tiangolo.com/)
- [Docker Docs](https://docs.docker.com/)
- [Celery Docs](https://docs.celeryq.dev/)
- [Streamlit Docs](https://docs.streamlit.io/)

### Инструменты
- [Railway](https://railway.app/) — простой деплой
- [Render](https://render.com/) — альтернатива Heroku
- [HuggingFace Spaces](https://huggingface.co/spaces) — бесплатный хостинг ML приложений
- [Fly.io](https://fly.io/) — деплой близко к пользователям

### Сообщества
- r/MLOps — DevOps для ML
- r/FastAPI — сообщество FastAPI
- Discord: MLOps.community

---

## ✅ ЧЕКЛИСТ ЗАВЕРШЕНИЯ МОДУЛЯ

- [ ] Понимаю разницу между монолитом и микросервисами
- [ ] Реализовал очереди задач с Celery
- [ ] Настроил кэширование с Redis
- [ ] Создал REST API с FastAPI
- [ ] Построил веб-интерфейс (Streamlit/Gradio)
- [ ] Упаковал приложение в Docker
- [ ] Задеплоил на облачную платформу
- [ ] Написал unit-тесты и evals
- [ ] Настроил логирование и мониторинг
- [ ] completed финальный проект

---

## 🎯 СЛЕДУЮЩИЙ ШАГ

Поздравляем! Вы освоили production-развёртывание AI-агентов. 

**Далее:** Переходите к **Модулю 9** для изучения продвинутых тем:
- Fine-tuning моделей
- Мультимодальные агенты
- MCP (Model Context Protocol)
- Самообучающиеся системы

Или сразу приступайте к **финальному проекту курса**, объединяющему все полученные знания!
