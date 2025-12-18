---
title: "Автоматизированная чистка площадок в РСЯ Яндекс Директ (Мастер Кампания)"
date: "2025-12-17T23:13:09+0000"
draft: false
description: "Как эффективно очистить рекламную сеть Яндекса (РСЯ) от ботовых и мусорных площадок без ручного аудита?"
h1: "Автоматизированная чистка площадок в РСЯ(Яндекс Директ): как отсеивать «ботовые» и мусорные площадки через парсинг главных страниц на стоп-слова"
---

## Кратко по проблеме

В рекламной сети Яндекса (РСЯ) до сих пор сохраняется проблема некачественного трафика:  
- ботовые площадки,
- сайты-«перекупщики» трафика,  
- агрегаторы дешёвого контента,  
- и прочие «мусорные» ресурсы,

Если ваша целевая аудитория взрослые люди из b2b то вам надо отсеивать:
- задачники для детей
- подготовка к егэ
- игры(развлекательный контент)
- сериалы, фильмы
- книжные библиотеки и т.д.

которые **сливают бюджет**, **искажают метрики**, **понижают CTR** и **снижают ROI**.

Ручная проверка десятков или сотен площадок — неэффективна.

> **Решение** — автоматизация: парсинг главных страниц площадок с анализом текстового контента на наличие **«стоп-слов»** как индикатора низкого качества.

---

## Почему «стоп-слова» работают как маркер мусора?

Ботвенные и перекупные площадки часто:

- Используют **шаблонные фразы** для привлечения трафика (в т.ч. через парсеры и автогенерацию);  
- Содержат **дублирующий, бессмысленный или обманчивый контент**;  
- Применяют **«цепляющие» формулировки**, характерные для лохотронов:  
  - `«заработок без вложений»`,  
  - `«автоматическая выплата»`,  
  - `«бесплатно навсегда»` и т.п.;  
- Не имеют **уникального контента** или редакционной политики.

Анализ текста главной страницы позволяет выявить такие сигналы **раньше**, чем площадка начнёт «съедать» бюджет.


Этапы:
- Для этого вам нужно будет выгрузить из мастера отчетов все площадки по которым были показы
- Подготовить список минус фраз (у каждой тематики будет свой список минус фраз)
- Загрузить площадки в тестовый файлик ```data.txt```
- Запустить скрипт и запустить скрипт (можно использовать бесплатный ресурс для запуска скрипта на google colab )

## Алгоритм работы скрипта

### 1. Подготовка
- Устанавливаются и инициализируются NLP-компоненты библиотеки `natasha`:
  - сегментация (`Segmenter`),
  - морфологический словарь (`MorphVocab`),
  - обученные эмбеддинги (`NewsEmbedding`),
  - морфологический теггер (`NewsMorphTagger`).
- Из файла `data.txt` загружается список URL’ов площадок для проверки.
- Задаётся список **минус-фраз** (например: `школа`, `раскраска`, `игры`, `кредит срочно`), характерных для школьного, развлекательного или низкокачественного контента.
- Определяется множество **стоп-слов** для фильтрации служебных и частотных слов.

### 2. Предобработка минус-фраз
- Каждая минус-фраза разбивается на отдельные слова.
- Все слова **лемматизируются** (приводятся к нормальной форме: *«игры» → «игра»*, *«задания» → «задание»*), чтобы обеспечить устойчивость к морфологическим вариациям.

### 3. Обработка каждой площадки
Для каждого URL выполняются следующие шаги:
- Извлечение домена (очистка от протокола и пути).
- HTTP-запрос с таймаутом и маскирующим `User-Agent`.
- При успешном ответе (`HTTP 200`):
  - Удаляются нерелевантные HTML-элементы: `<script>`, `<style>`, `<meta>`, `<noscript>`, `<iframe>`, `<svg>`.
  - Извлекается видимый текст → очищается (нижний регистр, удаление цифр, спецсимволов, лишних пробелов).
  - Если длина текста > 300 символов — продолжается анализ; иначе домен считается «чистым».

