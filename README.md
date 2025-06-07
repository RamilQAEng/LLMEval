# 🌐 Полный гайд по оценке LLM‑систем

> **Цель документа** – стать "настольной книгой" для команды (и для Enterprise‑клиентов), описывающей *все* ключевые направления, метрики, фреймворки и практики автоматизированной оценки больших языковых моделей (LLM), RAG‑систем и многоагентных пайплайнов. Документ спроектирован как живой курс: каждую секцию можно раскрывать глубже или экспортировать в презентации / воркшопы.

---

## 1. Почему оценка стала критически важной

### 1.1. Эволюция требований

* **2020–2022** – Proof‑of‑Concept: достаточно «работает / не работает».
* **2023–2024** – Продуктовый MVP: появились запросы на SLA по качеству и безопасности.
* **2025+** – **Enterprise‑Grade**: регуляторы (EU AI Act, ISO/IEC 42001), юридические риски, brand‑safety, cost‑control. Оценка = обязательный этап CI/CD.

### 1.2. 5 бизнес‑драйверов оценки

1. **Trust** (доверие клиентов)
2. **Compliance** (индустрия, регулятор)
3. **Cost Efficiency** (качество ‑ токены)
4. **Velocity** (быстрый релиз через авто‑гейты)
5. **Differentiation** (продажа «надёжного» ИИ как конкурентного преимущества)

---

## 2. Таксономия направлений оценки

| № | Направление                               | Что измеряем                                                      | Тип рисков                    | Ключевые метрики                               |
| - | ----------------------------------------- | ----------------------------------------------------------------- | ----------------------------- | ---------------------------------------------- |
| 1 | **Quality / Utility**Качество, полезность | Корректность, релевантность, полнота                              | Некачественные ответы → churn | Faithfulness, Relevancy, Helpfulness, Fluency  |
| 2 | **Safety & Red Teaming**                  | Токсичность, jailbreak, ненадёжные советы                         | Reputational, Legal           | Harmlessness, Toxicity, Jailbreak Success Rate |
| 3 | **Robustness**                            | Устойчивость к noised / adversarial input                         | Product Failure               | Robust Accuracy, Error Consistency             |
| 4 | **Privacy / PII**                         | Лик утечек личных данных, memorization                            | GDPR/CCPA штрафы              | PII Recall, Leakage Precision                  |
| 5 | **Bias & Fairness**                       | Демографический bias, дискриминация                               | DEI, PR кризис                | Bias Score, Equalized Odds                     |
| 6 | **Controllability**                       | Способность строго следовать инструкциям (style, length, persona) | Brand‑voice drift             | Obedience Rate, Style Accuracy                 |
| 7 | **Cost‑Effectiveness**                    | Качество / токен, Latency                                         | \$\$\$                        | QPS/\$, Quality‑per‑Token                      |

> **Замечание:** в современной практике 1–5 считаются «минимумом для продакшна», 6–7 ‑ ключевой дифференциатор.

---

## 3. Каталог метрик (EN / RU)

### 3.1 Метрики качества (Quality / Utility)

| EN                       | RU                 | Что меряет                           | Как вычислить                             |
| ------------------------ | ------------------ | ------------------------------------ | ----------------------------------------- |
| **Faithfulness**         | Достоверность      | Ответ основан на источнике (RAG)     | LLM‑Judge / n‑gram overlap контекст‑ответ |
| **Context Coverage**     | Покрытие контекста | Сколько ключевых фактов использовано | RAGAS `context_recall`                    |
| **Answer Relevancy**     | Релевантность      | Ответ «в тему» вопроса               | Cosine(Q, A) + LLM‑Judge                  |
| **Helpfulness**          | Полезность         | Насколько ответ решает задачу        | Human / LLM Likert 1‑5                    |
| **Fluency / Coherence**  | Связность          | Грамматика, логика                   | LLM‑Judge или LanguageTool API            |
| **Creativity**           | Креативность       | Новизна формулировок                 | LLMeBench `diversity_score`               |
| **Conciseness / Length** | Краткость          | Относительный объём                  | len(tokens) vs target                     |
| **Correctness**          | Корректность       | Фактическая точность                 | External knowledge check                  |

### 3.2 Метрики безопасности

