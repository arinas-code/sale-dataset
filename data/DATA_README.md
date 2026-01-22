#  Анализ продаж видеоигр
**Источник:** [Kaggle](https://www.kaggle.com/datasets/volodymyrpivoshenko/video-game-sales-dataset)
**Дата загрузки:** 19.01.2026

## Изменения, внесённые в данные:
1. Разделила датасет на 6 таблиц
```
--Создание таблицы издателей
create table publishers (
  publisher_id integer PRIMARY KEY,
  publisher_name varchar(255) not null
);
--Создание таблицы жанров
create Table genres (
  genre_id integer primary key,
  genre_name varchar(100) not null
);
--Создание таблицы с платформами
create Table platforms (
  platform_id integer primary key,
  platform_name varchar(50) not null
);
--Создание таблицы с играми
CREATE TABLE games (
    game_id INTEGER PRIMARY KEY,
    name VARCHAR(500) NOT NULL,
    global_rank INTEGER,
    release_year INTEGER NULL,
    publisher_id INTEGER,
    genre_id INTEGER,
    FOREIGN KEY (publisher_id) REFERENCES publishers(publisher_id),
    FOREIGN KEY (genre_id) REFERENCES genres(genre_id)
);
--Создание связующей таблицы M:M (игры ↔ платформы)
create Table game_platforms (
  game_id integer,
  platform_id integer,
  FOREIGN KEY (game_id) REFERENCES games(game_id),
  FOREIGN KEY (platform_id) REFERENCES platforms(platform_id),
  PRIMARY KEY (game_id, platform_id)
);
--Создание таблицы с продажами
create Table sales (
  game_id integer,
  na_sales decimal(10,2),
  eu_sales decimal(10,2),
  jp_sales decimal(10,2),
  other_sales decimal(10,2),
  FOREIGN KEY (game_id) REFERENCES games(game_id)
)
```
