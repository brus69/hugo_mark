---
title: "Анализ цифрового маркетинга с помощью Python и MySQL | На пути к науке о данных"
date: "2025-11-19T23:13:09+0000"
draft: false
description: ""
h1: "Анализ цифрового маркетинга с помощью Python и MySQL"
urldel: "https://towardsdatascience.com/digital-marketing-analysis-simultaneously-with-python-and-mysql-ee00e05a3813/"
---

## Введение

В этом кратком путешествии мы рассмотрим небольшой и простой набор данных с основными метриками маркетинга веб-сайта, такими как «пользователи», «сеансы» и «отскоки», за период в пять месяцев.

Цель этой настройки — не столько понять производительность веб-сайта, сколько получить базовые, но полезные знания для ответа на ряд обязательных операционных маркетинговых вопросов.

Мы сосредоточимся на двух мощных и наиболее используемых цифровых инструментах, исследуя два пути, которые приведут нас к одинаковым результатам в конце дня.

С одной стороны, мы изучим синтаксис **MySQL Workbench** с различными запросами, параллельно для каждого вопроса будем использовать синтаксис **Python** с графическими и визуальными ресурсами. Обе среды будут озаглавлены как # MySQL и # Python соответственно. Для каждого вопроса — примечания и объяснения по обоим кодам для более глубокого понимания.

### MySQL

```
SELECT * FROM case_sql;
```

### Python

```
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
import matplotlib.style as style
%matplotlib inline
color = sns.color_palette()
from pandas.plotting import table 
from datetime import datetime

df = pd.read_csv("case.csv", sep=";")

df.shape
(31507, 7)

df.sample(15)
```

### 1. Давайте посмотрим на распределение пользователей по типу устройства

#### MySQL

```
SELECT 
    device_category, 
    ROUND(COUNT(users) / (SELECT 
                COUNT(users)
            FROM
                case_sql) * 100,
            1) AS percent 
FROM
    case_sql 
GROUP BY 1 
ORDER BY 1; 
```

Мы видим, что распределение по устройствам показывает **мобильные и настольные устройства** как наиболее частые типы доступа.

#### # Python

```
df.device_category.value_counts(dropna=False).plot(kind='pie', figsize=(8,4),
                                                  explode = (0.02, 0.02, 0.02),
                                                  autopct='%1.1f%%',
                                                  startangle=150);

plt.xticks(rotation=0, horizontalalignment="center")
plt.title("Device distribution", fontsize=10, loc="right");
```

### 2. Каков этот сценарий с точки зрения бренда?

#### MySQL

```
SELECT 
    brand, 
    COUNT(users) AS users, 
    ROUND(COUNT(users) / (SELECT 
                COUNT(users)
            FROM
                case_sql) * 100,
            2) AS percent 
FROM
    case_sql 
GROUP BY 1; 
```

На уровне бренда у бренда 2 наибольшее количество посещений — 56,28 % против 43,72 % от общего числа посещений у бренда 1.

#### Python

```
absolut = df["brand"].value_counts().to_frame()

absolut.plot(kind='pie', subplots=True, autopct='%1.2f%%', 
             explode= (0.05, 0.05), startangle=20, 
             legend=False, fontsize=12, figsize=(8,4))

plt.xticks(rotation=0, horizontalalignment="center")
plt.title("Brand's distribution", fontsize=10, loc="right");

display(absolut) 
```

### 3. В какой день недели на сайт бренда 1 пришло больше всего пользователей?

#### MySQL

```
SELECT 
    date, 
    DAYNAME(date) AS day_name, 
    SUM(users) AS users 
FROM
    case_sql 
WHERE
    brand = 'Brand 1' 
GROUP BY 1 
ORDER BY 3 DESC 
LIMIT 1; 
```

Из 298 412 пользователей с сентября 2019 года по январь 2020 года день, когда на сайт **бренда 1** пришло больше всего пользователей, — **22.11.2019**, всего 885 посещений, это была пятница.

#### Python

```
brand_1 = df[df["brand"] == "Brand 1"].copy()

''' sum total users that came from all "channelgrouping" for the same date, 
assign it 'brandgroup' no matter the type of device '''

brandgroup = brand_1.groupby(["date","weekday"])[["default_channel_grouping","users"]].sum()

users = brandgroup[brandgroup["users"] == brandgroup.users.max()].copy()

users.reset_index(["date"], inplace=True)
users.reset_index(["weekday"], inplace=True)

print(f"""Date: {users.date} nnTotal users: {users.users} nnDay of week: {users.weekday}""")
```

