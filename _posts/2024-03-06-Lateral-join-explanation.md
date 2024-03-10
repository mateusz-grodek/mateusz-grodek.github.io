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
CREATE TABLE buckets (
    id int8 GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    CHECK (end_date >= start_date)
);
```

```sql
WITH bucket_starts AS (
    SELECT now() - '1 weeks'::INTERVAL * random() AS START
    FROM generate_series(1,5) i
)
INSERT INTO buckets (start_date, end_date)
SELECT
    START,
    START + '3 days'::INTERVAL + random() * '4 days'::INTERVAL
FROM
    bucket_starts;
```

```sql
SELECT * FROM buckets;
 id | start_date  | end_date  
----+-------------+------------
1	| 2024-03-10  | 2024-03-15
2	| 2024-03-09  | 2024-03-13
3	| 2024-03-10  | 2024-03-17
4	| 2024-03-06  | 2024-03-13
5	| 2024-03-08  | 2024-03-11
```

Stworzyliśmy tabelę z 5 koszykami. Aby wgenerować kolejne daty z koszyka możemy wykorzystać funkcję lateral join.

```sql
SELECT
    e.*,
    l.day_lateral
FROM
    buckets e,
    lateral (
        SELECT day_lateral::DATE
        FROM generate_series(e.start_date, e.end_date, '1 day'::INTERVAL) AS day_lateral
    ) AS l


id  | start_date  | end_date   |     day_lateral      
----+-------------+------------+------------
1	| 2024-03-10  |	2024-03-15 | 2024-03-10
1	| 2024-03-10  |	2024-03-15 | 2024-03-11
1	| 2024-03-10  |	2024-03-15 | 2024-03-12
1	| 2024-03-10  |	2024-03-15 | 2024-03-13
1	| 2024-03-10  |	2024-03-15 | 2024-03-14
1	| 2024-03-10  |	2024-03-15 | 2024-03-15
2	| 2024-03-09  |	2024-03-13 | 2024-03-09
2	| 2024-03-09  |	2024-03-13 | 2024-03-10
2	| 2024-03-09  |	2024-03-13 | 2024-03-11
2	| 2024-03-09  |	2024-03-13 | 2024-03-12
2	| 2024-03-09  |	2024-03-13 | 2024-03-13
3	| 2024-03-10  |	2024-03-17 | 2024-03-10
3	| 2024-03-10  |	2024-03-17 | 2024-03-11
3	| 2024-03-10  |	2024-03-17 | 2024-03-12
3	| 2024-03-10  |	2024-03-17 | 2024-03-13
3	| 2024-03-10  |	2024-03-17 | 2024-03-14
3	| 2024-03-10  |	2024-03-17 | 2024-03-15
3	| 2024-03-10  |	2024-03-17 | 2024-03-16
3	| 2024-03-10  |	2024-03-17 | 2024-03-17
4	| 2024-03-06  |	2024-03-13 | 2024-03-06
4	| 2024-03-06  |	2024-03-13 | 2024-03-07
4	| 2024-03-06  |	2024-03-13 | 2024-03-08
4	| 2024-03-06  |	2024-03-13 | 2024-03-09
4	| 2024-03-06  |	2024-03-13 | 2024-03-10
4	| 2024-03-06  |	2024-03-13 | 2024-03-11
4	| 2024-03-06  |	2024-03-13 | 2024-03-12
4	| 2024-03-06  |	2024-03-13 | 2024-03-13
5	| 2024-03-08  |	2024-03-11 | 2024-03-08
5	| 2024-03-08  |	2024-03-11 | 2024-03-09
5	| 2024-03-08  |	2024-03-11 | 2024-03-10
5	| 2024-03-08  |	2024-03-11 | 2024-03-11
```

Szybkie wyjaśnienie. Dla każdego wiersza z tabeli buckets generujemy dni pomiędzy start_date, a end_date. Dla pierwszego wiersza 2024-03-10 i 2024-03-15 będzie to 6 dni, które najpierw generujemy, następnie przypisują się do koszyka i sprawdzamy kolejny wiersz z tabeli bucket. Kolejny wiersz to 2024-03-09 i 2024-03-13, co daje nam 5 dni różnicy, zatem dla każdego koszyka LATERAL wygeneruje inną liczę wierszy.  Co ważne w naszym lateral subquery możemy się odniść do informacji z tabeli buckets w tym przypadku e.start_date i e.end_date `FROM generate_series(e.start_date, e.end_date, '1 day'::INTERVAL) AS day_lateral` w klasycznym podzapytaniu sypnęłoby błędem. 


### Przykład nr. 2

Stworzymy 2 tabelki. W jednej będzie nazwa użytkownika i dostępne środki na koncie. W drugiej tabelce będą produky z ich ceną. Naszym zdaniem będzie przypisanie każdemu użytkownikowi top 3 najdroższe produkty na które go stać. 

Najpierw przykładowe dane:

```sql
CREATE TABLE products_table AS
    SELECT   id,
             id * 10 * random() AS price,
             'product ' || id AS product
    FROM generate_series(1, 1000) AS id;