### 4. Анализ контента (NLP-обработка)
- Текст передаётся в `Doc` (объект Natasha).
- Выполняется:
  - **Сегментация** на токены,
  - **Морфологическая разметка** (`tag_morph`),
  - **Лемматизация** каждого токена.
- Фильтрация:
  - исключаются части речи: `PUNCT`, `SYM`, `ADP`, `CONJ`, `PART`,
  - исключаются слова длиной ≤ 2 символа и стоп-слова.
- Из оставшихся лемм формируются **биграммы** (пары соседних слов) как приближение к осмысленным фразам.
- Выбираются **топ-50 самых частых биграмм** (с ограничением обрабатываемого текста до 3000 симв. для скорости).

### 5. Сопоставление с минус-фразами
Для каждой извлечённой фразы (биграммы) проверяется:
- **Быстрая фильтрация**: содержит ли фраза *хотя бы одно слово* из множества лемм минус-фраз.
- **Точное совпадение**: полностью ли совпадает с какой-либо лемматизированной минус-фразой (в т.ч. как подстрока).
- Все найденные совпадения сохраняются в виде пар:  
  `«фраза на сайте» ←→ «исходная минус-фраза»`

### 6. Классификация и логирование
- Если найдено ≥1 совпадение → домен добавляется в список **`flagged_domains`** (требует запрета в РСЯ).
- Иначе → в список **`clean_domains`**.
- Для каждого домена сохраняются:
  - исходный URL,
  - длина текста,
  - топ-фразы,
  - список совпадений,
  - статус ошибок (таймаут, HTTP-ошибки и др.).

### 7. Формирование отчётов
Генерируются три файла:

| Файл | Содержание | Назначение |
|------|------------|------------|
| `flagged_domains.txt` | Чистый список доменов (по одному на строку) | Готов к копированию в интерфейс **Яндекс.Директ → Исключения площадок** |
| `clean_domains.txt` | Список «безопасных» площадок | Для анализа, ретаргетинга, создания белых списков |
| `detailed_report.txt` | Полный отчёт с примерами совпадений, топ-фразами, статистикой | Для аудита, настройки порогов, доработки минус-фраз |

В конце выводится итог:
- Общее количество обработанных площадок,
- Количество «грязных» / «чистых»,
- Общее и среднее время обработки.

---

###  Ключевые преимущества подхода
- **Морфологическая устойчивость**: лемматизация позволяет находить совпадения при разных падежах, числах и формах (например: `«играть»`, `«играю»`, `«играл»` → `«играть»`).
- **Контекстная точность**: анализ **фраз (биграмм)** снижает ложные срабатывания по отдельным словам.
- **Высокая скорость и отказоустойчивость**: ограничение длины текста, таймауты, обработка ошибок.
- **Готовность к внедрению**: результаты сразу пригодны для загрузки в РСЯ.
- **Аудитируемость**: детальный отчёт позволяет понять *почему* площадка была отмечена.

> Подходит для регулярной (еженедельной/ежемесячной) автоматической чистки площадок в B2B, SaaS, финтехе, премиум-товарах и других нишах, где школьный, развлекательный или «лёгкий» трафик не конвертируется.

Ниже сам скрипт

```python

pip install natasha

```