### 3.1 Сколько пользователей пришло на бренд 2 в тот же день?

#### MySQL

```
SELECT 
    DATE(date) AS date, 
    DAYNAME(date) AS dayofweek, 
    SUM(CASE
        WHEN brand = 'Brand 1' THEN users 
        ELSE NULL
    END) AS b1_users,
    SUM(CASE
        WHEN brand = 'Brand 2' THEN users 
        ELSE NULL
    END) AS b2_users
FROM
    case_sql 
GROUP BY 1, 2 
ORDER BY 3 DESC 
LIMIT 1; 
```

Фактически, оба бренда зафиксировали наибольшее количество посещений **в один и тот же день**.

#### Python

```
brand_2 = df[df["brand"] == "Brand 2"].copy()

brandgroup.rename(columns = {'users':'brand1_users'}, inplace = True)

brandgroup["brand2_users"] = brand_2.groupby(["date","weekday"])[["default_channel_grouping","users"]].sum()

users2 = brandgroup[brandgroup["brand2_users"] == brandgroup.brand2_users.max()].copy()
```

### 4. Из всех групп каналов, какая внесла наибольшее количество пользователей?

#### MySQL

```
SELECT 
    default_channel_grouping AS channels,
    SUM(users) AS total_users,
    ROUND(SUM(users) / (SELECT 
                SUM(users)
            FROM
                case_sql) * 100,
            1) AS percent 
FROM
    case_sql
GROUP BY 1
ORDER BY 2 DESC;
```

**Органический поиск** — это канал, который генерирует больше всего пользователей (почти 141 000), представляя почти половину от общего числа посещений на обоих сайтах, за ним следуют Платный поиск и Прямой. Дисплей занимает 4-е место, а Социальные сети — 6-е, внося 6722 пользователя.

#### Python

```
ax = df.groupby("default_channel_grouping")["users"].sum().sort_values(ascending=True)
.plot(kind="bar", figsize=(9,6), fontsize=12, linewidth=2, 
      color=sns.color_palette("rocket"), grid=False, table=False)

for p in ax.patches:
    ax.annotate("%.0f" % p.get_height(), (p.get_x() + p.get_width() / 2., p.get_height()),
                ha='center', va='center', xytext=(0, 7), textcoords='offset points')

plt.xlabel("Channel groups", fontsize=10)
plt.xticks(rotation=90, horizontalalignment="center")
plt.ylabel("Absolute values", fontsize=10)
plt.title("Best channel group (highest number of users)", fontsize=10, loc="right");
```

### 4.1 Из всех групп каналов, какая внесла наибольшее количество пользователей с разбивкой по брендам?

#### MySQL

```
SELECT 
    default_channel_grouping AS channels,
    SUM(CASE 
        WHEN brand = 'brand 1' THEN users 
        ELSE NULL 
    END) AS Brand_1, 
    SUM(CASE 
        WHEN brand = 'brand 2' THEN users 
        ELSE NULL 
    END) AS Brand_2 
FROM
    case_sql
GROUP BY 1 
ORDER BY 3 DESC; 
```

#### Python

```
type_pivot = df.pivot_table(
    columns="brand",
    index="default_channel_grouping",
    values="users", aggfunc=sum)

display(type_pivot)

type_pivot.sort_values(by=["Brand 2"], ascending=True).plot(kind="bar", figsize=(12,8) ,fontsize = 15)
plt.xlabel("Channel groups", fontsize=10)
plt.xticks(rotation=90, horizontalalignment="center")
plt.ylabel("Absolute values", fontsize=10)
plt.title("Channel groups by brand (highest number of users)", fontsize=10, loc="right");
```

**Органический поиск** привлёк 105 062 пользователей для бренда 2 и 35 911 пользователей для бренда 1. За исключением «Other», в котором бренд 1 превосходит, бренд 2 вносит наибольший вклад, привлекая пользователей на сайт по всем каналам.

### 5. Среди всех каналов, какой бренд внёс процент платных сеансов не менее 5 % в течение 2019 года?

#### MySQL

```
SELECT 
    brand,
    default_channel_grouping AS channels,
    ROUND(SUM(sessions) / (SELECT 
                SUM(sessions)
            FROM
                case_sql) * 100,
            1) AS percent
FROM
    case_sql
WHERE
    default_channel_grouping IN ('Paid Search' , 'Paid Social', 'Display', 'Other Advertising') 
        AND date < '2020-01-01' 
GROUP BY 1 , 2
HAVING percent > 5 
ORDER BY 1 , 3 DESC
```

