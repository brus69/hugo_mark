---
title: "Создание системы преобразования текста в SQL: повторение подхода Pinterest"
date: "2025-11-19T23:13:09+0000"
draft: true
description: "Научитесь создавать систему преобразования текста в SQL. В нашем руководстве показано, как повторить подход Pinterest к преобразованию естественного языка в SQL."
h1: "Создание системы преобразования текста в SQL: Руководство по копированию подхода Pinterest"
urldel: "https://www.analyticsvidhya.com/blog/2025/10/build-a-text-to-sql-system/"
---

Данные имеют важное значение для принятия решений в современном бизнесе. Многие сотрудники, однако, не знакомы с SQL. Это создаёт препятствие между вопросами и ответами.

Система Text-to-SQL решает эту проблему напрямую. Она переводит простые вопросы в запросы к базе данных. В этой статье показано, как создать генератор SQL. Мы будем следовать идеям инженерной команды Pinterest по созданию системы Text-to-SQL. Вы узнаете, как преобразовывать естественный язык в SQL. Мы также будем использовать передовые методы, такие как RAG для выбора таблиц.

### Понимание подхода Pinterest

Pinterest хотел сделать данные доступными для всех. Их сотрудникам нужны были идеи из обширных наборов данных. Большинство из них не были экспертами SQL. Эта задача привела к созданию платформы Pinterest Text-to-SQL. Их путь представляет собой отличную дорожную карту для создания подобных инструментов.

#### Первая версия

Их первая система была простой. Пользователь задавал вопрос и также указывал таблицы базы данных, которые, по его мнению, были релевантны. Система генерировала SQL-запрос.

Давайте рассмотрим архитектуру более подробно:

1. Пользователь задаёт аналитический вопрос, выбирая таблицы для использования.
2. Из хранилища метаданных таблиц извлекаются соответствующие схемы таблиц.
3. Вопрос, выбранный диалект SQL и схемы таблиц объединяются в запрос Text-to-SQL.
4. Запрос передаётся в LLM.
5. Потоковый ответ генерируется и отображается пользователю.

Этот подход работал, но у него был существенный недостаток. Пользователи часто не имели представления, в каких таблицах содержатся ответы на их вопросы.

#### Вторая версия

Чтобы решить эту проблему, их команда создала более интеллектуальную систему. Она использовала технику под названием Retrieval-Augmented Generation (RAG). Вместо того чтобы просить пользователя указать таблицы, система находила их автоматически. Она искала в коллекции описаний таблиц, чтобы найти наиболее релевантные для вопроса. Использование RAG для выбора таблиц сделало инструмент более удобным для пользователя.

1. Для создания векторного индекса сводок таблиц и исторических запросов к ним используется автономная работа.
2. Предположим, пользователь не указал никаких таблиц. В этом случае его вопрос преобразуется в эмбеддинги, и на основе векторного индекса выполняется поиск сходства для определения N наиболее подходящих таблиц.
3. N лучших таблиц вместе со схемой таблицы и аналитическим вопросом объединяются в запрос для LLM, чтобы выбрать K наиболее релевантных таблиц.
4. Вверху K таблиц возвращаются пользователю для проверки или изменения.
5. Возобновляется стандартный процесс Text-to-SQL с подтверждёнными пользователем таблицами.

Мы будем воспроизводить этот мощный двухэтапный подход.

### Наш план: упрощённая репликация

Это руководство поможет вам создать генератор SQL в двух частях. Сначала мы создадим основной механизм, который преобразует естественный язык в SQL. Затем мы добавим интеллектуальную функцию поиска таблиц.

1. **Основная система:** мы создадим базовую цепочку. Она принимает вопрос и список имён таблиц для создания SQL-запроса.
* **Пользовательский ввод:** предоставляет аналитический вопрос, выбранные таблицы и диалект SQL.
* **Извлечение схемы:** система извлекает соответствующие схемы таблиц из хранилища метаданных.
* **Сборка запроса:** объединяет вопрос, схемы и диалект в запрос.
* **Генерация LLM:** модель выводит SQL-запрос.
* **Проверка и выполнение:** запрос проверяется на безопасность, выполняется, и результаты возвращаются.

2. **Система с улучшением RAG:** мы добавим компонент поиска. Этот компонент автоматически предлагает правильные таблицы для любого вопроса.
* **Автономное индексирование:** журналы SQL-запросов суммируются с помощью LLM, встраиваются и сохраняются в векторном индексе с метаданными.
* **Пользовательский запрос:** пользователь предоставляет аналитический вопрос на естественном языке.
* **Извлечение:** вопрос встраивается, сопоставляется с векторным хранилищем, и возвращаются N лучших таблиц-кандидатов.
* **Выбор таблицы:** LLM ранжирует и выбирает K наиболее релевантных таблиц.
* **Извлечение схемы и создание запроса:** система извлекает схемы для этих таблиц и создаёт запрос Text-to-SQL.
* **Генерация SQL:** LLM генерирует SQL-запрос.
* **Проверка и выполнение:** запрос проверяется, выполняется, и результаты + SQL возвращаются пользователю.