```

```sql
CREATE TABLE users_table
(
    id                 int,
    username           text,
    cash               numeric
);
 
INSERT INTO users_table VALUES
    (1, 'Jurek', '502'),
    (2, 'Kamil', '2310'),
    (3, 'Czaro', '65')
```

```sql
SELECT  * FROM products_table LIMIT 7;
id  | price       | product    |    
----+-------------+------------+
1	| 1.3157565077| product 1  |
2	| 1.7794870667| product 2  |
3	| 14.367369866| product 3  |
4	| 7.9555639835| product 4  |
5	| 12.231595441| product 5  |
6	| 23.543000974| product 6  |
7	| 13.834326571| product 7  |
					 		 
```

```sql
SELECT  * FROM  users_table;
id  | user       | cash    |    
----+------------+-------- +
1	| Jurek	     |  502    |
2	| Kamil	     |  2310   |
3	| Czaro	     |  65     |
```

Teraz przykad jakie kroki musimy wykonać aby uzyskać pożądany wynik. Użyjemy takiego pseudopythona:

```python
for user in users_table:
    for product in products_table.orderby(price,desc): #Zakładamy że tak wyglądała by posortowana tabela z produktami
           if products_table.price <= users_table.cash:
                found+=1
                print(user, product)
                if found >3:
                    break
           else:
               jump to next product
           end
```

I teraz jak zaimplementować to z użyciem LATERALa:

```sql
SELECT        *
FROM      users_table AS u,
    LATERAL  (SELECT      *
        FROM       products_table  AS p
        WHERE       p.price < u.cash
        ORDER BY p.price DESC
        LIMIT 3
       ) AS x
ORDER BY u.id, price DESC;

id  | username   | cash    |  productt_id |  price    |   
----+------------+-------- +
1	| Jurek	     |   502   | product 741  |498.3935036|
1	| Jurek	     |   502   | product 143  |497.1360881|
1	| Jurek	     |   502   | product 162  |488.6783401|
2	| Kamil	     |   2310  | product 973  |2304.653878|
2	| Kamil	     |   2310  | product 275  |2304.328873|
2	| Kamil	     |   2310  | product 303  |2293.111048|
3	| Czaro	     |   65	   | product 10	  |62.28928108|
3	| Czaro	     |   65	   | product 86	  |59.72326317|
3	| Czaro	     |   65	   | product 593  |56.70062070|
```

Krótkie wyjaśnienie. Patrzymy na pierwszy wiersz z tabeli users_table. Mamy Jurka który ma na koncie 502zł, następnie z posortowanej po cenie malejąco listy produktów pokolei patrzymy czy produkt mieści się w budżecie. Jeśli nie to sprawdzamy kolejny. Jeśli jest ok to przpisujemy i lecimy dalej aż znajdziemy 3 produkty. Później przechodzimy do kolejnego usera. Tu poraz kolejny warto zauważyć że w podzapytaniu LATERAL porwónujemy kolumne p.price z zewnętrzą kolumną u.cash z tabeli user_table. W klasycznym JOIN to by nie przeszło :( .