| EN                         | RU                 | Метод                             |
| -------------------------- | ------------------ | --------------------------------- |
| **Toxicity**               | Токсичность        | Perspective API / Detoxify        |
| **Jailbreak Success Rate** | Доля успешных атак | PromptInject list + binary result |
| **Harmlessness**           | Безвредность       | Anthropic Safety Scale (0‑5)      |
| **Self‑Harm Score**        | Контент о суициде  | OASIS classifier                  |

### 3.3 Метрики робастности

| EN                    | RU                 | Пример теста                           |
| --------------------- | ------------------ | -------------------------------------- |
| **Robust Accuracy**   | Устойчив. точность | Synonym noise / Typos                  |
| **Error Consistency** | Соглас. ошибок     | Повторяемость output при perturbations |

### 3.4 Метрики приватности

| EN                    | RU                | Что делаем                                |
| --------------------- | ----------------- | ----------------------------------------- |
| **PII Recall**        | Полнота утечки    | К‑во выявленных PII / вставленных         |
| **Memorization Rate** | Коэф. запоминания | Known‑secret prompts → exact string match |

### 3.5 Метрики bias / fairness

| EN                 | RU           | Метод                    |
| ------------------ | ------------ | ------------------------ |
| **Bias Score**     | Оценка bias  | StereoSet / CrowS‑Pairs  |
| **Equalized Odds** | Равные шансы | Classification disparity |

### 3.6 Cost‑метрики

| EN                         | RU                         | Формула                       |
| -------------------------- | -------------------------- | ----------------------------- |
| **Cost per Quality Point** | Стоимость за балл качества | (Tokens \* \$) / QualityScore |
| **Latency (p95)**          | Задержка p95               | ms                            |

---

## 4. Обзор актуальных фреймворков (2025)

### 4.1 Quality Evaluation

| Фреймворк              | Язык          | Кратко                        | Особенности                     |
| ---------------------- | ------------- | ----------------------------- | ------------------------------- |
| **RAGAS**              | Python        | RAG‑метрики                   | 🤝 LangChain, API agnostic      |
| **TruLens**            | Python        | Generic LLM eval + monitoring | Real‑time traces, OpenTelemetry |
| **DeepEval**           | Python        | Unit‑style тесты              | PyTest‑like DSL                 |
| **LLM‑Evals (OpenAI)** | YAML + Python | Батарея задач                 | Шаблоны evals, CI hooks         |
| **MT‑Bench**           | Python        | Pairwise LLM ranking          | Gold prompts (LMSYS)            |

### 4.2 Safety & Red Teaming

| Framework                        | Scope              | Highlights (RU пояснение)                     |
| -------------------------------- | ------------------ | --------------------------------------------- |
| **PromptInject**                 | Jailbreak          | 1000+ adversarial prompts (атаки на jailbreak, обход фильтров) |
| **Guardrails AI**                | Content policy     | Declarative JSON spec + filters (декларативные правила и фильтры контента) |
| **SecEval**                      | OSS security tests | XSS, jailbreak, SQL‑injection prompts (тесты на уязвимости: XSS, jailbreak, SQL-инъекции) |
| **Anthropic Red Teamer Toolkit** | Proprietary        | Scenario libraries, auto‑grading (проприетарные сценарии red-teaming и автооценка) |

### 4.3 Robustness Testing

| Framework                | Method                              | RU пояснение                         |
| ------------------------ | ----------------------------------- | ------------------------------------ |
| **TextAttack**           | Perturbation attacks, adv. training | Атаки с искажениями текста, adversarial обучение |
| **RobustBench (text)**   | Benchmark leaderboards              | Бенчмарки устойчивости моделей к атакам |
| **DeepMind Robust Eval** | Private, state‑of‑the‑art           | Закрытый фреймворк SOTA-уровня по робастности |

### 4.4 Privacy / PII

| Framework                   | Особенности                 | RU пояснение                               |
| --------------------------- | --------------------------- | ------------------------------------------ |
| **PII‑Bench (ETH)**         | Synthetic leaks ↔ detection | Проверка на утечки синтетических персональных данных |
| **Google LLM Privacy Eval** | K‑anonymous probing         | K-анонимные тесты на утечки приватной информации |

### 4.5 Bias / Fairness

