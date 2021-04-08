# SQL-fundamentals
Documentation of my Journey working with basic Sqlite via RSqlite in markdown. Projects get more detailed as they advance.

## Project One: Analyzing CIA factbook database
Load Packages and establish DB connection

``` r
library(RSQLite)
con<-dbConnect(SQLite(), "./factbook.db")
```

``` sql
--Preview the tables available
SELECT *
FROM sqlite_master
WHERE type='table';
```

2 records

</div>

``` sql
--First 5 rows from facts table
SELECT *
FROM facts
LIMIT 5;
```

<div class="knitsql-table">

| id | code | name        |    area | area\_land | area\_water | population | population\_growth | birth\_rate | death\_rate | migration\_rate | created\_at                | updated\_at                |
| :- | :--- | :---------- | ------: | ---------: | ----------: | ---------: | -----------------: | ----------: | ----------: | --------------: | :------------------------- | :------------------------- |
| 1  | af   | Afghanistan |  652230 |     652230 |           0 |   32564342 |               2.32 |       38.57 |       13.89 |            1.51 | 2015-11-01 13:19:49.461734 | 2015-11-01 13:19:49.461734 |
| 2  | al   | Albania     |   28748 |      27398 |        1350 |    3029278 |               0.30 |       12.92 |        6.58 |            3.30 | 2015-11-01 13:19:54.431082 | 2015-11-01 13:19:54.431082 |
| 3  | ag   | Algeria     | 2381741 |    2381741 |           0 |   39542166 |               1.84 |       23.67 |        4.31 |            0.92 | 2015-11-01 13:19:59.961286 | 2015-11-01 13:19:59.961286 |
| 4  | an   | Andorra     |     468 |        468 |           0 |      85580 |               0.12 |        8.13 |        6.96 |            0.00 | 2015-11-01 13:20:03.659945 | 2015-11-01 13:20:03.659945 |
| 5  | ao   | Angola      | 1246700 |    1246700 |           0 |   19625353 |               2.78 |       38.78 |       11.49 |            0.46 | 2015-11-01 13:20:08.625072 | 2015-11-01 13:20:08.625072 |

Displaying records 1 - 5

</div>

### **Min and Max populations states**

``` sql
SELECT MIN(population) as min_pop, MAX(population) as max_pop,
MIN(population_growth) as min_pop_g, MAX(population_growth) as max_pop_g
FROM facts;
```

<div class="knitsql-table">

| min\_pop |   max\_pop | min\_pop\_g | max\_pop\_g |
| -------: | ---------: | ----------: | ----------: |
|        0 | 7256490011 |           0 |        4.02 |

1 records

</div>

**Full solution workbook can be viewed [here](https://github.com/rickyboshe/SQL-fundamentals/blob/main/Guided-project.md)**
