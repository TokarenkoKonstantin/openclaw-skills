# 🔄 Multi-Model Support - Поддержка нескольких моделей

## Описание навыка

OpenClaw поддерживает работу с множественными LLM провайдерами, обеспечивая гибкость, fallback и оптимизацию стоимости.

## 🎯 Возможности

### 1. Multi-Provider
- Поддержка 100+ LLM провайдеров
- Единый API для всех моделей
- Прозрачное переключение
- Vendor independence

### 2. Model Selection
- Выбор модели под задачу
- Routing по сложности
- Cost-based selection
- Performance-based selection

### 3. Fallback & Retry
- Автоматический fallback при ошибках
- Retry logic с backoff
- Health check провайдеров
- Graceful degradation

### 4. Cost Optimization
- Tracking расходов по провайдерам
- Выбор самой дешёвой модели
- Бюджетирование
- Alert при превышении

## 💬 Примеры использования

### Пример 1: Выбор модели под задачу
```python
from litellm import completion

def select_model(task_complexity: str):
    """Выбор модели на основе сложности задачи"""
    
    if task_complexity == "simple":
        # Простые задачи - дешёвая модель
        return "ollama/llama3"
    elif task_complexity == "medium":
        # Средние задачи - баланс цена/качество
        return "anthropic/claude-3-haiku-20240307"
    elif task_complexity == "complex":
        # Сложные задачи - лучшая модель
        return "openai/gpt-4-turbo-preview"
    else:
        # По умолчанию - средняя
        return "anthropic/claude-3-sonnet-20240229"

# Использование
model = select_model("complex")
response = completion(
    model=model,
    messages=[{"role": "user", "content": "Сложная задача..."}]
)
```

### Пример 2: Fallback при ошибке
```python
from litellm import completion
from typing import List

def completion_with_fallback(
    messages: List[dict],
    fallback_models: List[str] = None
):
    """Выполнение с автоматическим fallback"""
    
    if fallback_models is None:
        fallback_models = [
            "openai/gpt-4-turbo-preview",
            "anthropic/claude-3-opus-20240229",
            "anthropic/claude-3-sonnet-20240229",
            "ollama/llama3"  # Локальный fallback
        ]
    
    last_error = None
    
    for model in fallback_models:
        try:
            print(f"Trying {model}...")
            response = completion(
                model=model,
                messages=messages,
                timeout=30
            )
            print(f"Success with {model}")
            return response
        except Exception as e:
            print(f"Failed with {model}: {e}")
            last_error = e
            continue
    
    # Все модели не сработали
    raise Exception(f"All models failed. Last error: {last_error}")

# Использование
response = completion_with_fallback(
    messages=[{"role": "user", "content": "Привет!"}]
)
```

### Пример 3: Cost tracking
```python
from litellm import completion, cost_per_token
import json

class CostTracker:
    def __init__(self):
        self.total_cost = 0
        self.usage_by_model = {}
    
    def track_completion(self, model: str, response):
        """Отслеживание стоимости вызова"""
        
        usage = response.usage
        prompt_tokens = usage.prompt_tokens
        completion_tokens = usage.completion_tokens
        
        # Получаем стоимость
        prompt_cost, completion_cost = cost_per_token(
            model=model,
            prompt_tokens=prompt_tokens,
            completion_tokens=completion_tokens
        )
        
        total = prompt_cost + completion_cost
        self.total_cost += total
        
        # Обновляем статистику по модели
        if model not in self.usage_by_model:
            self.usage_by_model[model] = {
                "calls": 0,
                "cost": 0,
                "tokens": 0
            }
        
        self.usage_by_model[model]["calls"] += 1
        self.usage_by_model[model]["cost"] += total
        self.usage_by_model[model]["tokens"] += prompt_tokens + completion_tokens
        
        return response
    
    def get_report(self):
        """Отчёт по расходам"""
        return {
            "total_cost": self.total_cost,
            "by_model": self.usage_by_model
        }

# Использование
tracker = CostTracker()

response = completion(
    model="openai/gpt-4-turbo-preview",
    messages=[{"role": "user", "content": "Привет!"}]
)
tracker.track_completion("openai/gpt-4-turbo-preview", response)

print(tracker.get_report())
# {"total_cost": 0.01, "by_model": {...}}
```

## 🏗️ Архитектура

### Model Router:
```
┌─────────────────────────────────────────┐
│         Request Router                  │
│  - Анализ задачи                         │
│  - Выбор модели                         │
│  - Load balancing                       │
└─────────────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│         Model Gateway                   │
│  - Единый API                           │
│  - Трансформация запросов               │
│  - Трансформация ответов                │
└─────────────────────────────────────────┘
               │
    ┌──────────┼──────────┐
    ▼          ▼          ▼
┌────────┐ ┌────────┐ ┌────────┐
│ OpenAI │ │Anthropic│ │ Ollama │
│ GPT-4  │ │ Claude │ │Llama3  │
└────────┘ └────────┘ └────────┘
```

### Поддерживаемые провайдеры:
- **OpenAI** - GPT-4, GPT-3.5
- **Anthropic** - Claude 3 (Opus, Sonnet, Haiku)
- **Google** - Gemini Pro, Gemini Ultra
- **Meta** - Llama 2, Llama 3 (через Ollama)
- **Mistral** - Mistral, Mixtral
- **Cohere** - Command, Command R
- **Replicate** - различные open-source модели
- **Local** - Ollama, LM Studio

## 🛠️ Технологии

### Primary:
- **LiteLLM** - единый API для 100+ провайдеров
- **LangChain** - абстракция над моделями
- **Haystack** - production-ready

### Cost Tracking:
- **LiteLLM Proxy** - встроенный tracking
- **Custom middleware** - свой tracking
- **Provider dashboards** - нативные инструменты

### Monitoring:
- **Health checks** - доступность провайдеров
- **Latency tracking** - время ответа
- **Error rates** - частота ошибок
- **Cost alerts** - уведомления о расходах

## 📊 Метрики эффективности

- Cost savings (% экономии)
- Fallback success rate
- Average latency
- Provider uptime
- Cost per task

## ✅ Преимущества

- Гибкость в выборе модели
- Fallback при недоступности
- Оптимизация стоимости
- Vendor independence
- Лучшее соотношение цена/качество

## ⚠️ Ограничения

- Сложность управления несколькими API
- Разные форматы ответов
- Дополнительные зависимости
- Нужно тестировать на всех моделях

---

**Статус:** ✅ Одобрено 2026-03-24  
**Версия:** 1.0  
**Последнее обновление:** 2026-03-24
