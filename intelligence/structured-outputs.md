# 📊 Structured Outputs - Структурированные ответы

## Описание навыка

OpenClaw использует Pydantic models и схемы для генерации структурированных, предсказуемых и валидированных ответов.

## 🎯 Возможности

### 1. Schema-based Responses
- Определение схем для каждого типа ответа
- Валидация выходных данных
- Type safety
- Консистентность форматов

### 2. Data Extraction
- Извлечение структурированных данных из текста
- Парсинг сущностей (имена, даты, суммы)
- Классификация и категоризация
- Нормализация форматов

### 3. API Responses
- JSON-формат для интеграций
- Предсказуемая структура
- Документированные поля
- Версионность схем

### 4. Error Handling
- Валидация с понятными ошибками
- Fallback на дефолтные значения
- Retry logic при ошибках
- Graceful degradation

## 💬 Примеры использования

### Пример 1: Извлечение данных
```python
from pydantic import BaseModel, Field
from typing import List, Optional

class ContactInfo(BaseModel):
    name: str = Field(description="Полное имя")
    email: Optional[str] = Field(description="Email адрес")
    phone: Optional[str] = Field(description="Номер телефона")
    location: Optional[str] = Field(description="Город/страна")

class ProjectInfo(BaseModel):
    name: str = Field(description="Название проекта")
    status: str = Field(description="Статус: active/completed/on_hold")
    technologies: List[str] = Field(description="Используемые технологии")
    description: str = Field(description="Краткое описание")

# Использование
text = """
Проект: Neuroforge AI
Статус: активен
Стек: Python, LangChain, ChromaDB
Описание: AI портфолио с модулями для чат-ботов
Контакты: consttokarenko@gmail.com, Челябинск
"""

result = extract_structured(text, ProjectInfo)
# result.name = "Neuroforge AI"
# result.status = "active"
# result.technologies = ["Python", "LangChain", "ChromaDB"]
```

### Пример 2: Ответы в формате JSON
```python
class OpenClawResponse(BaseModel):
    success: bool = Field(description="Успешно ли выполнено")
    data: dict = Field(description="Данные ответа")
    message: str = Field(description="Человекочитаемое сообщение")
    sources: List[str] = Field(default=[], description="Источники информации")
    confidence: float = Field(ge=0, le=1, description="Уверенность в ответе")

# Пример ответа
{
    "success": true,
    "data": {
        "task": "cron_setup",
        "status": "completed",
        "files_created": ["trigger-cycle.sh", "CRON-SETUP.md"]
    },
    "message": "Cron настроен для запуска каждые 12 часов",
    "sources": ["~/kiro-kaizen/CRON-SETUP.md"],
    "confidence": 0.95
}
```

### Пример 3: Валидация ввода
```python
from pydantic import validator, ValidationError
from datetime import datetime

class TaskInput(BaseModel):
    title: str
    priority: str  # low, medium, high
    due_date: datetime
    tags: List[str]
    
    @validator('priority')
    def validate_priority(cls, v):
        if v not in ['low', 'medium', 'high']:
            raise ValueError('Priority must be low, medium, or high')
        return v
    
    @validator('due_date')
    def validate_due_date(cls, v):
        if v < datetime.now():
            raise ValueError('Due date must be in the future')
        return v

# Валидный ввод
task = TaskInput(
    title="Внедрить Memory System",
    priority="high",
    due_date=datetime(2026, 3, 25),
    tags=["openclaw", "feature"]
)

# Невалидный ввод
try:
    invalid_task = TaskInput(
        title="Задача",
        priority="urgent",  # Ошибка!
        due_date=datetime.now(),
        tags=[]
    )
except ValidationError as e:
    print(f"Ошибка валидации: {e}")
```

## 🏗️ Архитектура

### Schema Definition:
```
┌─────────────────────────────────────────┐
│         Schema Registry                 │
│  - Реестр всех схем                     │
│  - Версионность                         │
│  - Документация                         │
└─────────────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│         Schema Validator                │
│  - Валидация ввода                      │
│  - Валидация вывода                     │
│  - Преобразование типов                 │
└─────────────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│         Response Generator              │
│  - Генерация по схеме                   │
│  - Заполнение полей                     │
│  - Форматирование                       │
└─────────────────────────────────────────┘
```

### Типы схем:
1. **Input Schemas** - валидация ввода пользователя
2. **Output Schemas** - структура ответов
3. **Data Schemas** - хранение данных
4. **API Schemas** - интеграции

## 🛠️ Библиотеки

### Python:
- **Pydantic** - валидация и схемы
- **Instructor** - structured LLM outputs
- **Marshmallow** - сериализация
- **Attrs** - data classes

### LLM Integration:
- **OpenAI Function Calling** - структурированные вызовы
- **Anthropic Tool Use** - tools с схемами
- **Guidance** - constrained generation

## 📊 Метрики эффективности

- Validation success rate
- Schema compliance rate
- Error reduction
- Integration reliability

## ✅ Преимущества

- Предсказуемые форматы
- Автоматическая валидация
- Type safety
- Лучшая интеграция
- Меньше ошибок парсинга

## ⚠️ Ограничения

- Требует определения схем
- Может ограничить гибкость
- Дополнительная сложность
- Нужно поддерживать актуальность

---

**Статус:** ✅ Одобрено 2026-03-24  
**Версия:** 1.0  
**Последнее обновление:** 2026-03-24
