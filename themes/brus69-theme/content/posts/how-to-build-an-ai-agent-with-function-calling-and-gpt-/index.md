---
title: "Как создать ИИ-агент с вызовом функций и GPT-5 | На пути к науке о данных"
date: "2025-11-19T23:13:09+0000"
draft: false
description: ""
h1: "Как создать ИИ-агент с вызовом функций и GPT-5"
urldel: "https://towardsdatascience.com/how-to-build-an-ai-agent-with-function-calling-and-gpt-5/"
---

## Большие языковые модели (LLMs)

**Большие языковые модели (LLMs)** — это продвинутые системы искусственного интеллекта, построенные на основе глубоких нейронных сетей, таких как трансформеры, и обученные на огромных объёмах текста для генерации языка, похожего на человеческий. Примеры LLM включают ChatGPT, Claude, Gemini и Grok. Они могут решать множество сложных задач и используются в различных областях, таких как наука, здравоохранение, образование и финансы.

### Агент ИИ расширяет возможности LLM

Агент ИИ расширяет возможности LLM для решения задач, выходящих за рамки их предварительно обученных знаний. LLM может написать руководство по Python на основе того, что она изучила во время обучения. Если вы попросите её забронировать рейс, потребуется доступ к вашему календарю, веб-поиск и возможность выполнять действия, которые выходят за рамки предварительно обученных знаний LLM.

### Примеры действий

* **Прогноз погоды:** LLM подключается к инструменту веб-поиска, чтобы получить последний прогноз погоды.
* **Агент по бронированию:** агент ИИ может проверить календарь пользователя, выполнить веб-поиск для посещения сайта бронирования, такого как Expedia, чтобы найти доступные варианты перелёта и отелей, представить их пользователю для подтверждения и завершить бронирование от имени пользователя.

### Как работает агент ИИ

Агенты ИИ формируют систему, которая использует большую языковую модель для планирования, рассуждений и выполнения действий для взаимодействия со своей средой с помощью инструментов, предложенных моделью для решения конкретной задачи.

### Базовая структура агента ИИ

* **Большая языковая модель (LLM):** LLM — это «мозг» агента ИИ. Она принимает запрос пользователя, планирует и рассуждает над запросом, разбивает проблему на шаги, которые определяют, какие инструменты следует использовать для выполнения задачи.
* **Инструмент:** это структура, которую агент использует для выполнения действия на основе плана и рассуждений, полученных от большой языковой модели. Например, если вы попросите LLM забронировать столик в ресторане, возможными инструментами будут календарь для проверки вашей доступности и инструмент веб-поиска для доступа к веб-сайту ресторана и бронирования столика.

### Иллюстративное принятие решений агентом по бронированию

Агенты ИИ могут получать доступ к различным инструментам в зависимости от задачи. Инструментом может быть хранилище данных, такое как база данных. Например, агент поддержки клиентов может получить доступ к учётным данным клиента и истории покупок и решить, когда извлечь эту информацию, чтобы помочь решить проблему.

Агенты ИИ используются для решения широкого спектра задач, и существует множество мощных агентов. Кодовые агенты, особенно агентские IDE, такие как Cursor, Windsurf и GitHub Copilot, помогают инженерам писать и отлаживать код быстрее и создавать проекты быстрее. Агенты CLI, такие как Claude Code и Codex CLI, могут взаимодействовать с рабочим столом пользователя и терминалом для выполнения задач кодирования. ChatGPT поддерживает агентов, которые могут выполнять действия, такие как бронирование мест от имени пользователя. Агенты также интегрированы в рабочие процессы поддержки клиентов для общения с клиентами и решения их проблем.

### Вызов функций

Вызов функций — это метод подключения большой языковой модели (LLM) к внешним инструментам, таким как API или базы данных. Он используется при создании агентов ИИ для подключения LLM к инструментам. В вызове функций каждый инструмент определяется как кодовая функция (например, API погоды для получения последнего прогноза) вместе с JSON-схемой, которая определяет параметры функции и инструктирует LLM, когда и как вызывать функцию для выполнения задачи.

