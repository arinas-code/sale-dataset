#  Анализ продаж видеоигр
**Источник:** [Kaggle](https://www.kaggle.com/datasets/volodymyrpivoshenko/video-game-sales-dataset)
**Дата загрузки:** 19.01.2026

## Изменения, внесённые в данные:
1.  В столбце `year` значения 'N/A' преобразованы в NULL
2.  Данные загружены в PostgreSQL для анализа

## Для воспроизведения анализа:
-- Создание таблицы и очистка данных

1.Загрузила сырые данные во временную таблицу
```
COPY games_staging FROM 'Путь к файлу' WITH (FORMAT CSV, HEADER)
```
2. Затем поменяла тип данных на int с условием, что данные не 'N/A'
```
CREATE TABLE games AS
SELECT 
    rank,
    name,
    platform,
    NULLIF(year, 'N/A')::INT AS year, -- преобразование
    genre,
    publisher,
    na_sales,
    eu_sales,
    jp_sales,
    other_sales,
    global_sales
FROM games_staging
```
3. Верификация
```
SELECT COUNT(*) FROM games;  -- 16,598
SELECT COUNT(*) FROM games WHERE year IS NULL;  -- 271
```
