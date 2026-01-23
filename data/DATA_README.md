#  Анализ продаж видеоигр
**Источник:** [Kaggle](https://www.kaggle.com/datasets/volodymyrpivoshenko/video-game-sales-dataset)
**Дата загрузки:** 19.01.2026

## Изменения, внесённые в данные:
1. Создала диаграмму со связями в dbdiagram.io
```
Table publishers {
  publisher_id integer [primary key]
  publisher_name varchar(255)
}

Table genres {
  genre_id integer [primary key]
  genre_name varchar(100)
}

Table platforms {
  platform_id integer [primary key]
  platform_name varchar(50)
}

Table games {
  game_id integer [primary key]
  name varchar(500)
  global_rank integer
  platform varchar(50)
  release_year integer
  genre varchar(100)
  publisher varchar(255)
  publisher_id integer
  genre_id integer
}

Table game_platforms {
  game_id integer [primary key]
  platform_ids integer[]
}

Table sales {
  game_id integer [primary key]
  na_sales decimal(10,2)
  eu_sales decimal(10,2)
  jp_sales decimal(10,2)
  other_sales decimal(10,2)
}

Ref: games.publisher_id > publishers.publisher_id
Ref: games.genre_id > genres.genre_id
Ref: game_platforms.game_id > games.game_id
Ref: game_platforms.platform_ids > platforms.platform_id
Ref: sales.game_id > games.game_id
```
2. Создала 6 таблиц в pgAdmin
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
CREATE TABLE game_platforms (
    game_id INTEGER PRIMARY KEY,
    platform_ids INTEGER[],
    FOREIGN KEY (game_id) REFERENCES games(game_id)
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
3. Создала 6 разных csv файлов с помощью кода на Python. Также в таблице game_platforms сделала массив для столбца platform_id

```
import pandas as pd

def normalize_data(input_file='video_game_sales.csv'):
    df = pd.read_csv(input_file)
    
    # 1. Создаю файл  publishers
    publishers = df[['Publisher']].dropna().drop_duplicates().reset_index(drop=True)
    publishers['publisher_id'] = range(1, len(publishers) + 1)
    publishers.columns = ['publisher_name', 'publisher_id']
    publishers.to_csv('publishers.csv', index=False)
    
    # 2. Создаю файл genres
    genres = df[['Genre']].dropna().drop_duplicates().reset_index(drop=True)
    genres['genre_id'] = range(1, len(genres) + 1)
    genres.columns = ['genre_name', 'genre_id']
    genres.to_csv('genres.csv', index=False)
    print(f"genres.csv: {len(genres)} строк")
    
    # 3. Создаю файл platforms
    platforms = df[['Platform']].dropna().drop_duplicates().reset_index(drop=True)
    platforms['platform_id'] = range(1, len(platforms) + 1)
    platforms.columns = ['platform_name', 'platform_id']
    platforms.to_csv('platforms.csv', index=False)
    print(f"platforms.csv: {len(platforms)} строк")
    
    # 4. Создаю файл games
    games = df.copy()
    games['game_id'] = range(1, len(games) + 1)
    
    # Добавляю publisher_id и genre_id
    publisher_dict = dict(zip(publishers['publisher_name'], publishers['publisher_id']))
    genre_dict = dict(zip(genres['genre_name'], genres['genre_id']))
    
    games['publisher_id'] = games['Publisher'].map(publisher_dict)
    games['genre_id'] = games['Genre'].map(genre_dict)
    
    # Переименовываю колонки
    games = games[['game_id', 'Name', 'Rank', 'Platform', 'Year', 'Genre', 
                   'Publisher', 'publisher_id', 'genre_id']]
    games.columns = ['game_id', 'name', 'global_rank', 'platform', 'release_year', 
                     'genre', 'publisher', 'publisher_id', 'genre_id']
    
    games.to_csv('games.csv', index=False)
    print(f"games.csv: {len(games)} строк")
    
    # 5. Создаю файл game_platforms
    platform_dict = dict(zip(platforms['platform_name'], platforms['platform_id']))
    
    # ВРЕМЕННО добавляю platform_id в games для группировки
    games_with_platform = games.copy()
    games_with_platform['platform_id'] = games_with_platform['platform'].map(platform_dict)
    
    game_platforms = games_with_platform.groupby('game_id')['platform_id'].apply(
        lambda x: '{' + ','.join(map(str, sorted(set(x)))) + '}'
    ).reset_index()
    
    game_platforms.to_csv('game_platforms.csv', index=False)
    print(f"game_platforms.csv: {len(game_platforms)} строк")
    
    # 6. Создаю файл sales 
    sales = pd.DataFrame({
        'game_id': games['game_id'],
        'na_sales': df['NA_Sales'],
        'eu_sales': df['EU_Sales'],
        'jp_sales': df['JP_Sales'],
        'other_sales': df['Other_Sales']
    })
    
    sales.to_csv('sales.csv', index=False)
    print(f"sales.csv: {len(sales)} строк")
    
    print("\nГотово! 6 таблиц создано.")

normalize_data()
```
4. Импортировала данные в созданные таблицы в pgAdmin.
