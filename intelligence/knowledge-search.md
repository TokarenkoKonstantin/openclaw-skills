# 🔍 Knowledge Search - RAG Capability

## Описание навыка

OpenClaw может искать информацию в документах, базах знаний и внешних источниках, используя семантический поиск и RAG (Retrieval Augmented Generation).

## 🎯 Возможности

### 1. Document Search
- Загрузка и индексирование документов
- Семантический поиск по содержанию
- Извлечение релевантных фрагментов
- Поддержка multiple форматов (PDF, MD, TXT, DOCX)

### 2. Knowledge Base
- Структурированная база знаний
- Категоризация и тегирование
- Связи между документами
- Версионность и история

### 3. External Sources
- Поиск в интернете (при наличии API)
- Интеграция с внешними API
- Парсинг веб-страниц
- RSS и новости

### 4. Contextual Answers
- Ответы на основе документов
- Цитирование источников
- Уверенность в ответе (confidence score)
- Отказ при недостатке информации

## 💬 Примеры использования

### Пример 1: Поиск по документации
```
Пользователь: Как настроить cron для Kaizen цикла?

OpenClaw (ищет в документации):
[Найдено в ~/kiro-kaizen/CRON-SETUP.md]

Для настройки cron:
1. Открой crontab: crontab -e
2. Добавь: 0 */12 * * * /root/kiro-kaizen/trigger-cycle.sh
3. Сохрани и выйди

Источник: CRON-SETUP.md, раздел "Настройка Cron"
```

### Пример 2: Вопрос по коду
```
Пользователь: Как работает функция update_memory()?

OpenClaw (ищет в коде):
[Найдено в ~/kiro-kaizen/update-memory.sh]

Функция update_memory():
1. Читает learnings.md, mistakes.md, corrections.md
2. Генерирует ~/MEMORY.md
3. Копирует файлы в ~/memory/
4. Логирует обновление

Строки 15-45: основная логика
```

### Пример 3: Исследование темы
```
Пользователь: Что мы знаем о RAG системах?

OpenClaw (ищет в knowledge base):
[Найдено в нескольких источниках]

Из ~/projects/ai-portfolio/AI-TOOLS-COLLECTION.md:
- LlamaIndex - лучший фреймворк для RAG
- ChromaDB - vector database для embeddings
- Haystack - production-ready NLP framework

Из ~/projects/openclaw-skills/ai-ml/rag-systems.md:
- RAG = Retrieval + Generation
- Vector search + LLM
- Контекст из документов → ответ

Резюме: RAG позволяет LLM отвечать на основе документов.
```

## 🏗️ Архитектура

### RAG Pipeline:
```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Document   │────▶│   Chunking  │────▶│  Embedding  │
│   Loader    │     │   & Split   │     │   Model     │
└─────────────┘     └─────────────┘     └─────────────┘
                                               │
                                               ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│    LLM      │◀────│   Context   │◀────│   Vector    │
│  Response   │     │   Builder   │     │   Search    │
└─────────────┘     └─────────────┘     └─────────────┘
```

### Компоненты:
1. **Document Loader** - загрузка файлов
2. **Text Splitter** - разбивка на чанки
3. **Embedding Model** - векторизация текста
4. **Vector Store** - хранение embeddings
5. **Retriever** - поиск релевантных чанков
6. **Context Builder** - сборка контекста
7. **LLM** - генерация ответа

## 🛠️ Технологии

### Vector Databases:
- **ChromaDB** - простая, локальная
- **Pinecone** - облачная, масштабируемая
- **Weaviate** - с графом знаний
- **Qdrant** - быстрая, с фильтрами

### Embedding Models:
- **OpenAI** - text-embedding-ada-002
- **Sentence Transformers** - локальные
- **Cohere** - multilingual

### Frameworks:
- **LlamaIndex** - специализация на RAG
- **LangChain** - универсальный
- **Haystack** - production-ready

## 📊 Метрики эффективности

- Precision@K (точность поиска)
- Recall (полнота извлечения)
- Answer relevance (релевантность ответа)
- Source accuracy (точность цитирования)

## ⚠️ Ограничения

- Качество зависит от документов
- Может быть медленным на больших объёмах
- Требует настройки chunking strategy
- Контекст ограничен размером окна LLM

---

**Статус:** ✅ Одобрено 2026-03-24  
**Версия:** 1.0  
**Последнее обновление:** 2026-03-24
