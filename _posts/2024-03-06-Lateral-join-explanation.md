---
title: "Lateral join: Wyjaśnienie i prosty przykład"
toc: true
---

Lateral join to obok left, right, inner i outer join kolejny ze sposobów złączeń w relacyjnych bazach danych. Pozwala na wygenerowanie wielu wierszy na podstawie pojedynczego wiersza danych oraz do odniesienia się do zmiennej z pozapytania. Wykorzystywany w PostgreSQL, Oracle, DB2  MS SQL. Jego znajomość jest przydane i warto się jej przyjrzeć. Może dzięki temu zaoczędzicie w przyszłości czas na szukanie rozwiązania lub zabłyśniecie na rozmowie rekrutacyjnej.


### Jak działa SELECT

Select działa jak pętla która zwraca wiersze z tabeli.

```sql
SELECT * FROM table
```
Taki select w pythonie mógłby wyglądać następująco

```python
for row in table:
    print(row)
```


### Jak działa LATERAL

Jest to zagnieżdżona pętla która dla każdego wiersza z głównego SELECTa wykonuje zadanie z LATERAL JOINa.

```sql
SELECT * FROM foo, LATERAL (SELECT * FROM bar WHERE bar.id = foo.bar_id) ss;
```

Jak to by wyglądało w pythonie:

```python
for row_f in foo:
    for row_b in bar:
        print(row_f,row_b)
```


### Przykład nr. 1

W pierwszym przykładzie wygenerujemy kolejne daty dla przedziału czasowego w koszykach.

Przygotujmy dane:

```sql
CREATE TABLE events (
    id int8 GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    event_start DATE NOT NULL,
    event_end DATE NOT NULL,
    CHECK (event_end >= event_start)
);
```

```sql
WITH event_starts AS (
    SELECT now() - '2 weeks'::INTERVAL * random() AS START
    FROM generate_series(1,5) i
)
INSERT INTO events (event_start, event_end)
SELECT
    START,
    START + '3 days'::INTERVAL + random() * '4 days'::INTERVAL
FROM
    event_starts;
```

```sql
SELECT * FROM events;
 id | event_start | event_end  
----+-------------+------------
  1 | 2022-09-15  | 2022-09-22
  2 | 2022-09-06  | 2022-09-11
  3 | 2022-09-06  | 2022-09-10
  4 | 2022-09-12  | 2022-09-18
  5 | 2022-09-05  | 2022-09-10
```

Stworzyliśmy tabelę z 5 koszykami. Aby wgenerować kolejne daty z koszyka możemy wykorzystać funkcję lateral join.

```sql
SELECT
    e.*,
    l.*
FROM
    events e,
    lateral (
        SELECT x::DATE
        FROM generate_series(e.event_start, e.event_end, '1 day'::INTERVAL) AS x
    ) AS l


id | event_start | event_end  |     x      
----+-------------+------------+------------
  1 | 2022-09-15  | 2022-09-22 | 2022-09-15
  1 | 2022-09-15  | 2022-09-22 | 2022-09-16
  1 | 2022-09-15  | 2022-09-22 | 2022-09-17
  1 | 2022-09-15  | 2022-09-22 | 2022-09-18
  1 | 2022-09-15  | 2022-09-22 | 2022-09-19
  1 | 2022-09-15  | 2022-09-22 | 2022-09-20
  1 | 2022-09-15  | 2022-09-22 | 2022-09-21
  1 | 2022-09-15  | 2022-09-22 | 2022-09-22
  2 | 2022-09-06  | 2022-09-11 | 2022-09-06
  2 | 2022-09-06  | 2022-09-11 | 2022-09-07
  2 | 2022-09-06  | 2022-09-11 | 2022-09-08
  2 | 2022-09-06  | 2022-09-11 | 2022-09-09
  2 | 2022-09-06  | 2022-09-11 | 2022-09-10
  2 | 2022-09-06  | 2022-09-11 | 2022-09-11
  3 | 2022-09-06  | 2022-09-10 | 2022-09-06
  3 | 2022-09-06  | 2022-09-10 | 2022-09-07
  3 | 2022-09-06  | 2022-09-10 | 2022-09-08
  3 | 2022-09-06  | 2022-09-10 | 2022-09-09
  3 | 2022-09-06  | 2022-09-10 | 2022-09-10
  4 | 2022-09-12  | 2022-09-18 | 2022-09-12
  4 | 2022-09-12  | 2022-09-18 | 2022-09-13
  4 | 2022-09-12  | 2022-09-18 | 2022-09-14
  4 | 2022-09-12  | 2022-09-18 | 2022-09-15
  4 | 2022-09-12  | 2022-09-18 | 2022-09-16
  4 | 2022-09-12  | 2022-09-18 | 2022-09-17
  4 | 2022-09-12  | 2022-09-18 | 2022-09-18
  5 | 2022-09-05  | 2022-09-10 | 2022-09-05
  5 | 2022-09-05  | 2022-09-10 | 2022-09-06
  5 | 2022-09-05  | 2022-09-10 | 2022-09-07
  5 | 2022-09-05  | 2022-09-10 | 2022-09-08
  5 | 2022-09-05  | 2022-09-10 | 2022-09-09
  5 | 2022-09-05  | 2022-09-10 | 2022-09-10
```