Мы будем использовать Python, LangChain и OpenAI для создания системы Text-to-SQL. В качестве источника данных будет использоваться база данных SQLite в памяти.

### Практическое руководство: создание собственного генератора SQL

Давайте начнём создавать нашу систему. Следуйте этим шагам, чтобы создать работающий прототип.

#### Шаг 1: настройка среды

Сначала мы устанавливаем необходимые библиотеки Python. LangChain помогает нам соединить компоненты. Langchain-openai обеспечивает подключение к LLM. FAISS помогает создать наш ретривер, а Pandas отображает данные в удобном виде.

```
!pip install -qU langchain langchain-openai faiss-cpu pandas langchain_community
```

Далее вы должны настроить свой API-ключ OpenAI. Этот ключ позволяет нашему приложению использовать модели OpenAI.

```
import os
from getpass import getpass

OPENAI_API_KEY = getpass("Enter your OpenAI API key: ")
os.environ["OPENAI_API_KEY"] = OPENAI_API_KEY
```

#### Шаг 2: моделирование базы данных

Система Text-to-SQL нуждается в базе данных для запросов. Для этого демонстрационного примера мы создадим простую базу данных SQLite в памяти. Она будет содержать три таблицы: users, pins и boards. Эта настройка имитирует базовую версию структуры данных Pinterest.

```
import sqlite3
import pandas as pd

# Create a connection to an in-memory SQLite database
conn = sqlite3.connect(':memory:')
cursor = conn.cursor()

# Create tables
cursor.execute('''
CREATE TABLE users (
    user_id INTEGER PRIMARY KEY,
    username TEXT NOT NULL,
    join_date DATE NOT NULL,
    country TEXT
)
''')

cursor.execute('''
CREATE TABLE pins (
    pin_id INTEGER PRIMARY KEY,
    user_id INTEGER,
    board_id INTEGER,
    image_url TEXT,
    description TEXT,
    created_at DATETIME,
    FOREIGN KEY(user_id) REFERENCES users(user_id),
    FOREIGN KEY(board_id) REFERENCES boards(board_id)
)
''')

cursor.execute('''
CREATE TABLE boards (
    board_id INTEGER PRIMARY KEY,
    user_id INTEGER,
    board_name TEXT NOT NULL,
    category TEXT,
    FOREIGN KEY(user_id) REFERENCES users(user_id)
)
''')

# Insert sample data
cursor.execute("INSERT INTO users (user_id, username, join_date, country) VALUES (1, 'alice', '2023-01-15', 'USA')")
cursor.execute("INSERT INTO users (user_id, username, join_date, country) VALUES (2, 'bob', '2023-02-20', 'Canada')")

# ... (далее следует вставка данных в таблицы)

conn.commit()
print("Database created and populated successfully.")
```

#### Шаг 3: создание основной цепочки Text-to-SQL

Языковая модель не может напрямую видеть нашу базу данных. Ей нужно знать структуры таблиц, или схемы. Мы создаём функцию для получения операторов `CREATE TABLE`. Эта информация сообщает модели о столбцах, типах данных и ключах.

```
def get_table_schemas(conn, table_names):
    """Fetches the CREATE TABLE statement for a list of tables."""
    schemas = []
    cursor = conn.cursor() # Get cursor from the passed connection
    for table_name in table_names:
        query = f"SELECT sql FROM sqlite_master WHERE type='table' AND name='{table_name}';"
        cursor.execute(query)
        result = cursor.fetchone()
        if result:
            schemas.append(result[0])
    return "\n\n".join(schemas)

# Example usage
sample_schemas = get_table_schemas(conn, ['users', 'pins'])
print(sample_schemas)
```

С функцией схемы готово, мы создаём нашу первую цепочку. Шаблон запроса инструктирует модель о её задаче. Он объединяет схемы и вопрос пользователя. Затем мы подключаем этот запрос к модели.

```
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough, RunnableLambda
import sqlite3 # Import sqlite3

template = """
You are a master SQL expert. Based on the provided table schema and a user's question, write a syntactically correct SQLite SQL query.
Only return the SQL query and nothing else.
Here is the database schema:
{schema}
Here is the user's question:
{question}
"""

prompt = ChatPromptTemplate.from_template(template)
llm = ChatOpenAI(model="gpt-4.1-mini", temperature=0)
sql_chain = prompt | llm | StrOutputParser()

Let's test our chain with a question where we explicitly provide the table names.

user_question = "How many pins has alice created?"
table_names_provided = ["users", "pins"]

# Retrieve the schema in the main thread before invoking the chain
schema = get_table_schemas(conn, table_names_provided)

# Pass the schema directly to the chain
generated_sql = sql_chain.invoke({"schema": schema, "table_names": table_names_provided, "question": user_question})

print("User Question:", user_question)
print("Generated SQL:", generated_sql)

# Clean the generated SQL by removing markdown code block syntax
cleaned_sql = generated_sql.strip()
if cleaned_sql.startswith("```sql"):
    cleaned_sql = cleaned_sql[len("```sql"):].strip()
if cleaned_sql.endswith("```"):
    cleaned_sql = cleaned_sql[:-len("```")].strip()

