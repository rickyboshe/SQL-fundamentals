Analyzing CIA Factbook
================
Fredrick Boshe
08/04/2021

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

<div class="knitsql-table">

| type  | name             | tbl\_name        | rootpage | sql                                                                                                                                                                                                                                                                                                                                                                  |
| :---- | :--------------- | :--------------- | -------: | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| table | facts            | facts            |        2 | CREATE TABLE “facts” (“id” INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL, “code” varchar(255) NOT NULL, “name” varchar(255) NOT NULL, “area” integer, “area\_land” integer, “area\_water” integer, “population” integer, “population\_growth” float, “birth\_rate” float, “death\_rate” float, “migration\_rate” float, “created\_at” datetime, “updated\_at” datetime) |
| table | sqlite\_sequence | sqlite\_sequence |        3 | CREATE TABLE sqlite\_sequence(name,seq)                                                                                                                                                                                                                                                                                                                              |

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

### **Country with the minimum population**

``` sql
SELECT *
FROM facts
WHERE population ==(SELECT MIN(population)
                    FROM facts);
```

<div class="knitsql-table">

|  id | code | name       | area | area\_land | area\_water | population | population\_growth | birth\_rate | death\_rate | migration\_rate | created\_at                | updated\_at                |
| --: | :--- | :--------- | ---: | ---------: | ----------: | ---------: | -----------------: | ----------: | ----------: | --------------: | :------------------------- | :------------------------- |
| 250 | ay   | Antarctica |   NA |     280000 |          NA |          0 |                 NA |          NA |          NA |              NA | 2015-11-01 13:38:44.885746 | 2015-11-01 13:38:44.885746 |

1 records

</div>

### **Country with the maximum population**

``` sql
SELECT *
FROM facts
WHERE population ==(SELECT MAX(population)
                    FROM facts);
```

<div class="knitsql-table">

|  id | code | name  | area | area\_land | area\_water | population | population\_growth | birth\_rate | death\_rate | migration\_rate | created\_at                | updated\_at                |
| --: | :--- | :---- | ---: | ---------: | ----------: | ---------: | -----------------: | ----------: | ----------: | --------------: | :------------------------- | :------------------------- |
| 261 | xx   | World |   NA |         NA |          NA | 7256490011 |               1.08 |        18.6 |         7.8 |              NA | 2015-11-01 13:39:09.910721 | 2015-11-01 13:39:09.910721 |

1 records

</div>

### **Min and Max populations states (excluding ‘World’)**

``` sql
SELECT MIN(population) as min_pop, MAX(population) as max_pop,
MIN(population_growth) as min_pop_g, MAX(population_growth) as max_pop_g
FROM facts
WHERE name != "World";
```

<div class="knitsql-table">

| min\_pop |   max\_pop | min\_pop\_g | max\_pop\_g |
| -------: | ---------: | ----------: | ----------: |
|        0 | 1367485388 |           0 |        4.02 |

1 records

</div>

### **Average value of Population and Area (excluding ‘World’)**

``` sql
SELECT AVG(population), AVG(area)
FROM facts
WHERE name != "World";
```

<div class="knitsql-table">

| AVG(population) | AVG(area) |
| --------------: | --------: |
|        32242667 |  555093.5 |

1 records

</div>

### **Countries with an above average population and below average area**

``` sql
SELECT*
FROM facts
WHERE population > (SELECT AVG(population)
                    FROM facts
                    WHERE name != "World")
AND area < (SELECT AVG(area)
            FROM facts
            WHERE name != "World");
```

<div class="knitsql-table">

| id | code | name       |   area | area\_land | area\_water | population | population\_growth | birth\_rate | death\_rate | migration\_rate | created\_at                | updated\_at                |
| -: | :--- | :--------- | -----: | ---------: | ----------: | ---------: | -----------------: | ----------: | ----------: | --------------: | :------------------------- | :------------------------- |
| 14 | bg   | Bangladesh | 148460 |     130170 |       18290 |  168957745 |               1.60 |       21.14 |        5.61 |            0.46 | 2015-11-01 13:20:52.753843 | 2015-11-01 13:20:52.753843 |
| 65 | gm   | Germany    | 357022 |     348672 |        8350 |   80854408 |               0.17 |        8.47 |       11.42 |            1.24 | 2015-11-01 13:25:21.942190 | 2015-11-01 13:25:21.942190 |
| 80 | iz   | Iraq       | 438317 |     437367 |         950 |   37056169 |               2.93 |       31.45 |        3.77 |            1.62 | 2015-11-01 13:26:41.627918 | 2015-11-01 13:26:41.627918 |
| 83 | it   | Italy      | 301340 |     294140 |        7200 |   61855120 |               0.27 |        8.74 |       10.19 |            4.10 | 2015-11-01 13:26:58.014646 | 2015-11-01 13:26:58.014646 |
| 85 | ja   | Japan      | 377915 |     364485 |       13430 |  126919659 |               0.16 |        7.93 |        9.51 |            0.00 | 2015-11-01 13:27:08.040081 | 2015-11-01 13:27:08.040081 |

Displaying records 1 - 5

</div>

### **Countries with the most people**

``` sql
SELECT*, MAX(population)
FROM facts
WHERE name != "World";
```

<div class="knitsql-table">

| id | code | name  |    area | area\_land | area\_water | population | population\_growth | birth\_rate | death\_rate | migration\_rate | created\_at                | updated\_at                | MAX(population) |
| -: | :--- | :---- | ------: | ---------: | ----------: | ---------: | -----------------: | ----------: | ----------: | --------------: | :------------------------- | :------------------------- | --------------: |
| 37 | ch   | China | 9596960 |    9326410 |      270550 | 1367485388 |               0.45 |       12.49 |        7.53 |            0.44 | 2015-11-01 13:22:53.813142 | 2015-11-01 13:22:53.813142 |      1367485388 |

1 records

</div>