#### Python

```
df = df.groupby(["date","brand","default_channel_grouping"])["sessions"].sum().to_frame().copy()

df["percent"] = (df.apply(lambda x: x/x.sum())*100).round(2)

df = df.reset_index().copy()

df.sample(5)
```

### 6. Сколько посещений получили оба бренда по типу устройства?

#### MySQL

```
SELECT 
    brand,
    SUM(CASE
        WHEN device_category = 'Desktop' THEN users
        ELSE NULL
    END) AS desktop,
    SUM(CASE
        WHEN device_category = 'Mobile' THEN users
        ELSE NULL
    END) AS mobile,
    SUM(CASE
        WHEN device_category = 'Tablet' THEN users
        ELSE NULL
    END) AS tablet
FROM
    case_sql
GROUP BY 1
ORDER BY 1;
```

#### Python

```
type_pivot = df.pivot_table(
    columns="device_category",
    index="brand",
    values="users", aggfunc=sum)

display(type_pivot)

ax = type_pivot.sort_values(by=["brand"], ascending=False).plot(kind="bar", figsize=(12,8) ,fontsize = 15);

for p in ax.patches:
    ax.annotate("%.0f" % p.get_height(), (p.get_x() + p.get_width() / 2., p.get_height()), ha='center', va='center', xytext=(0, 7), textcoords='offset points')

plt.xlabel("Brands", fontsize=10)
plt.xticks(rotation=0, horizontalalignment="center")
plt.ylabel("Absolute values", fontsize=10)
plt.title("Brand by type of device", fontsize=10, loc="right")
plt.legend(["Desktop","Mobile","Tablet"]);
```

### 6.1 Каков средний уровень использования устройства по каналам?

#### MySQL

```
SELECT 
    default_channel_grouping,
    AVG(CASE
        WHEN device_category = 'Desktop' THEN users
        ELSE NULL
    END) AS desktop,
    AVG(CASE
        WHEN device_category = 'Mobile' THEN users
        ELSE NULL
    END) AS mobile,
    AVG(CASE
        WHEN device_category = 'Tablet' THEN users
        ELSE NULL
    END) AS tablet
FROM
    case_sql
GROUP BY 1
ORDER BY 1;
```

#### Python

```
type_pivot = df.pivot_table(
    columns="device_category",
    index="default_channel_grouping",
    values="users", aggfunc=np.mean)

display(type_pivot)

type_pivot.sort_values(by=["default_channel_grouping"], ascending=False).plot(kind="bar", figsize=(12,8) ,fontsize = 15);

plt.xlabel("Date", fontsize=10)
plt.xticks(rotation=90, horizontalalignment="center")
plt.ylabel("Absolute values", fontsize=10)
plt.title("Average use of device types by channel grouping", fontsize=10, loc="right")
plt.legend(["Desktop","Mobile","Tablet"]);
```

### 7. Как оценить показатель отказов групп каналов?

Показатель отказов рассчитывается как общее количество отказов, делённое на общее количество сеансов.

#### MySQL

```
SELECT 
    default_channel_grouping,
    SUM(sessions) AS sessions,
    SUM(bounces) AS bounces,
    ROUND(SUM(bounces) / SUM(sessions) * 100, 2) AS bounces_r
FROM
    case_sql
GROUP BY 1
ORDER BY 4 DESC;
```

**Средний показатель отказов: 54,93 %** (avg_bounces_r)

```
SELECT 
    SUM(sessions) AS sessions,
    SUM(bounces) AS bounces,
    ROUND(SUM(bounces) / SUM(sessions) * 100, 2) AS bounces_r,
    AVG(ROUND(bounces/sessions*100, 2)) AS avg_bounces_r
FROM
    case_sql;
```

#### Python

```
dfbounce = df.groupby("default_channel_grouping")["users"].sum().to_frame()

dfbounce["sessions"] = df.groupby("default_channel_grouping")["sessions"].sum()

dfbounce["bounces"] = df.groupby("default_channel_grouping")["bounces"].sum()

dfbounce["bounces_r"] = dfbounce.apply(lambda x: 0.0 if x["sessions"] == 0.0 else (x["bounces"] / x["sessions"])*100, axis=1).round(2)

dff = dfbounce.copy()

dfbounce.drop(["users"],axis=1,inplace=True)

dfbounce.sort_values(by="bounces_r", ascending=False)
```

### 7.1 Похоже ли, что показатель отказов на сайте улучшается или ухудшается с течением времени?

