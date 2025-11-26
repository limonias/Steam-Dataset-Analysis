CREATE OR REPLACE TABLE steam_games_flat AS
WITH raw_data AS (
    SELECT unnest(games) AS g
    FROM read_json_auto(
        'https://github.com/vintagedon/steam-dataset-2025/raw/main/data/01_raw/steam_2025_5k-dataset-games_20250831.json.gz',
        maximum_object_size=268435456
    )
)
SELECT
    g.app_details.appid AS appid,
    g.app_details.data.name AS name,
    COALESCE(g.app_details.data.price_overview.final / 100.0, 0) AS price,
    g.app_details.data.is_free AS is_free,
    g.app_details.data.release_date.date AS release_date_raw,
    g.app_details.data.genres AS genres,
    g.app_details.data.categories AS categories,
    COALESCE(g.app_details.data.recommendations.total, 0) AS recommendations
FROM raw_data
WHERE g.app_details.success = true;
100% ?██████████████████████████████████████? (00:00:05.55 elapsed)


SELECT name, recommendations, price
FROM steam_games_flat
ORDER BY recommendations DESC
LIMIT 20;
┌────────────────────────────────────────────────┬─────────────────┬────────┐
│                      name                      │ recommendations │ price  │
│                    varchar                     │      int64      │ double │
├────────────────────────────────────────────────┼─────────────────┼────────┤
│ Tom Clancy's Rainbow SixR Siege X              │         1213757 │    0.0 │
│ Rust                                           │         1043348 │  39.99 │
│ PAYDAY 2                                       │          436763 │   9.99 │
│ Rocket LeagueR                                 │          434282 │    0.0 │
│ Monster Hunter: World                          │          305588 │  29.99 │
│ Arma 3                                         │          221519 │  29.99 │
│ Dying Light 2 Stay Human: Reloaded Edition     │          150434 │  19.79 │
│ Starfield                                      │          109647 │  69.99 │
│ Killing Floor 2                                │           85750 │   2.99 │
│ METAL GEAR SOLID V: THE PHANTOM PAIN           │           68724 │  19.99 │
│ Shadow of the Tomb Raider: Definitive Edition  │           65010 │    0.0 │
│ DEVOUR                                         │           64666 │   9.99 │
│ Batman: Arkham Asylum Game of the Year Edition │           53914 │  19.99 │
│ Little Nightmares                              │           49343 │  19.99 │
│ Gorilla Tag                                    │           40179 │  19.99 │
│ Deep Rock Galactic: Survivor                   │           36448 │  12.99 │
│ Marvel's Guardians of the Galaxy               │           32274 │  59.99 │
│ Deus Ex: Mankind Divided                       │           29714 │  29.99 │
│ World War 3                                    │           28948 │    0.0 │
│ The Talos Principle                            │           28413 │  28.99 │
├────────────────────────────────────────────────┴─────────────────┴────────┤
│ 20 rows                                                         3 columns │
└───────────────────────────────────────────────────────────────────────────┘

SELECT RIGHT(release_date_raw, 4) AS release_year, COUNT(*) AS games_count
FROM steam_games_flat
WHERE length(release_date_raw) >= 4
GROUP BY 1 ORDER BY 2 DESC;

┌──────────────┬─────────────┐
│ release_year │ games_count │
│   varchar    │    int64    │
├──────────────┼─────────────┤
│ 2025         │        1175 │
│ 2024         │        1109 │
│ 2023         │         741 │
│ 2022         │         734 │
│ 2021         │         703 │
│ soon         │         621 │
│ 2020         │         522 │
│ 2019         │         442 │
│ 2018         │         427 │
│ 2017         │         385 │
│ 2016         │         305 │
│ nced         │         297 │
│ 2015         │         167 │
│ 2014         │         117 │
│ 2026         │          77 │
│ 2013         │          36 │
│ 2012         │          24 │
│ 2011         │          16 │
│ 2009         │          11 │
│ 2010         │          10 │
│ 2027         │           4 │
│ 2028         │           3 │
│ 2006         │           3 │
│ 2008         │           3 │
│ 9 г.         │           2 │
│ 2007         │           1 │
│ 2034         │           1 │
├──────────────┴─────────────┤
│ 27 rows          2 columns │
└────────────────────────────┘