print("Cleaned SQL:", cleaned_sql)

# Let's run the generated SQL to verify it works
try:
    result_df = pd.read_sql_query(cleaned_sql, conn)
    display(result_df)
except Exception as e:
    print(f"Error executing SQL query: {e}")
```

Система правильно сгенерировала SQL и нашла правильный ответ.

#### Шаг 4: улучшение с помощью RAG для выбора таблиц

Наша основная система работает хорошо, но требует от пользователей знания имён таблиц. Это именно та проблема, которую решила команда Pinterest Text-to-SQL. Мы теперь реализуем RAG для выбора таблиц. Мы начнём с написания простых описаний естественного языка для каждой таблицы. Эти описания отражают смысл содержимого каждой таблицы.

```
table_summaries = {
    "users": "Contains information about individual users, including their username, join date, and country of origin.",
    "pins": "Contains data about individual pins, linking to the user who created them and the board they belong to. Includes descriptions and creation timestamps.",
    "boards": "Stores information about user-created boards, including the board's name, category, and the user who owns it."
}
```

Далее мы создаём векторный магазин. Этот инструмент преобразует наши сводки в числовые представления (вложения). Он позволяет нам находить наиболее релевантные сводки таблиц для вопроса пользователя с помощью поиска по сходству.

```
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import FAISS
from langchain.schema import Document

# Create LangChain Document objects for each summary
summary_docs = [
    Document(page_content=summary, metadata={"table_name": table_name})
    for table_name, summary in table_summaries.items()
]

embeddings = OpenAIEmbeddings()
vector_store = FAISS.from_documents(summary_docs, embeddings)
retriever = vector_store.as_retriever()
print("Vector store created successfully.")
```

#### Шаг 5: объединение всего в цепочку с поддержкой RAG

Теперь мы создаём окончательную интеллектуальную цепочку. Эта цепочка автоматизирует весь процесс. Она принимает вопрос, использует ретривер для поиска релевантных таблиц, извлекает их схемы, а затем передаёт всё нашей `sql_chain`.

```
def get_table_names_from_docs(docs):
    """Extracts table names from the metadata of retrieved documents."""
    return [doc.metadata['table_name'] for doc in docs]

# We need a way to get schema using table names and the connection within the chain
# Use the thread-safe function that recreates the database for each call
def get_schema_for_rag(x):
    table_names = get_table_names_from_docs(x['table_docs'])
    # Call the thread-safe function to get schemas
    schema = get_table_schemas(conn, table_names)
    return {"question": x['question'], "table_names": table_names, "schema": schema}

full_rag_chain = (
    RunnablePassthrough.assign(
        table_docs=lambda x: retriever.invoke(x['question'])
    )
    | RunnableLambda(get_schema_for_rag) # Use RunnableLambda to call the schema fetching function
    | sql_chain # Pass the dictionary with question, table_names, and schema to sql_chain
)

Let's test the complete system. We ask a question without mentioning any tables. The system should handle everything.

user_question_no_tables = "Show me all the boards created by users from the USA."

# Pass the user question within a dictionary
final_sql = full_rag_chain.invoke({"question": user_question_no_tables})

print("User Question:", user_question_no_tables)
print("Generated SQL:", final_sql)

# Clean the generated SQL by removing markdown code block syntax, being more robust
cleaned_sql = final_sql.strip()
if cleaned_sql.startswith("```sql"):
    cleaned_sql = cleaned_sql[len("```sql"):].strip()
if cleaned_sql.endswith("```"):
    cleaned_sql = cleaned_sql[:-len("```")].strip()

# Also handle cases where there might be leading/trailing newlines after cleaning
cleaned_sql = cleaned_sql.strip()

print("Cleaned SQL:", cleaned_sql)

# Verify the generated SQL
try:
    result_df = pd.read_sql_query(cleaned_sql, conn)
    display(result_df)
except Exception as e:
    print(f"Error executing SQL query: {e}")
```

### Заключение

Мы успешно создали прототип, который показывает, как создать генератор SQL. Для переноса этого в производственную среду требуется больше шагов. Вы можете автоматизировать процесс суммирования таблиц. Вы также можете включить исторические запросы в векторный магазин для повышения точности. Это соответствует пути, выбранному командой Pinterest Text-to-SQL. Эта основа представляет собой чёткий путь к созданию мощного инструмента для работы с данными.

### Часто задаваемые вопросы

**Q1. Что такое система Text-to-SQL?**

A. Система Text-to-SQL переводит вопросы, написанные на простом языке (например, на английском), в запросы к базе данных SQL. Это позволяет пользователям, не являющимся техническими специалистами, получать данные без написания кода.

**Q2. Почему RAG полезен для Text-to-SQL?**

A. RAG помогает системе автоматически находить наиболее релевантные таблицы базы данных для вопроса пользователя. Это избавляет пользователей от необходимости знать структуру базы данных.

**Q3. Что такое LangChain?**

A. LangChain — это фреймворк для разработки приложений, основанных на языковых моделях. Он помогает соединять различные компоненты, такие как запросы, модели и ретриверы, в единую цепочку.

