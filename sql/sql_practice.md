# SQL Practice Log

## Basics: SELECT, WHERE, ORDER BY, LIMIT

### 1. Останні 4 фільми (за роком)
```sql
-- Задача: last four movies, most recent first
-- Пастка: спершу зробив WHERE year >= 2010 — вгадав дані, а не вимогу.
-- "Останні N" — це завжди ORDER BY + LIMIT, без хардкоду значень.
SELECT * FROM movies
ORDER BY year DESC
LIMIT 4;
```

### 2. Міста на захід від Чикаго
```sql
-- Задача: cities west of Chicago, west to east
-- Пастка: взяв latitude замість longitude.
-- Захід-схід = longitude, північ-південь = latitude.
SELECT * FROM north_american_cities
WHERE longitude < -87.629798
ORDER BY longitude;
```

### 3. 3-тє і 4-те найбільші міста США
```sql
-- Задача: third and fourth largest US cities by population
-- Прийом: OFFSET пропускає топ-2, LIMIT бере наступні 2
SELECT * FROM north_american_cities
WHERE country = 'United States'
ORDER BY population DESC
LIMIT 2 OFFSET 2;
```

## JOIN

### 1. Базовий INNER — зв'язок не через id
```sql
-- Задача: будівлі, у яких є працівники
-- Пастка: ON — не обов'язково id. Тут зв'язок через назву будівлі.
-- DISTINCT потрібен, бо в одній будівлі багато працівників
SELECT DISTINCT buildings.building_name
FROM buildings
JOIN employees ON buildings.building_name = employees.building;
```

### 2. LEFT JOIN — зберегти рядки без пари
```sql
-- Задача: всі будівлі та ролі, включно з порожніми будівлями
-- Пастка: "including empty" = LEFT, бо INNER викине будівлі без працівників
SELECT DISTINCT buildings.building_name, employees.role
FROM buildings
LEFT JOIN employees ON buildings.building_name = employees.building;
```

### 3. Знайти те, у чого немає пари
```sql
-- Задача: будівлі без жодного працівника
-- Головний патерн. Три частини працюють разом:
-- FROM buildings — зберігаємо саме будівлі (те, що шукаємо)
-- LEFT JOIN — дозволяє NULL там, де пари нема
-- WHERE ... IS NULL — лишає тільки ті, де пари не знайшлось
SELECT buildings.building_name
FROM buildings
LEFT JOIN employees ON employees.building = buildings.building_name
WHERE employees.building IS NULL;
```

## Aggregation: COUNT, SUM, GROUP BY, HAVING

### 1. Кількість працівників по ролях
```sql
-- Задача: number of Employees of each role in the studio
-- Каркас: GROUP BY розкладає рядки по купках, COUNT рахує кожну окремо
SELECT role, COUNT(role)
FROM employees
GROUP BY role;
```

### 2. Порожній результат ≠ зламаний запит
```sql
-- Задача: total years employed by all Engineers
-- Пастка: спершу написав role = "Engineers" (з різних міркувань додав s) —
-- синтаксис правильний, але WHERE нічого не знайшов, бо в даних "Engineer".
-- Урок: порожній результат — спершу звіряй значення в WHERE з даними,
-- а не одразу лізь міняти логіку запиту.
SELECT SUM(years_employed)
FROM employees
WHERE role = 'Engineer';
```

### 3. JOIN + GROUP BY + SUM(вираз) — усе разом
```sql
-- Задача: total domestic + international sales attributed to each director
-- Пастка 1: GROUP BY movie_id замість GROUP BY director — кожен movie_id
-- унікальний, тому кожна "група" = один фільм, і SUM нічого не сумує.
-- Пастка 2: "total sales" = одне число, тому SUM(a + b) в одному виразі,
-- а не два окремих SUM(a), SUM(b) в різних колонках.
SELECT movies.director,
       SUM(boxoffice.domestic_sales + boxoffice.international_sales) AS total_sales
FROM movies
LEFT JOIN boxoffice ON movies.id = boxoffice.movie_id
GROUP BY movies.director;
```

## Subqueries: IN, EXISTS

### 1. IN — підзапит повертає список значень
```sql
-- Задача: будівлі, у яких працює хоча б один Artist
-- Пастка: спершу переплутав building (назва будівлі в employees)
-- з name (ім'я людини) — підзапит має повертати саме назви будівель.
-- Механіка: спочатку виконується внутрішній SELECT (список назв),
-- потім зовнішній перевіряє building_name IN (цей список).
SELECT building_name
FROM buildings
WHERE building_name IN (
    SELECT building FROM employees WHERE role = 'Artist'
);
```

### 2. EXISTS — підзапит перевіряє наявність зв'язку
```sql
-- Задача: та сама (будівлі з хоча б одним Artist), інший підхід
-- Різниця з IN: EXISTS не будує список наперед, а для КОЖНОГО рядка
-- зовнішньої таблиці питає "чи є хоч один пов'язаний рядок?".
-- Тому підзапит звертається до зовнішньої таблиці через аліас (b.building_name)
-- і сам по собі окремо не запуститься.
SELECT building_name
FROM buildings b
WHERE EXISTS (
    SELECT 1 FROM employees e
    WHERE e.building = b.building_name AND e.role = 'Artist'
);
```