### Базовая структура агента веб-поиска

Основная логика агента веб-поиска:

* Определить функцию для обработки веб-поиска.
* Определить пользовательские инструкции, которые направляют большую языковую модель в определении, когда вызывать функцию веб-поиска на основе запроса. Например, если запрос касается текущей погоды, агент веб-поиска распознает необходимость поиска в интернете для получения последних отчётов о погоде. Однако если запрос просит написать руководство по языку программирования, например, по Python, что-то, что LLM может ответить на основе предварительно обученных знаний, она не будет вызывать функцию веб-поиска, а ответит напрямую.

### Предварительные требования

**Создайте учётную запись OpenAI и сгенерируйте ключ API**

1. Создайте учётную запись OpenAI, если у вас её нет.
2. Сгенерируйте ключ API.

**Настройка и активация среды**

```
python3 -m venv env
source env/bin/activate
```

**Экспорт ключа API OpenAI**

```
export OPENAI_API_KEY="Your Openai API Key"
```

**Настройка Tavily для веб-поиска**

Tavily — это специализированный инструмент веб-поиска для агентов ИИ. Создайте учётную запись на Tavily.com, и после настройки вашего профиля будет сгенерирован API-ключ, который вы можете скопировать в свою среду. Новым учётным записям предоставляется 1000 бесплатных кредитов, которые можно использовать для до 1000 веб-поисков.

**Экспорт ключа API Tavily**

```
export TAVILY_API_KEY="Your Tavily API Key"
```

**Установка пакетов**

```
pip3 install openai
pip3 install tavily-python
```

### Создание агента веб-поиска с вызовом функций шаг за шагом

**Шаг 1: создание функции веб-поиска с помощью Tavily**

Функция веб-поиска реализована с использованием Tavily, которая служит инструментом для вызова функций в агенте веб-поиска.

```
from tavily import TavilyClient
import os

tavily = TavilyClient(api_key=os.getenv("TAVILY_API_KEY"))

def web_search(query: str, num_results: int = 10):
    try:
        result = tavily.search(
            query=query,
            search_depth="basic",
            max_results=num_results,
            include_answer=False,
            include_raw_content=False,
            include_images=False
        )

        results = result.get("results", [])

        return {
            "query": query,
            "results": results,
            "sources": [
                {"title": r.get("title", ""), "url": r.get("url", "")}
                for r in results
            ]
        }

    except Exception as e:
        return {
            "error": f"Search error: {e}",
            "query": query,
            "results": [],
            "sources": [],
        }
```

**Шаг 2: создание схемы инструмента**

Схема инструмента определяет пользовательские инструкции для ИИ-модели о том, когда она должна вызывать инструмент, в данном случае инструмент, который будет использоваться в функции веб-поиска. Она также определяет условия и действия, которые должны быть предприняты, когда модель вызывает инструмент.

```
tool_schema = [
    {
        "type": "function",
        "name": "web_search",
        "description": """Execute a web search to fetch up to date information. Synthesize a concise, 
        self-contained answer from the content of the results of the visited pages.
        Fetch pages, extract text, and provide the best available result while citing 1-3 sources (title + URL). 
        If sources conflict, surface the uncertainty and prefer the most recent evidence.
        """,
        "strict": True,
        "parameters": {
            "type": "object",
            "properties": {
                "query": {
                    "type": "string",
                    "description": "Query to be searched on the web.",
                },
            },
            "required": ["query"],
            "additionalProperties": False
        },
    },
]
```

**Шаг 3: создание агента веб-поиска с помощью GPT-5 и вызова функций**

