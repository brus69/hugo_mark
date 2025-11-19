---
title: "Оперативное проектирование для анализа временных рядов с использованием больших языковых моделей | На пути к науке о данных"
date: "2025-11-19T22:53:35+0000"
draft: false
description: ""
h1: "Оперативное проектирование для анализа временных рядов с использованием больших языковых моделей"
urldel: "https://towardsdatascience.com/prompt-engineering-for-time-series-analysis-with-large-language-models/"
---

### Данные часто отличаются от результатов обычного анализа, в основном из-за проблем, связанных со временной зависимостью, с которыми в конечном итоге сталкивается каждый специалист по работе с данными.

Что, если бы вы могли ускорить и улучшить свой анализ с помощью правильной подсказки?

Большие языковые модели (LLMs) уже меняют правила игры для анализа временных рядов. Если вы объедините LLMs с умным проектированием подсказок, они могут открыть двери к методам, которые большинство аналитиков ещё не пробовали.

Они отлично справляются с выявлением закономерностей, обнаружением аномалий и прогнозированием.

В этом руководстве собраны проверенные стратегии, начиная с простой подготовки данных и заканчивая продвинутой проверкой моделей. К концу у вас будут практические инструменты, которые помогут вам сделать шаг вперёд.

Всё здесь основано на исследованиях и реальных примерах, так что вы получите практические инструменты, а не просто теорию!

#### 1. Основные стратегии проектирования подсказок для временных рядов

**1.1 Подсказки на основе патчей для прогнозирования**

**Framework Patch Instruct**
Хороший приём — разбить временной ряд на перекрывающиеся «патчи» и передать эти патчи в LLM с помощью структурированных подсказок. Этот подход, называемый PatchInstruct, очень эффективен и сохраняет точность примерно на том же уровне.

**Пример реализации:**
```
## System
You are a time-series forecasting expert in meteorology and sequential modeling.
Input: overlapping patches of size 3, reverse chronological (most recent first).

## User
Patches:
- Patch 1: [8.35, 8.36, 8.32]
- Patch 2: [8.45, 8.35, 8.25]
- Patch 3: [8.55, 8.45, 8.40]
...
- Patch N: [7.85, 7.95, 8.05]

## Task
1. Forecast next 3 values.
2. In ≤40 words, explain recent trend.

## Constraints
- Output: Markdown list, 2 decimals.
- Ensure predictions align with observed trend.

## Example
- Input: [5.0, 5.1, 5.2] → Output: [5.3, 5.4, 5.5].

## Evaluation Hook
Add: "Confidence: X/10. Assumptions: [...]".
```

**Почему это работает:**
* LLM заметит краткосрочные временные закономерности в данных.
* Используется меньше токенов, чем при загрузке необработанных данных (так меньше затрат).
* Сохраняется интерпретируемость, поскольку вы можете перестроить патчи позже.

**1.2 Нулевое проектирование подсказок с контекстными инструкциями**

Давайте представим, что вам нужен быстрый базовый прогноз.

Для этого подойдёт нулевое проектирование подсказок с контекстом. Вы просто даёте модели чёткое описание набора данных, частоты и временного горизонта прогноза, и она может идентифицировать закономерности без дополнительного обучения!

```
## System
You are a time-series analysis expert specializing in [domain].
Your task is to identify patterns, trends, and seasonality to forecast accurately.

## User
Analyze this time series: [x1, x2, ..., x96]
- Dataset: [Weather/Traffic/Sales/etc.]
- Frequency: [Daily/Hourly/etc.]
- Features: [List features]
- Horizon: [Number] periods ahead

## Task
1. Forecast [Number] periods ahead.
2. Note key seasonal or trend patterns.

## Constraints
- Output: Markdown list of predictions (2 decimals).
- Add ≤40-word explanation of drivers.

## Evaluation Hook
End with: "Confidence: X/10. Assumptions: [...]".
```

**1.3 Соседство с дополненной подсказкой**

Иногда одного временного ряда недостаточно. Мы можем добавить «соседние» ряды, которые похожи, и тогда LLM сможет выявить общие структуры и улучшить прогнозы:

```
## System
You are a time-series analyst with access to 5 similar historical series.
Use these neighbors to identify shared patterns and refine predictions.

## User
Target series: [current time series data]

Neighbors:
- Series 1: [ ... ]
- Series 2: [ ... ]
...

## Task
1. Predict the next [h] values of the target.
2. Explain in ≤40 words how neighbors influenced the forecast.

## Constraints
- Output: Markdown list of [h] predictions (2 decimals).
- Highlight any divergences from neighbors.

## Evaluation Hook
End with: "Confidence: X/10. Assumptions: [...]".
```

#### 2. Подсказки для предварительной обработки и анализа временных рядов

**2.1 Тестирование на стационарность и преобразование**

Одной из первых вещей, которые должны сделать специалисты по работе с данными, прежде чем моделировать временные ряды, является проверка того, является ли ряд стационарным.

Если это не так, им необходимо применить преобразования, такие как дифференцирование, логарифмирование или Бокс-Кокс.

**Подсказка для тестирования на стационарность и применения преобразований**

```
## System
You are a time-series analyst.

## User
Dataset: [N] observations
- Time period: [specify]
- Frequency: [specify]
- Suspected trend: [linear / non-linear / seasonal]
- Business context: [domain]

## Task
1. Explain how to test for stationarity using:
   - Augmented Dickey-Fuller
   - KPSS
   - Visual inspection
2. If non-stationary, suggest transformations: differencing, log, Box-Cox.
3. Provide Python code (statsmodels + pandas).

## Constraints
- Keep explanation ≤120 words.
- Code should be copy-paste ready.

## Evaluation Hook
End with: "Confidence: X/10. Assumptions: [...]".
```

**2.2 Автокорреляция и анализ лага**

**Автокорреляция** во временных рядах измеряет, насколько сильно текущие значения коррелируют с их собственными прошлыми значениями при разных лагах.

С помощью правильных графиков (ACF/PACF) вы можете наблюдать лаги, которые имеют наибольшее значение, и создавать на их основе функции.

**Подсказка для автокорреляции**

```
## System
You are a time-series expert.

## User
Dataset: [brief description]
- Length: [N] observations
- Frequency: [daily/hourly/etc.]
- Raw sample: [first 20–30 values]

## Task
1. Provide Python code to generate ACF & PACF plots.
2. Explain how to interpret:
   - AR lags
   - MA components
   - Seasonal patterns
3. Recommend lag features based on significant lags.
4. Show Python code to engineer these lags (handle missing values).

## Constraints
- Output: ≤150 words explanation + Python snippets.
- Use statsmodels + pandas.

## Evaluation Hook
End with: "Confidence: X/10. Key lags flagged: [list]".
```

**2.3 Сезонное разложение и анализ тренда**

Разложение помогает увидеть историю, стоящую за данными, и увидеть её в разных слоях: тренд, сезонность и остатки.

**Подсказка для сезонного разложения**

```
## System
You are a time-series expert.

## User
Data: [time series]
- Suspected seasonality: [daily/weekly/yearly]
- Business context: [domain]

## Task
1. Apply STL decomposition.
2. Compute:
   - Seasonal strength Qs = 1 - Var(Residual)/Var(Seasonal+Residual)
   - Trend strength Qt = 1 - Var(Residual)/Var(Trend+Residual)
3. Interpret trend & seasonality for business insights.
4. Recommend modeling approaches.
5. Provide Python code for visualization.

## Constraints
- Keep explanation ≤150 words.
- Code should use statsmodels + matplotlib.

## Evaluation Hook
End with: "Confidence: X/10. Key business implications: [...]".
```

#### 3. Обнаружение аномалий с помощью LLMs

**3.1 Прямое проектирование подсказок для обнаружения аномалий**

Обнаружение аномалий во временных рядах обычно занимает много времени и требует больших усилий.

LLMs могут действовать как бдительный аналитик, выявляя выходящие за рамки значения в ваших данных.

**Подсказка для обнаружения аномалий**

```
## System
You are a senior data scientist specializing in time-series anomaly detection.

## User
Context:
- Domain: [Financial/IoT/Healthcare/etc.]
- Normal operating range: [specify if known]
- Time period: [specify]
- Sampling frequency: [specify]
- Data: [time series values]

## Task
1. Detect anomalies with timestamps/indices.
2. Classify as:
   - Point anomalies
   - Contextual anomalies
   - Collective anomalies
3. Assign confidence scores (1–10).
4. Explain reasoning for each detection.
5. Suggest potential causes (domain-specific).

## Constraints
- Output: Markdown table (columns: Index, Type, Confidence, Explanation, Possible Cause).
- Keep narrative ≤150 words.

## Evaluation Hook
End with: "Overall confidence: X/10. Further data needed: [...]".
```