```python
import requests
from bs4 import BeautifulSoup
from natasha import (
    Segmenter,
    MorphVocab,
    NewsEmbedding,
    NewsMorphTagger,
    Doc
)
from collections import Counter
import re
import time

# Инициализация компонентов Natasha
segmenter = Segmenter()
morph_vocab = MorphVocab()
emb = NewsEmbedding()
morph_tagger = NewsMorphTagger(emb)

# Чтение URL из файла
with open('data.txt', 'r') as f:
    urls = [url.strip() for url in f.readlines() if url.strip()]

# МИНУС-ФРАЗЫ (список строк)
minus_phrases = [
"школа",
"школьный",
"школьники",
"уроки",
"домашнее задание",
"игра",
"игры",
"играть",
"игровой",
"геймер",
"скачать игру",
"фильм",
"фильмы",
"кино",
"смотреть онлайн",
"сериал",
"расписание",
"уроков",
"занятий",
"расписание звонков",
"школьное расписание",
"егэ",
"огэ",
"экзамен",
"тесты",
"задания",
"тренажер",
"вопросы егэ",
"подготовка к экзамену",
"варианты огэ",
"ответы егэ",
"ответы огэ",
"картина по номерам",
"разукраска",
"разукраски",
"рисовать",
"рисунок",
"рисунки",
"картинки",
"раскраска",
"раскраски для детей",
"дети",
"детский",
"мультфильм",
"мультики",
"смотреть мультфильм",
"каникулы",
"досуг",
"гдз",
"деньги без отказа",
"взять кредит",
"займ на карту",
"срочные деньги",
"мгновенный перевод",
"займ без проверки",
"кредит с плохой историей",
"одобрение кредита",
"деньги срочно",
"займ круглосуточно",
"деньги в долг",
"кредитная карта",
]

print(f"Загружено минус-фраз: {len(minus_phrases)}")
print("Примеры минус-фраз:")
for i, phrase in enumerate(minus_phrases[:5], 1):
    print(f"  {i}. {phrase}")

# Стоп-слова для фильтрации
stop_words = {
    'и', 'в', 'на', 'с', 'по', 'для', 'не', 'что', 'как', 'это', 'то', 'но',
    'а', 'или', 'из', 'у', 'за', 'же', 'ли', 'бы', 'во', 'до', 'о', 'со', 'от',
    'об', 'ну', 'вот', 'да', 'нет', 'так', 'же', 'бы', 'вы', 'мы', 'они', 'он',
    'она', 'оно', 'меня', 'тебя', 'его', 'ее', 'нас', 'вас', 'их', 'мой', 'твой',
    'наш', 'ваш', 'свой', 'кто', 'что', 'какой', 'который', 'где', 'когда', 'куда',
    'откуда', 'почему', 'зачем', 'сколько', 'очень', 'можно', 'нужно', 'должен',
    'самый', 'еще', 'уже', 'тоже', 'только', 'просто', 'даже', 'все', 'всё'
}

def clean_text(text):
    """Очистка текста от лишних символов"""
    text = re.sub(r'\b\w*\d\w*\b', '', text)
    text = re.sub(r'[^\w\s]', ' ', text)
    text = re.sub(r'\s+', ' ', text)
    return text.strip().lower()

def extract_domain(url):
    """Извлечение доменного имени из URL"""
    domain = url.replace('http://', '').replace('https://', '')
    domain = domain.split('/')[0]
    return domain

def lemmatize_word(word):
    """Лемматизация одного слова (обертка для morph_vocab)"""
    # Для Natasha нужно указать POS и feats, но можно использовать упрощенный вариант
    try:
        # Создаем простой документ для лемматизации
        doc = Doc(word)
        doc.segment(segmenter)
        doc.tag_morph(morph_tagger)

        if doc.tokens:
            token = doc.tokens[0]
            token.lemmatize(morph_vocab)
            return token.lemma.lower()
    except:
        pass
    return word.lower()

def lemmatize_phrase(phrase):
    """Лемматизация фразы (разбивает на слова и лемматизирует каждый)"""
    words = re.findall(r'\b[а-яё]+\b', phrase.lower())
    lemmatized_words = []
    for word in words:
        lemma = lemmatize_word(word)
        if lemma and len(lemma) > 1:
            lemmatized_words.append(lemma)
    return lemmatized_words

# Лемматизируем минус-фразы
print("\nЛемматизация минус-фраз...")
minus_phrases_lemmatized = []
for phrase in minus_phrases:
    lemmatized = lemmatize_phrase(phrase)
    minus_phrases_lemmatized.append(lemmatized)
    print(f"  '{phrase}' → {lemmatized}")

def extract_top_phrases_fast(text, n=50, max_text_length=5000):
    """
    Быстрое извлечение топ-N фраз (биграмм) из текста
    """
    # Ограничиваем длину текста для скорости
    if len(text) > max_text_length:
        text = text[:max_text_length]

    # Создаем документ и обрабатываем
    doc = Doc(text)
    doc.segment(segmenter)
    doc.tag_morph(morph_tagger)

    # Собираем леммы
    lemmas = []
    for token in doc.tokens:
        token.lemmatize(morph_vocab)
        lemma = token.lemma.lower()

        # Фильтруем стоп-слова и короткие слова
        if (token.pos not in ['PUNCT', 'SYM', 'ADP', 'CONJ', 'PART'] and
            len(lemma) > 2 and
            lemma not in stop_words):
            lemmas.append(lemma)

    # Если мало слов, возвращаем пустой список
    if len(lemmas) < 2:
        return []

    # Создаем биграммы (фразы из 2 слов)
    phrases_counter = Counter()
    for i in range(len(lemmas) - 1):
        phrase = f"{lemmas[i]} {lemmas[i+1]}"
        phrases_counter[phrase] += 1

    # Возвращаем топ-N фраз
    return [phrase for phrase, count in phrases_counter.most_common(n)]

def check_minus_phrases_fast(domain_phrases, minus_phrases_list):
    """
    Быстрая проверка фраз домена против минус-фраз
    """
    matches = []

    # Создаем словарь для быстрого поиска отдельных слов из минус-фраз
    minus_words_set = set()
    for minus_phrase in minus_phrases_list:
        minus_words_set.update(minus_phrase)

    for domain_phrase in domain_phrases:
        # Разбиваем фразу домена на слова
        domain_words = domain_phrase.split()

        # Проверка 1: ищем отдельные слова из минус-фраз
        for domain_word in domain_words:
            if domain_word in minus_words_set:
                # Находим, к какой минус-фразе относится это слово
                for minus_phrase in minus_phrases_list:
                    if domain_word in minus_phrase:
                        matches.append((domain_phrase, ' '.join(minus_phrase)))
                        break

        # Проверка 2: ищем полные минус-фразы
        domain_phrase_str = ' '.join(domain_words)
        for minus_phrase in minus_phrases_list:
            if len(minus_phrase) > 1:
                minus_phrase_str = ' '.join(minus_phrase)
                if minus_phrase_str in domain_phrase_str:
                    if (domain_phrase, minus_phrase_str) not in matches:
                        matches.append((domain_phrase, minus_phrase_str))

    return list(set(matches))  # Убираем дубликаты

# Основная обработка
flagged_domains = []  # Домены с минус-фразами
clean_domains = []    # Домены без минус-фраз
all_results = {}
processed_count = 0

print(f"\n{'='*80}")
print(f"Начинаю обработку {len(urls)} доменов...")
print('='*80)

start_time = time.time()

for url in urls:
    processed_count += 1
    try:
        # Добавляем протокол если его нет
        if not url.startswith('http'):
            url = f'http://{url}'

        domain = extract_domain(url)
        print(f"\n[{processed_count}/{len(urls)}] Обработка: {domain}")

        headers = {
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
        }

        r = requests.get(url, timeout=10, headers=headers)

        if r.status_code == 200:
            soup = BeautifulSoup(r.text, 'html.parser')

            # Удаляем ненужные элементы
            for element in soup(["script", "style", "meta", "noscript", "iframe", "svg"]):
                element.decompose()

            # Получаем текст
            text = soup.get_text(separator=' ', strip=True)
            text = clean_text(text)

            if len(text) > 300:
                # Извлекаем топ-50 фраз
                top_phrases = extract_top_phrases_fast(text, n=50, max_text_length=3000)

                if top_phrases:
                    # Проверяем фразы против минус-фраз
                    matches = check_minus_phrases_fast(top_phrases, minus_phrases_lemmatized)

                    # Сохраняем результаты
                    all_results[domain] = {
                        'url': url,
                        'text_length': len(text),
                        'top_phrases': top_phrases,
                        'matches': matches,
                        'has_minus_phrases': len(matches) > 0,
                        'phrase_count': len(top_phrases)
                    }

                    # Выводим информацию
                    if matches:
                        print(f"НАЙДЕНО МИНУС-ФРАЗ: {len(matches)}")
                        flagged_domains.append(domain)
                        # Показываем примеры совпадений
                        for i, (domain_phrase, minus_phrase) in enumerate(matches[:2], 1):
                            print(f"     {i}. '{domain_phrase}' ←→ '{minus_phrase}'")
                    else:
                        print(f"  ✓ ЧИСТЫЙ (извлечено фраз: {len(top_phrases)})")
                        clean_domains.append(domain)

                    # Показываем топ-5 фраз для информации
                    if top_phrases and len(top_phrases) > 0:
                        print(f"  Топ-3 фразы: {', '.join(top_phrases[:3])}")
                else:
                    print(f"  ? Не удалось извлечь фразы")
                    all_results[domain] = {
                        'url': url,
                        'error': 'Не удалось извлечь фразы',
                        'has_minus_phrases': False
                    }
                    clean_domains.append(domain)
            else:
                print(f"  ? Мало текста ({len(text)} символов)")
                all_results[domain] = {
                    'url': url,
                    'error': f'Мало текста ({len(text)} симв.)',
                    'has_minus_phrases': False
                }
                clean_domains.append(domain)
        else:
            print(f"  ✗ Ошибка HTTP: {r.status_code}")
            all_results[domain] = {
                'url': url,
                'error': f'HTTP {r.status_code}',
                'has_minus_phrases': False
            }
            clean_domains.append(domain)

    except requests.exceptions.Timeout:
        print(f"  ⏰ Таймаут при запросе")
        all_results[domain] = {
            'url': url,
            'error': 'Таймаут',
            'has_minus_phrases': False
        }
        clean_domains.append(domain)
    except requests.exceptions.RequestException as e:
        print(f" Ошибка сети: {str(e)[:50]}...")
        all_results[domain] = {
            'url': url,
            'error': str(e)[:100],
            'has_minus_phrases': False
        }
        clean_domains.append(domain)
    except Exception as e:
        print(f" Неизвестная ошибка: {str(e)[:50]}...")
        all_results[domain] = {
            'url': url,
            'error': str(e)[:100],
            'has_minus_phrases': False
        }
        clean_domains.append(domain)

# Расчет времени выполнения
end_time = time.time()
processing_time = end_time - start_time

# Итоговый отчет
print(f"\n{'='*80}")
print("ИТОГОВЫЙ ОТЧЕТ")
print('='*80)

print(f"\nСтатистика обработки:")
print(f"  Всего доменов в файле: {len(urls)}")
print(f"  Успешно обработано: {len(all_results)}")
print(f"  Домены с минус-фразами: {len(flagged_domains)}")
print(f"  Чистые домены: {len(clean_domains)}")
print(f"  Время выполнения: {processing_time:.1f} секунд")
print(f"  Среднее время на домен: {processing_time/max(1, len(all_results)):.2f} сек.")

# Детальный отчет
if flagged_domains:
    print(f"\n{'='*80}")
    print("ДЕТАЛЬНЫЙ ОТЧЕТ ПО ДОМЕНАМ С МИНУС-ФРАЗАМИ:")
    print('='*80)

    for i, domain in enumerate(flagged_domains, 1):
        result = all_results[domain]
        print(f"\n{i}. {domain}")
        print(f"   URL: {result['url']}")
        print(f"   Длина текста: {result.get('text_length', 'N/A')} симв.")
        print(f"   Извлечено фраз: {result.get('phrase_count', 'N/A')}")
        print(f"   Найдено совпадений: {len(result['matches'])}")

        # Группируем совпадения по минус-фразам
        minus_phrase_groups = {}
        for domain_phrase, minus_phrase in result['matches']:
            if minus_phrase not in minus_phrase_groups:
                minus_phrase_groups[minus_phrase] = []
            minus_phrase_groups[minus_phrase].append(domain_phrase)

        for minus_phrase, domain_phrases in minus_phrase_groups.items():
            print(f"     Минус-фраза: '{minus_phrase}'")
            for domain_phrase in domain_phrases[:3]:  # Показываем до 3 примеров
                print(f"       • '{domain_phrase}'")
            if len(domain_phrases) > 3:
                print(f"       ... и еще {len(domain_phrases) - 3}")

# Сохранение результатов
print(f"\n{'='*80}")
print("СОХРАНЕНИЕ РЕЗУЛЬТАТОВ:")
print('='*80)

# Сохраняем списки доменов
with open('flagged_domains.txt', 'w', encoding='utf-8') as f:
    f.write(f"# Домены с минус-фразами ({len(flagged_domains)})\n")
    f.write("# Сгенерировано: " + time.strftime("%Y-%m-%d %H:%M:%S") + "\n")
    f.write("=" * 50 + "\n")
    for domain in flagged_domains:
        f.write(f"{domain}\n")

with open('clean_domains.txt', 'w', encoding='utf-8') as f:
    f.write(f"# Чистые домены ({len(clean_domains)})\n")
    f.write("# Сгенерировано: " + time.strftime("%Y-%m-%d %H:%M:%S") + "\n")
    f.write("=" * 50 + "\n")
    for domain in clean_domains:
        f.write(f"{domain}\n")

# Сохраняем детальный отчет
with open('detailed_report.txt', 'w', encoding='utf-8') as f:
    f.write("ОТЧЕТ АНАЛИЗА ДОМЕНОВ\n")
    f.write("=" * 60 + "\n\n")
    f.write(f"Дата анализа: {time.strftime('%Y-%m-%d %H:%M:%S')}\n")
    f.write(f"Всего доменов: {len(all_results)}\n")
    f.write(f"Домены с минус-фразами: {len(flagged_domains)}\n")
    f.write(f"Чистые домены: {len(clean_domains)}\n")
    f.write(f"Время обработки: {processing_time:.1f} сек.\n\n")

    f.write("МИНУС-ФРАЗЫ ДЛЯ ПРОВЕРКИ:\n")
    f.write("-" * 40 + "\n")
    for i, phrase in enumerate(minus_phrases, 1):
        f.write(f"{i:2}. {phrase}\n")

    f.write("\n" + "=" * 60 + "\n")
    f.write("ДОМЕНЫ С МИНУС-ФРАЗАМИ:\n")
    f.write("=" * 60 + "\n\n")

    for domain in flagged_domains:
        result = all_results[domain]
        f.write(f"Домен: {domain}\n")
        f.write(f"URL: {result['url']}\n")
        f.write(f"Найдено совпадений: {len(result['matches'])}\n")
        f.write("Совпадения:\n")
        for domain_phrase, minus_phrase in result['matches']:
            f.write(f"  • '{domain_phrase}' ←→ '{minus_phrase}'\n")
        f.write("Топ-10 фраз домена:\n")
        for i, phrase in enumerate(result.get('top_phrases', [])[:10], 1):
            f.write(f"  {i:2}. {phrase}\n")
        f.write("-" * 50 + "\n\n")

    f.write("\n" + "=" * 60 + "\n")
    f.write("ЧИСТЫЕ ДОМЕНЫ:\n")
    f.write("=" * 60 + "\n\n")
    for i, domain in enumerate(clean_domains, 1):
        f.write(f"{i:4}. {domain}\n")

print(f"\nФайлы сохранены:")
print(f"  flagged_domains.txt - {len(flagged_domains)} доменов с минус-фразами")
print(f"  clean_domains.txt - {len(clean_domains)} чистых доменов")
print(f"  detailed_report.txt - детальный отчет")

print(f"\n✓ Обработка завершена!")
```