---
title: "Lateral join: Wyjaśnienie i prosty przykład"
#categories:
#  - Edge Case
#tags:
#  - content
#  - css
#  - edge case
#  - lists
#  - markup
---

Lateral join to jeden ze sposobów złoczeń w relacyjnych bazach danych. Wykorzystywany w PostgreSQL, Oracle, DB2  MS SQL. Jej znajomość jest przydane i warto się jej przyżeć. Może dzięki temu zaoczędzicie w przyszłości czas na szukanie rozwiązania lub zabłyśniecie na rozmowie rekrutacyjnej.


### Jak działa SELECT

Select działa jak pętla która zwraca weirsze z tabeli.


### Jak działa LATERAL

Jest to zagnieżdżona pętla która dla każdego wiersza z głównego SELECTa wykonuje zadanie z LATERAL JOINa.