#### MySQL

```
SELECT 
    YEAR(date) AS year, 
    MONTH(date) AS month,  
    DATE_FORMAT(date, '%b') AS month_,   
    ROUND(SUM(bounces) / SUM(sessions) * 100, 2) AS bounces_r  
    case_sql                        
GROUP BY 1 , 2 , 3                  
ORDER BY 1 , 2 , 3;                 
```

#### Python

```
df_date = df.groupby("date")[['sessions','bounces']].sum()

''' create function to assess the bounce rate, assign it as 'bounce_r'
Return 0 if session's value is 0, else divide the bounces by sessions 
for each date and multiply it by 100 to get the percentage '''

def div(bounces, sessions):
    return lambda row: 0.0 if row[sessions] == 0.0 else float((row[bounces]/(row[sessions])))*100

df_date["bounce_r"] = (df_date.apply(div('bounces', 'sessions'), axis=1)).round(1)

df_date.drop(["sessions","bounces"], axis=1, inplace=True)

ax = df_date.plot(kind="line", figsize=(14,6), fontsize=12, linewidth=2)

plt.xlabel("Date", fontsize=10)
plt.xticks(rotation=90, horizontalalignment="center")
plt.ylabel("Rate", fontsize=10)
plt.title("Evolution of the bounce rate over time", fontsize=10, loc="right");
```

Показатель отказов на сайте **улучшается** с течением времени.

### 7.2 Показатель отказов по каналам и брендам

#### Python

```
b1 = df[df["brand"] == "Brand 1"]
b2 = df[df["brand"] == "Brand 2"]

dfbrand = b1.groupby("default_channel_grouping")["sessions"].sum().to_frame()
dfbrand.rename(columns={"sessions":"sessions1"}, inplace=True)

dfbrand["bounces1"] = b1.groupby("default_channel_grouping")["bounces"].sum()

dfbrand["1bounces_r"] = dfbrand.apply(lambda x: 0.0 if x["sessions1"] == 0.0 else (x["bounces1"] / x["sessions1"]*100), axis=1).round(2)

dfbrand["sessions2"] = b2.groupby("default_channel_grouping")["sessions"].sum()

dfbrand["bounces2"] = b2.groupby("default_channel_grouping")["bounces"].sum()

dfbrand["2bounces_r"] = dfbrand.apply(lambda x: 0.0 if x["sessions2"] == 0.0 else (x["bounces2"] / x["sessions2"]*100), axis=1).round(2)

dfbrand.sort_values(by="1bounces_r", ascending=False)
```

### 8. Пропорции между входящими и платными медиа

#### MySQL

```
SELECT 
    brand,
    CASE
        WHEN
            default_channel_grouping IN ('Paid Search', 
                'Paid Social',
                'Display',
                'Other Advertising')
        THEN
            'Paid'
        WHEN
            default_channel_grouping IN ('Direct',
                'Native',
                'Organic Search',
                'Referral',
                'Social',
                'Email',
                '(Other)')
        THEN
            'Organic'
        ELSE NULL
    END AS media,
    ROUND(SUM(bounces) / SUM(sessions) * 100, 2) AS bounce_r
FROM
    case_sql
GROUP BY brand , media
ORDER BY 1;
```

#### Python

```
media_dict = 
{
    'Display': 'paid',
    'Paid Search': 'paid',
    'Paid Social': 'paid',
    'Other Advertising': 'paid',
    'Direct': 'organic',
    'Native': 'organic',
    'Organic Search': 'organic',
    'Referral': 'organic',
    'Social': 'organic',
    'Email': 'organic',
    '(Other)': 'organic'
}

df['media'] = df['default_channel_grouping'].map(media_dict)

cols = ['brand','media','sessions','bounces']

df = df.reindex(columns = cols)

df = df.groupby(["brand","media"])[["sessions","bounces"]].sum()

df["bounces_r"] = df.apply(lambda x: 0.0 if x["sessions"] == 0.0 else (x["bounces"] / x["sessions"])*100, axis=1).round(2)
```

### Заключение

Как и было обещано, мы рассмотрели пошаговый подход к проведению простого цифрового маркетингового анализа, используя MySQL Workbench и Python.

Оба инструмента имеют свои особенности, требования, но рассуждения относительно схожи, если не учитывать их графические возможности и ограничения.

Не стесняйтесь загружать наборы данных и исследовать их, практикуясь в технических деталях, рассмотренных здесь, и реализуя новый код для дальнейших маркетинговых вопросов.