| Framework     | Факты                    | RU пояснение                              |
| ------------- | ------------------------ | ----------------------------------------- |
| **FairBench** | Demographic parity tests | Тесты на демографическую справедливость (равенство групп) |
| **BiasEval**  | StereoSet integration    | Проверка стереотипов с помощью StereoSet (гендер, этнос и т.п.) |

---

## 5. Наш собственный фреймворк (Python 3.13‑ready)

### 5.1 Архитектура

- **Core**: Pydantic 2.11 schemas → единый формат оценки.
- **Adapters**: OpenAI, Anthropic, Mistral via OpenRouter.
- **Metrics**: вбитые модули Detoxify, custom cost‑calculator, LLM-based scoring.
- **Runner**: Async‑powered, совместим с `pytest -q`.
- **Reporting**: Markdown + JSON + HTML (Streamlit dashboard v2).

### 5.2 Особенности

1. Поддержка **Python 3.13** (PEP 693 literal types + improved perf).
2. Плагинная модель: легко добавить новую метрику (см. `metrics/base.py`).
3. **Гибкая работа с input-данными**:
    - Не требуется строгое поле ground truth (как в RAGAS).
    - Можно прогонять pipeline на context + answer → автоматически проставляется `score=0` для метрик, требующих reference.
    - Позволяет обрабатывать неполные QA-таблицы, ответы чат-ботов, агентов.
4. Поддержка batch evaluation и interactive mode (CLI).
5. Авто-гейты для GitHub Actions (`exit 1` при нарушении thresholds).
6. Хранит историю run-ов в SQLite → можно строить регресс-графики.

> В отличие от RAGAS, наш фреймворк более гибкий по обработке ground truth и позволяет встраиваться в любую RAG / Generative / Agent-цепочку даже при отсутствии полноценного reference датасета.

---

## 6. CI/CD Integration Blueprint

```
name: llm-pipeline
on: [push]

jobs:
  quality:
    steps:
      - run: pytest tests/quality
  safety:
    steps:
      - run: pytest tests/safety
  robustness:
    steps:
      - run: python robustness/run.py --threshold 0.8
  pii:
    steps:
      - run: python pii_scan.py datasets/
  deploy:
    needs: [quality, safety, robustness, pii]
    if: success()
    steps:
      - run: helm upgrade ...
```

*Пороговые значения* хранятся в `eval_config.yml` и версионируются.

---

## 7. Enterprise‑угол: что важно клиентам

1. **Explainability**: возможность показать, *почему* ответ принят / отклонён.
2. **Audit Trail**: неизменяемые логи оценки (WORM storage).
3. **On‑Prem Deployment**: air‑gapped кластер + offline модели‑судьи.
4. **Custom Policy**: индустрия (финансы, медицина) требует собственные safety rules.
5. **TCO Visibility**: прогноз стоимости при scale‑out.

---

## 8. Тренды 2025–2026

* **Multi‑Agent Judging**: ансамбль моделей (GPT‑4o + Claude‑Sonnet).
* **Retrieval‑Grounded Checking**: авто‑поиск фактов в внешних базах для fact‑check.
* **Probabilistic Guardrails**: Bayesian kill‑switch при высокой неопределённости.
* **LLM‑in‑Database**: Postgres FDW с live‑eval UDF (pgai\_eval()).

---

## 9. Интеграция в цепочки агентов / чат‑ботов

1. **Pre‑Prompt Sanitizer** → удаляет PII.
2. **Generation** → основная модель.
3. **Judge Layer** → LLM‑as‑a‑Judge + cost monitor.
4. **Post‑Processor** → style / tone enforcement.
5. **Logging & Metrics** → отправка в Prometheus + Grafana.

*Все шаги «конвейера» должны быть idempotent и трейсируемы.*

---

## 10. Summary & Homework

* **Оценка ≠ один скрипт** – это полноценная подсистема.
* Без safety‑гейтов нельзя продавать Enterprise.
* Наш Python 3.13‑фреймворк уже покрывает 70% потребностей, дополняем его Robustness + Bias.

---

### === CONTINUE FROM HERE ===

*Следующая часть* – углублённые примеры кода, сравнительные бенчмарки стоимости и чеклист для аудит‑отчёта (GxP, SOC2, EU AI Act).
