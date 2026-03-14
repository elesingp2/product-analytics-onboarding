Помимо базовых навыков рекомендую повторить:

- шарить как работает Lag/Lead![[Pasted image 20260314134233.png]]
- Знать разницу **[UNION](https://www.postgresqltutorial.com/postgresql-tutorial/postgresql-union/), [EXCEPT](https://www.postgresqltutorial.com/postgresql-tutorial/postgresql-except/), [INTERCEPT](https://www.postgresqltutorial.com/postgresql-tutorial/postgresql-intersect/)**.

**Задание:**

На встрече 1-1 мы обсудим как работает этот код:

1.

```sql
select
  code, model, price,
  avg(price) over w as avgprc,
  row_number() over w as number
from PC
window w as (
  order by code
  rows between current row and 5 following
)
```

2.

```sql
WITH user_activity AS (
  SELECT
    username,
    TO_TIMESTAMP(registration_date, 'YYYY-MM-DD HH24:MI:SS.US') AS registration_date,
    TO_TIMESTAMP(prompt_date, 'YYYY-MM-DD HH24:MI:SS.US') AS prompt_date
  FROM
    results
  WHERE
    registration_date IS NOT NULL
),
retention_counts AS (
  SELECT
    registration_date::DATE AS registration_date,
    date_series::DATE AS date_series,
    COUNT(DISTINCT username) AS user_count
  FROM
    user_activity
  CROSS JOIN LATERAL generate_series(registration_date, registration_date + interval '7 day', interval '1 day') AS d(date_series)
  WHERE
    date_series <= prompt_date
  GROUP BY
    registration_date::DATE, date_series::DATE
),
initial_user_count AS (
  SELECT
    registration_date::DATE AS registration_date,
    COUNT(DISTINCT username) AS initial_user_count
  FROM
    user_activity
  GROUP BY
    registration_date::DATE
)
SELECT
  r.registration_date,
  r.date_series,
  r.user_count AS active_users,
  i.initial_user_count AS new_users,
  ROUND((r.user_count * 100.0 / i.initial_user_count),1) AS retention_rate
FROM
  retention_counts r
JOIN
  initial_user_count i
ON
  r.registration_date = i.registration_date
ORDER BY
  r.registration_date, r.date_series;
```

#### Основные конструкции SQL

Нужно знать и уметь использовать без подсказок:

- **CTE (WITH ...)** — именованные подзапросы, делают сложные запросы читаемыми. Выше уже есть пример с retention — разбери почему там используется CTE, а не подзапрос.
- **CASE WHEN** — условная логика прямо в SELECT, аналог if/else.
- **COALESCE** — замена NULL на дефолтное значение.
- **FILTER (WHERE ...)** — агрегация с условием, например `COUNT(*) FILTER (WHERE status = 'active')`.

Задание: переписать retention-запрос из задания №2 так, чтобы добавить разбивку по платформе (mobile / web) — используй CASE WHEN.

#### Оптимизация запросов

Базово нужно понимать:

- Что такое **индекс** и почему запросы без него работают медленно на больших таблицах.