Наконец, я создам агента, с которым можно будет общаться, который сможет искать в интернете, когда ему понадобится актуальная информация. Я буду использовать GPT-5-mini, быструю и точную модель от OpenAI, вместе с вызовом функций для вызова схемы инструмента и уже определённой функции веб-поиска.

```
from datetime import datetime, timezone
import json
from openai import OpenAI
import os
from tavily import TavilyClient

tavily = TavilyClient(api_key=os.getenv("TAVILY_API_KEY"))

def web_search(query: str, num_results: int = 10):
    try:
        result = tavily.search(
            query=query,
            search_depth="basic",
            max_results=num_results,
            include_answer=False,
            include_raw_content=False,
            include_images=False
        )

        results = result.get("results", [])

        return {
            "query": query,
            "results": results,
            "sources": [
                {"title": r.get("title", ""), "url": r.get("url", "")}
                for r in results
            ]
        }

    except Exception as e:
        return {
            "error": f"Search error: {e}",
            "query": query,
            "results": [],
            "sources": [],
        }

tool_schema = [
    {
        "type": "function",
        "name": "web_search",
        "description": """Execute a web search to fetch up to date information. Synthesize a concise, 
        self-contained answer from the content of the results of the visited pages.
        Fetch pages, extract text, and provide the best available result while citing 1-3 sources (title + URL). 
        If sources conflict, surface the uncertainty and prefer the most recent evidence.
        """,
        "strict": True,
        "parameters": {
            "type": "object",
            "properties": {
                "query": {
                    "type": "string",
                    "description": "Query to be searched on the web.",
                },
            },
            "required": ["query"],
            "additionalProperties": False
        },
    },
]

client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

prev_response_id = None

tool_results = []

while True:

    if len(tool_results) == 0:
        user_message = input("User: ")

        """ commands for exiting chat """
        if isinstance(user_message, str) and user_message.strip().lower() in {"exit", "q"}:
            print("Exiting chat. Goodbye!")
            break

    else:
        user_message = tool_results.copy()

        tool_results = []

    today_date = datetime.now(timezone.utc).date().isoformat()

    response = client.responses.create(
        model = "gpt-5-mini",
        input = user_message,
        instructions=f"Current date is {today_date}.",
        tools = tool_schema,
        previous_response_id=prev_response_id,
        text = {"verbosity": "low"},
        reasoning={
            "effort": "low",
        },
        store=True,
        )

    prev_response_id = response.id

    for output in response.output:
        if output.type == "reasoning":
            print("Assistant: ","Reasoning ....")

            for reasoning_summary in output.summary:
                print("Assistant: ",reasoning_summary)

        elif output.type == "message":
            for item in output.content:
                print("Assistant: ",item.text)

        elif output.type == "function_call":
            function_name = globals().get(output.name)

            args = json.loads(output.arguments)
            function_response = function_name(**args)

            tool_results.append(
                {
                    "type": "function_call_output",
                    "call_id": output.call_id,
                    "output": json.dumps(function_response)
                }
            )

Когда вы запустите код, вы сможете легко общаться с агентом, чтобы задавать вопросы, требующие актуальной информации, например, о текущей погоде или последних выпусках продуктов. Агент отвечает актуальной информацией вместе с соответствующими источниками из интернета.

### Заключение

В этом посте я объяснил, как работает агент ИИ и как он расширяет возможности большой языковой модели для взаимодействия со своей средой, выполнения действий и решения задач с помощью инструментов. Я также объяснил вызов функций и то, как он позволяет LLM вызывать инструменты. Я продемонстрировал, как создать схему инструмента для вызова функций, которая определяет, когда и как LLM должна вызывать инструмент для выполнения действия. Я определил функцию веб-поиска с помощью Tavily для получения информации из интернета и затем показал шаг за шагом, как создать агента веб-поиска с помощью вызова функций и GPT-5-mini в качестве LLM. В итоге мы создали агента веб-поиска, способного получать актуальную информацию из интернета для ответа на запросы пользователей.