SELECT g_unnested.description AS genre, ROUND(AVG(price), 2) AS avg_price
FROM (
    SELECT price, UNNEST(genres) AS g_unnested
    FROM steam_games_flat WHERE price > 0
)
GROUP BY 1 ORDER BY 2 DESC;

┌───────────────────────┬───────────┐
│         genre         │ avg_price │
│        varchar        │  double   │
├───────────────────────┼───────────┤
│ Симуляторы            │     710.0 │
│ Инди                  │     465.0 │
│ Экшены                │     465.0 │
│ Animation & Modeling  │      46.8 │
│ Accion                │     30.66 │
│ Design & Illustration │     23.07 │
│ Strategy              │     21.65 │
│ Simulation            │     21.21 │
│ Software Training     │      17.0 │
│ Multijogador Massivo  │     16.99 │
│ Video Production      │     16.49 │
│ Aventura              │      16.0 │
│ Web Publishing        │     15.86 │
│ Massively Multiplayer │     15.59 │
│ Game Development      │     15.14 │
│ Utilities             │     14.12 │
│ Education             │     13.69 │
│ Sports                │     13.43 │
│ Racing                │     12.22 │
│ Simulacao             │     11.29 │
│ RPG                   │     11.01 │
│ Free To Play          │     10.81 │
│ Nudity                │     10.66 │
│ Acao                  │     10.49 │
│ Sexual Content        │     10.24 │
│ Early Access          │      9.92 │
│ Adventure             │      9.64 │
│ Audio Production      │      9.61 │
│ Action                │      9.22 │
│ Indie                 │      8.19 │
│ Casual                │      6.96 │
│ Photo Editing         │      6.65 │
│ Gore                  │      6.54 │
│ Violent               │      6.11 │
│ Aksiyon               │      4.49 │
│ Strateji              │      1.49 │
│ Simulasyon            │      1.49 │
│ Independant           │      0.52 │
├───────────────────────┴───────────┤
│ 38 rows                 2 columns │
└───────────────────────────────────┘

SELECT c_unnested.description AS category, COUNT(*) AS frequency
FROM (
    SELECT UNNEST(categories) AS c_unnested FROM steam_games_flat
)
GROUP BY 1 ORDER BY 2 DESC LIMIT 10;

┌────────────────────────────┬───────────┐
│          category          │ frequency │
│          varchar           │   int64   │
├────────────────────────────┼───────────┤
│ Single-player              │      6692 │
│ Family Sharing             │      5208 │
│ Steam Achievements         │      3148 │
│ Full controller support    │      2003 │
│ Steam Cloud                │      1929 │
│ Downloadable Content       │      1780 │
│ Multi-player               │      1736 │
│ Co-op                      │      1080 │
│ Partial Controller Support │      1075 │
│ PvP                        │       882 │
├────────────────────────────┴───────────┤
│ 10 rows                      2 columns │
└────────────────────────────────────────┘

SELECT
    CASE WHEN is_free THEN 'Free' ELSE 'Paid' END AS type,
    COUNT(*) AS total_games,
    ROUND(AVG(recommendations), 0) AS avg_reviews
FROM steam_games_flat GROUP BY 1;

┌─────────┬─────────────┬─────────────┐
│  type   │ total_games │ avg_reviews │
│ varchar │    int64    │   double    │
├─────────┼─────────────┼─────────────┤
│ Paid    │        6324 │       709.0 │
│ Free    │        1643 │       763.0 │
└─────────┴─────────────┴─────────────┘
