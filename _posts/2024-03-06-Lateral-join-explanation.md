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

Taki 
