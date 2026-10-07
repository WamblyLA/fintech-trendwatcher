# Fintech TrendWatcher

Сервис собирает финтех-публикации из RSS, CSV или демонстрационного набора, удаляет шум и дубликаты, выделяет важные сигналы и формирует итоговый отчёт для аналитика

Результат работы — ранжированный список событий с категорией, оценкой важности, объяснением и рекомендуемым действием. Доступны интерфейс на Streamlit и запуск из командной строки

## Архитектура

```text
RSS / CSV / demo
        |
Нормализация и фильтрация шума
        |
Определение категории
        |
Дедупликация: Fuzzy или TF-IDF
        |
Расчёт score и ранжирование
        |
Опциональное обогащение через LLM
        |
Отчёт и экспорт результатов
```

Ключевые файлы:

| Файл | Назначение |
| --- | --- |
| `app.py` | Streamlit-интерфейс и настройка pipeline |
| `trendwatcher/pipeline.py` | Нормализация, фильтрация, категоризация, дедупликация и ранжирование |
| `trendwatcher/rss.py` | Сбор публикаций из RSS |
| `trendwatcher/llm.py` | Обогащение сигналов через OpenRouter |
| `trendwatcher/demo_data.py` | Данные для демонстрационного запуска |
| `trendwatcher/score_weights.py` | Веса и расчёт итогового score |
| `scripts/run_pipeline.py` | Запуск pipeline из командной строки |
| `scripts/run_validation_experiment.py` | Воспроизводимый расчёт метрик |
| `prompts/` | Промпты для LLM |
| `data/` | Результаты pipeline и валидации |

## Запуск

Установить зависимости:

```bash
python -m pip install -r requirements.txt
```

Запустить интерфейс:

```bash
streamlit run app.py
```

Запустить pipeline из командной строки:

```bash
python scripts/run_pipeline.py
```

Для обработки собственного CSV:

```bash
python scripts/run_pipeline.py --input-csv path/to/articles.csv
```

CSV должен содержать поля `title`, `url`, `source`, `published_at`, `snippet`, `text`

Для LLM-обогащения укажите `OPENROUTER_API_KEY` и запустите:

```bash
python scripts/run_pipeline.py --use-llm
```

## Метрики

Валидация проведена на 60 RSS-публикациях с фиксированным `seed=42`

| Метрика | Значение |
| --- | ---: |
| Precision@10 | 1.000 |
| Recall@10 | 0.263 |
| NDCG@10 | 0.678 |
| Category Accuracy | 0.425 |

Повторный запуск валидации:

```bash
python scripts/run_validation_experiment.py
```

## Результаты

После запуска в `data/` создаются:

| Файл | Содержимое |
| --- | --- |
| `articles.csv` | Нормализованные публикации |
| `candidates.csv` | Публикации, прошедшие фильтрацию |
| `rejected.csv` | Отфильтрованные публикации и причины отклонения |
| `signals.json` | Итоговые карточки сигналов |
| `digest.md` | Финальный аналитический отчёт |
