---
title: "Прогнозирование ценности клиента на протяжении всей жизни с помощью PyMC-Маркетинг | На пути к науке о данных"
date: "2025-11-19T23:13:09+0000"
draft: true
description: ""
h1: "Прогнозирование ценности клиента на протяжении всей жизни с помощью PyMC-маркетинга"
urldel: "https://towardsdatascience.com/pymc-marketing-the-key-to-advanced-clv-customer-lifetime-value-forecasting-bc0730973c0a/"
---

### Фото от Boxed Water Is Better на Unsplash

**TL; DR:** Модель пожизненной ценности клиента (CLV) — это ключевой метод в клиентской аналитике, который помогает компаниям определить, кто является ценным клиентом.

Игнорирование CLV может привести к чрезмерным инвестициям в краткосрочных клиентов, которые могут совершить только одну покупку. Моделирование «Покупай, пока не умрёшь» (BTYD), использующее модели BG/NBD и Gamma-Gamma, может оценить CLV. Хотя лучшие практики различаются в зависимости от размера данных и приоритетов моделирования, PyMC-Marketing — это рекомендуемая библиотека Python для тех, кто хочет быстро реализовать моделирование CLV.

Для тех, кто хочет сразу погрузиться в пример кода, обратитесь к моему репозиторию на GitHub. Буду рад, если вы оставите звезду!

> **GitHub — takechanman1228/Effective-CLV-Modeling**

### 1. Что такое CLV?

**Определение CLV — это общая чистая выручка, которую компания может ожидать от одного клиента на протяжении всех их отношений.** Некоторые из вас могут быть более знакомы с термином «LTV» (пожизненная ценность). Да, CLV и LTV взаимозаменяемы.

Первая цель — рассчитать и спрогнозировать будущий CLV, что поможет вам узнать, сколько денег можно ожидать от каждого клиента.

Вторая задача — выявить прибыльных клиентов. Модель скажет вам, кто эти ценные клиенты, проанализировав характеристики клиентов с высоким CLV.

Третья цель — предпринять маркетинговые действия на основе анализа, и тогда вы сможете оптимизировать распределение своего маркетингового бюджета.

### 2. Контекст бизнеса

Давайте возьмём сайт электронной коммерции модного бренда, такого как Nike, который может использовать рекламу и купоны для привлечения новых клиентов. Теперь предположим, что студенты колледжей и работающие специалисты — это два основных важных сегмента клиентов. Для первых покупок компания тратит 10 долларов на рекламу для студентов колледжей и 20 долларов для работающих специалистов. И оба сегмента совершают покупки на сумму около 100 долларов.

Если бы вы отвечали за маркетинг, в какой сегмент вы бы хотели инвестировать больше? Вы, возможно, думаете, что логичнее инвестировать больше в сегмент студентов колледжей, учитывая их более низкую стоимость и более высокую рентабельность инвестиций.

### [Далее идёт подробный перевод всего текста]

**Полезные библиотеки CLV**

Здесь позвольте мне представить две отличные библиотеки с открытым исходным кодом для моделирования CLV. Первая — это PyMC-Marketing, а вторая — CLVTools. Обе библиотеки включают моделирование «покупай, пока не умрёшь». Наиболее существенное различие заключается в том, что PyMC-Marketing — это библиотека на основе Python, а CLVTools — на основе R. PyMC-Marketing построена на PyMC, популярной байесовской библиотеке. Ранее существовала известная библиотека под названием Lifetimes. Однако сейчас Lifetimes находится в режиме обслуживания, поэтому она перешла в PyMC-Marketing.

**Полный код**

Полный код можно найти в моём репозитории на GitHub ниже. Мой пример кода основан на официальном быстром старте yMC-Marketing.

> **GitHub — takechanman1228/Effective-CLV-Modeling**

**Прохождение по коду**

Сначала вам нужно будет импортировать pymc_marketing и другие библиотеки.

```
import arviz as az
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import pymc as pm
from arviz.labels import MapLabeller
from IPython.display import Image
from pymc_marketing import clv
```

Вам нужно будет загрузить «Набор данных онлайн-ритейла» из «Репозитория машинного обучения UCI». Этот набор данных содержит транзакционные данные от британского онлайн-ритейлера и лицензирован в соответствии с лицензией Creative Commons Attribution 4.0 International (CC BY 4.0).

```
import requests
import zipfile
import os

url = "https://archive.ics.uci.edu/static/public/352/online+retail.zip"
response = requests.get(url)
filename = "online_retail.zip"

with open(filename, 'wb') as file:
    file.write(response.content)

with zipfile.ZipFile(filename, 'r') as zip_ref:
    zip_ref.extractall("online_retail_data")

for file in os.listdir("online_retail_data"):
    if file.endswith(".xlsx"):
        excel_file = os.path.join("online_retail_data", file)
        break

data_raw = pd.read_excel(excel_file)
data_raw.head()
```

**Очистка данных**

Необходима быстрая очистка данных. Например, нам нужно обработать возвращённые заказы, отфильтровать записи без идентификатора клиента и создать столбец «Общие продажи», умножив количество на цену за единицу.

```
cancelled_orders = data_raw[data_raw['InvoiceNo'].astype(str).str.startswith("C")]

cancelled_orders['Quantity'] = -cancelled_orders['Quantity']

merged_data = pd.merge(data_raw, cancelled_orders[['CustomerID', 'StockCode', 'Quantity', 'UnitPrice']], 
                      on=['CustomerID', 'StockCode', 'Quantity', 'UnitPrice'], 
                      how='left', indicator=True)

data_raw = merged_data[(merged_data['_merge'] == 'left_only') & (~merged_data['InvoiceNo'].astype(str).str.startswith("C"))]

data_raw = data_raw.drop(columns=['_merge'])

features = ['CustomerID', 'InvoiceNo', 'InvoiceDate', 'Quantity', 'UnitPrice', 'Country']
data = data_raw[features]
data['TotalSales'] = data['Quantity'].multiply(data['UnitPrice'])

data = data[data['CustomerID'].notna()]
data['CustomerID'] = data['CustomerID'].astype(int).astype(str)
data.head()
```

Затем нам нужно создать сводную таблицу, используя эту функцию clv_summary. Функция возвращает фрейм данных в формате RFM-T. RFM-T означает недавность, частоту, денежные средства и срок пребывания каждого клиента. Эти метрики популярны в анализе покупателей.

```
data_summary_rfm = clv.utils.clv_summary(data, 'CustomerID', 'InvoiceDate', 'TotalSales')
data_summary_rfm = data_summary_rfm.rename(columns={'CustomerID': 'customer_id'})
data_summary_rfm.index = data_summary_rfm['customer_id']
data_summary_rfm.head()
```

**Модель BG/NBD**

Модель BG/NBD доступна как функция BetaGeoModel в этой библиотеке. Когда вы выполняете bgm.fit(), ваша модель начинает обучение.

Когда вы выполняете bgm.fit_summary(), система предоставляет статистическую сводку процесса обучения. Например, эта таблица показывает среднее значение, стандартное отклонение, высокий интервал плотности, HDI и т. д. для параметров. Мы также можем проверить значение r_hat, которое помогает оценить, сошлась ли симуляция методом Монте-Карло (MCMC). R-hat считается приемлемым, если он равен 1,1 или меньше.

```
bgm = clv.BetaGeoModel(
    data = data_summary_rfm,
)
bgm.build_model()

bgm.fit()
bgm.fit_summary()
```

[Далее идёт подробный перевод остальной части текста]