**3.2 Прогнозирование на основе обнаружения аномалий**

Вместо того чтобы искать аномалии напрямую, другая умная стратегия состоит в том, чтобы сначала спрогнозировать, что «должно» произойти, а затем измерить, где реальность отклоняется от этих ожиданий.

Эти отклонения могут выявить аномалии, которые иначе не выделялись бы.

**Подсказка для прогнозирования на основе обнаружения аномалий**

```
## System
You are an expert in forecasting-based anomaly detection.

## User
- Historical data: [time series]
- Forecast horizon: [N periods]

## Method
1. Forecast the next [N] periods.
2. Compare actual vs forecasted values.
3. Compute residuals (errors).
4. Flag anomalies where |actual - forecast| > threshold.
5. Use z-score & IQR methods to set thresholds.

## Task
Provide:
- Forecasted values
- 95% prediction intervals
- Anomaly flags with severity levels
- Recommended threshold values

## Constraints
- Output: Markdown table (columns: Period, Forecast, Interval, Actual, Residual, Anomaly Flag, Severity).
- Keep explanation ≤120 words.

## Evaluation Hook
End with: "Confidence: X/10. Threshold method used: [z-score/IQR]".
```

#### 4. Разработка функций для зависящих от времени данных

Умные функции могут сделать вашу модель или сломать её.

Существует слишком много вариантов: от лагов до скользящих окон, циклических функций и внешних переменных. Есть много чего можно добавить, чтобы уловить временные зависимости.

**4.1 Автоматизированное создание функций**

Настоящее волшебство происходит, когда вы создаёте значимые функции, которые улавливают тенденции, сезонность и временную динамику. LLMs могут помочь автоматизировать этот процесс, генерируя широкий спектр полезных функций.

**Комплексная подсказка по разработке функций:**

```
## System
You are a feature engineering expert for time series.

## User
Dataset: [description]
- Target variable: [specify]
- Temporal granularity: [hourly/daily/etc.]
- Business domain: [context]

## Task
Create temporal features across 5 categories:
1. **Lag Features**
   - Simple lags, seasonal lags, cross-variable lags
2. **Rolling Window Features**
   - Moving averages, std/min/max, quantiles
3. **Time-based Features**
   - Hour, day, month, quarter, year, DOW, WOY, is_weekend, is_holiday, time since events
4. **Seasonal & Cyclical Features**
   - Fourier terms, sine/cosine transforms, interactions
5. **Change-based Features**
   - Differences, pct changes, volatility measures

## Constraints
- Output: Python code using pandas/numpy.
- Add short guidance on feature selection (importance/collinearity).

## Evaluation Hook
End with: "Confidence: X/10. Features most impactful for [domain]: [...]".
```

**4.2 Интеграция внешних переменных**

Может случиться так, что целевого ряда недостаточно, чтобы объяснить всю историю.

Существуют внешние факторы, которые часто влияют на наши данные, например погода, экономические показатели или специальные события. Они могут добавить контекст и улучшить прогнозы.

Хитрость заключается в том, чтобы знать, как правильно интегрировать их, не нарушая временных правил. Вот подсказка для включения экзогенных переменных в ваш анализ.

**Подсказка для экзогенных переменных:**

```
## System
You are a time-series modeling expert.
Task: Integrate external variables (exogenous features) into a forecasting pipeline.

## User
Primary series: [target variable]
External variables: [list]
Data availability: [past only / future known / mixed]

## Task
1. Assess variable relevance (correlation, cross-correlation).
2. Align frequencies and handle resampling.
3. Create interaction features between external & target.
4. Apply time-aware cross-validation.
5. Select features suited for time-series models.
6. Handle missing values in external variables.

## Constraints
- Output: Python code for
  - Data alignment & resampling
  - Cross-correlation analysis
  - Feature engineering with external vars
  - Model integration:
    - ARIMA (with exogenous vars)
    - Prophet (with regressors)
    - ML models (with external features)

## Evaluation Hook
End with: "Confidence: X/10. Most impactful external variables: [...]".
```

