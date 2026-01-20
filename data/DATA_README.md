#  Анализ продаж видеоигр
**Источник:** [Kaggle](https://www.kaggle.com/datasets/volodymyrpivoshenko/video-game-sales-dataset)
**Дата загрузки:** 19.01.2026

## Изменения, внесённые в данные:
1. Разделила датасет на несколько таблиц

-- Создание таблицы издателей
```
CREATE TABLE publishers (
    publisher_id SERIAL PRIMARY KEY,
    publisher_name VARCHAR(255) NOT NULL UNIQUE
)
```
-- Создание таблицы жанров
```
CREATE TABLE genres (
    genre_id SERIAL PRIMARY KEY,
    genre_name VARCHAR(100) NOT NULL UNIQUE
)
```
-- Создание таблицы платформ
```
CREATE TABLE platforms (
    platform_id SERIAL PRIMARY KEY,
    platform_name VARCHAR(50) NOT NULL UNIQUE
)
```

-- Создание таблицы игр
```
CREATE TABLE games (
    game_id SERIAL PRIMARY KEY,
    name VARCHAR(500) NOT NULL,
    global_rank INTEGER,
    release_year INTEGER CHECK (release_year >= 1980 AND release_year <= EXTRACT(YEAR FROM CURRENT_DATE)),
    publisher_id INTEGER NOT NULL,
    genre_id INTEGER NOT NULL,
    FOREIGN KEY (publisher_id) REFERENCES publishers(publisher_id) ON DELETE RESTRICT,
    FOREIGN KEY (genre_id) REFERENCES genres(genre_id) ON DELETE RESTRICT
)
```
-- Связующая таблица для связи M:M (игры ↔ платформы)
```
CREATE TABLE game_platforms (
    game_id INTEGER NOT NULL,
    platform_id INTEGER NOT NULL,
    PRIMARY KEY (game_id, platform_id),
    FOREIGN KEY (game_id) REFERENCES games(game_id) ON DELETE CASCADE,
    FOREIGN KEY (platform_id) REFERENCES platforms(platform_id) ON DELETE CASCADE
)
```

-- Таблица продаж
```
CREATE TABLE sales (
    game_id INTEGER PRIMARY KEY,
    na_sales DECIMAL(10,2) DEFAULT 0.00 CHECK (na_sales >= 0),
    eu_sales DECIMAL(10,2) DEFAULT 0.00 CHECK (eu_sales >= 0),
    jp_sales DECIMAL(10,2) DEFAULT 0.00 CHECK (jp_sales >= 0),
    other_sales DECIMAL(10,2) DEFAULT 0.00 CHECK (other_sales >= 0),
    FOREIGN KEY (game_id) REFERENCES games(game_id) ON DELETE CASCADE
)
```

-- Индексы для оптимизации запросов
```
CREATE INDEX idx_games_year ON games(release_year);
CREATE INDEX idx_games_publisher ON games(publisher_id);
CREATE INDEX idx_sales_na ON sales(na_sales DESC);
CREATE INDEX idx_sales_eu ON sales(eu_sales DESC);
```
