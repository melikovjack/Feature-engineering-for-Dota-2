# Feature-engineering-for-Dota-2
# Dota Advanced Pipeline

## Что делает `pipe_adv.ipynb`

Ноутбук собирает табличные, draft- и chat-фичи по матчу, кодирует их в общую sparse-матрицу и обучает `LogisticRegression`.

### 1. Базовые матчевые фичи

Используются поля матча:

- `region`
- `game_mode`
- `avg_mmr`
- признаки приватности аккаунтов игроков

Что из этого строится:

- `region` → target encoding со сглаживанием
- `game_mode` → target encoding со сглаживанием
- `avg_mmr_log`
- `avg_mmr_sqrt`
- `avg_mmr_frac`
- `mmr_missing`
- `radiant_private_count`, `dire_private_count`, `private_count_diff`
- `radiant_public_count`, `dire_public_count`, `public_count_diff`

### 2. Draft-фичи

- **`HeroesEncoder`**
  - sparse-кодирование героев с `+1` для Radiant и `-1` для Dire
- **`HeroPairEncoder`**
  - sparse-кодирование пар героев внутри команды
  - пары с частотой ниже `hero_pair_min_freq` отбрасываются

### 3. Counter-фичи

Используется внешний файл:

- `data/hero_counter_top5_2024-01-01_2024-11-30.csv`

Из него строятся:

- количество положительных контр-попаданий
- сумма контр-весов
- средний контр-вес
- максимальный контр-вес
- разницы Radiant vs Dire

### 4. ADV-фичи

Используются временные ряды:

- `radiant_gold_adv`
- `radiant_exp_adv`

По ним строятся:

- длина ряда
- пустой ли ряд
- `mean`, `std`, `min`, `max`, `last`
- средний положительный и отрицательный перевес
- `slope`
- число смен знака
- доля положительных значений
- максимальный по модулю перевес
- линейный тренд: `slope`, `intercept`, `r2`
- бинированные признаки по квантилям

### 5. Chat-фичи

В ноутбуке сейчас активны три текстовых слоя:

1. `TF-IDF` отдельно для `radiant_chat_clean` и `dire_chat_clean`
2. Простые chat stats:
   - число сообщений
   - число токенов
   - `unique_ratio`
   - `caps_share`
   - `exclamation_count`
   - `question_count`
   - `profanity_hits`
   - `aggression_hits`
3. Dota tactical lexicon features из [dota_chat_tactical_features.py](https://github.com/Ovsyannikov-Ivan1/Feature-engineering-for-Dota-2/blob/main/dota_chat_tactical_features.py)
   - lane calls
   - push calls
   - fight calls
   - vision calls
   - smoke/gank calls
   - rosh calls
   - economy calls
   - defense calls
   - rune calls

Для каждой категории считаются:

- хиты у Radiant
- хиты у Dire
- разница Radiant − Dire
- общий tactical hits
- число уникальных тактических категорий
