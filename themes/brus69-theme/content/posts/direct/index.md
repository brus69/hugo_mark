---
title: Алгоритм проверки выдачи по SEO
date: 2023-01-15T09:00:00-07:00
draft: false
tags:
  - red
  - case
description: Описание Алгоритм проверки выдачи по SEO
h1: Алгоритм22 проверки выдачи по SEO E-commerce
featured: true
featuredImage: featured.jpg
---
## Заголовок 22222

> Tempor proident minim aliquip reprehenderit dolor et ad anim Lorem duis sint eiusmod. Labore ut ea duis dolor. Incididunt consectetur proident qui occaecat incididunt do nisi Lorem. Tempor do laborum elit laboris excepteur eiusmod do. Eiusmod nisi excepteur ut amet pariatur adipisicing Lorem.

```python
def content_site(url):
  r = requests.get(url, timeout=3)
  soup = BeautifulSoup(r.text)
  data = {
      'title': soup.title.get_text(),
      'h1': soup.h1.get_text(),
      'description': soup.find('meta', attrs={'name':'description'}).get('content'),
      'content': soup.find('div', class_='container').get_text()
  }
  return data
```

Occaecat nulla excepteur dolore[ ](nic.ru)[excepteur]() duis eiusmod ullamco officia anim in voluptate ea

![](assets/20251013_111118_image22.png)

occaecat officia. Cillum sint esse velit ea officia minim fugiat. Elit ea esse id aliquip pariatur cupidatat id duis minim incididunt ea ea. Anim ut duis sunt nisi.

Culpa cillum 12312 sit voluptate voluptate eiusmod dolor.

![](assets/20251013_111231_Screenshot1_2.png)

Enim nisi Lorem ipsum irure est excepteur voluptate eu in enim nisi. Nostrud ipsum Lorem anim sint labore consequat do.

| col1 | col2 | col3  | Итого | Результат |
| ---- | ---- | ----- | ---------- | ------------------ |
| 2    | 1    | 2     | 2          | 1                  |
| 4    | 4    | 3     | 3          | 4                  |
| 3    | 2    | 4     | 5          | 6                  |
| 3424 | 3432 | 34325 | 44235      | **742**      |
| 3    | 3    | 2     | 3          | 6                  |
