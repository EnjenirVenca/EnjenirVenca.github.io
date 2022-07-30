---
layout: page
title: SQL
permalink: docs/programovani/
parent: Programování/Software
toc: true
---

# SQL (PostgreSQL)

## Co je vlastně SQL?

- jazyk, který slouží k psaní strukturovaných dotazů (structured query language) = ke komunikaci s databází

## Co je MySQL?

- jeden z prvních open-source databázových systémů
- udržuje data strukturovaná v databázích a existuje několik dalších systémů pro spravování databází (PostgreSQL, Oracle, Microsoft SQL Server a další)
- každá z těchto databází pracuje s určitou verzí jazyka SQL, syntax zůstává stejná, ale mění se funkce a možnosti taky na základě toho jaký je to druh databáze (čistě relační nebo i objektově orientovaná)
- každý z databázových systémů ukládá data trochu jinak

## SQL operace - pořadí provedení

- SELECT, FROM, JOIN, (ON), WHERE, GROUP BY, HAVING, ORDER BY, LIMIT

## SELECT

- většinou se skládá z jednoduchých SELECT a FROM zakončený středníkem
- potom se dá ale ještě konkretizovat klauzolema jako:
  - **WHERE** - for filtering data based on a specific condiditon
  - **ORDER BY** - for sorting the result set
  - **LIMIT** - for limiting rows returned
  - **JOIN** - for querying data from multiple related tables
  - **GROUP BY** - for grouping data based on one or more columns
  - **HAVING** - for filtering groups
- když používám hvězdičku *, vybere to všechny sloupce z celé tabulky
- SELECT se dá taky používat pro jednoduché výpočty

```sql
SELECT
	price * quantity AS revenue
FROM
	sales
```

## ORDER BY

```sql
SELECT
	column1, column2
FROM
	table_name
ORDER BY column1 ASC,
		 column2 DESC;
```

- když použiju 2 a více "pravidel", tak se to nejdřív seřadí podle prvního sloupečku a pak těch dalších...

## SELECT DISTINCT

- když za distinct použiji jen jeden sloupeček, tak query podle toho sloupečku vyřadí duplikáty

- když jich dám víc – např. 2 tak to vlastně vyřadí jen duplikáty, kde je to stejné v obou sloupečkách a analogicky i při více sloupečkách

- slouží k výběru bez opakování duplikátů

## LIMIT a OFFSET

| employee_id | first_name | last_name |
| ----------- | ---------- | --------- |
| 121         | Adam       | Fripp     |
| 103         | Alexander  | Hunold    |
| 115         | Alexander  | Khoo      |
| 193         | Britney    | Everett   |
| 104         | Bruce      | Ernst     |
| 179         | Charles    | Johnson   |
| 109         | Daniel     | Faviet    |
| 105         | David      | Austin    |

```sql
SELECT
	employee_id, first_name, last_name
FROM
	employees
ORDER BY first_name
LIMIT 5 OFFSET 3;
```

- tabulka ukáže vybrané data, ale z tabulky posunuté 3 odshora (OFFSET) a jen 5 položek (LIMIT), funguje i pokud napíšu obráceně

| employee_id | first_name | last_name |
| ----------- | ---------- | --------- |
| 193         | Britney    | Everett   |
| 104         | Bruce      | Ernst     |
| 179         | Charles    | Johnson   |
| 109         | Daniel     | Faviet    |
| 105         | David      | Austin    |

## FETCH

- používá se, když chci přeskočit N řádků odkud pak query začne

## WHERE

- slouží k filtraci výběru ze SELECTu
- definuji, že hledám něco KDE (WHERE), je splněna nějaká mnou definovaná podmínka

### Porovnávací operátory

| operator | meaning               |
| -------- | --------------------- |
| =        | equal to              |
| <> (!=)  | not equal to          |
| <        | less than             |
| >        | greater than          |
| <=       | less than or equal    |
| >=       | greater than or equal |

- pomocí operátorů můžu specifikovat co má SELECT vybrat

### Logické operátory

| operator | meaning                                                      |
| -------- | ------------------------------------------------------------ |
| ALL      | return true if all comparisons are true                      |
| AND      | return true if both expressions are true                     |
| ANY      | return true if any one of the comparisons is true            |
| BETWEEN  | return true if the operand is within a range                 |
| EXISTS   | return true if a subquery contains any rows                  |
| IN       | return true if the operand is equal to one of the value in a list |
| LIKE     | return true if the operand matches a pattern                 |
| NOT      | reverse the result of any other Boolean operator             |
| OR       | return true if either expression is true                     |
| SOME     | return true if some of the expressions are true              |
| IS NULL  | return only null rows                                        |

- SOME = ANY

## CASE

- slouží k vytvoření kondicionálu
- přidávám WHEN, THEN, END 

```sql
SELECT
	first_name,
	last_name,
	hire_date,
	CASE (2000 - YEAR(hire_date))
		WHEN 1 THEN '1 year'
		WHEN 3 THEN '3 years'
		WHEN 5 THEN '5 years'
		WHEN 10 THEN '10 years'
		WHEN 15 THEN '15 years'
		WHEN 20 THEN '20 years'
		WHEN 25 THEN '25 years'
		WHEN 30 THEN '30 years'
	END aniversary
FROM
	employees
ORDER BY first_name;
```

## ALIAS (AS)

- umožňuje dát alias tabulce nebo sloupečku

```sql
SELECT
	inv_no AS invoice_number,
	amount,
	due_date AS 'Due date',
	cust_no 'Customer No'
FROM
	invoices i;
```

- alias se může používat vždycky až potom co se určí v SELECTu

```sql
SELECT
	d.department_name
FROM
	deparments AS d
```

## WITH

- s WITH jsem schopný vytvářet "temporary tabulky" neboli CTE (common table expression), které pak můžu v query dále používat

```sql
WITH regional_sales AS (
		SELECT region,
    	   	SUM (amount) AS total_sales
    	FROM orders
    	GROUP BY region
),
	top_regions AS (
		SELECT region
    	FROM regional_sales
    	WHERE total_sales > (SELECT SUM(total_sales)/10 FROM regional_sales)
)
SELECT region,
	   product,
	   SUM(quantity) AS product_units,
	   SUM(amount) AS product_sales
FROM orders
WHERE region IN (SELECT region FROM top_regions)
GROUP BY region, product;
```

- tzn. když mám WITH regional_sales AS, tak jsem si vytvořil "temporary tabulky" s názvem regional_sales a to zatím AS ji definuje (nefunguje v tomto případě jako alias)

## LEAD

- is used to access a row that follows the current row, at a specific physical offset and is generally used for comparing the value of the current row with the value of the next row following the current row (většinou within the partition?!)
- slouží k porovnávání řádku s jiným řádkem (většinou s tím co je pod námi vybraným)

## PARTITION BY

- divides rows into partitions, by default, it takes the query result as a single partition (partition je malá část nějaké větší databáze)

- rozdělí si to podle nějakých parametrů tabulky (uspořádá)

- vlastně když udělám *„SUM(delta) OVER (PARTITION BY inventory_id, goods_id) AS delta_total“* tak to jede nejdřív podle inventory a pak podle goods

## OVER

- used to determine which rows from the query are applied to the function, what order they are evaluated in by that function, and when the function’s calculations should restart

```sql
SELECT
	year,
	amount,
	group_id,
	LEAD(amount, 1) OVER (
    	PARTITION BY group_id
      ORDER BY year) next_year_sales
FROM
	sales;
```

|      | year | amount  | group_id | next_year_sales |
| ---- | ---- | ------- | -------- | --------------- |
| 1    | 2018 | 1474.00 | 1        | 1915.00         |
| 2    | 2019 | 1915.00 | 1        | 1646.00         |
| 3    | 2020 | 1646.00 | 1        | [null]          |
| 4    | 2018 | 1787.00 | 2        | 1911.00         |
| 5    | 2019 | 1911.00 | 2        | 1975.00         |
| 6    | 2020 | 1975.00 | 2        | [null]          |
| 7    | 2018 | 1760.00 | 3        | 1118.00         |
| 8    | 2019 | 1118.00 | 3        | 1516.00         |
| 9    | 2020 | 1516.00 | 3        | [null]          |

- in this example:
  - the **PARTITION BY** clause distributes rows into product groups (or partitions) specified by group id
  - the **ORDER BY** clause sorts rows in each product group by years in ascending order
  - the **LEAD()** function returns the sales of the next year from the current year for each product group

## JOIN

<img src="programovani_software/sql/assets/images/join.png"/>

- INNER JOIN = JOIN
- SELF JOIN je normální JOIN, jen se spojuje ta tabulka sama se sebou
- JOIN ON (používám, když potřebuju specifikovat ID) vs. JOIN USING (používám, pokud mají stejný název sloupce podle kterého chci joinovat)
- EQUI JOIN využívá WHERE =; nebo JOIN ON (typ INNER JOINu)

```sql
SELECT column_list
FROM table1, table2...
WHERE table1.column_name = table2.column_name;

-----------------------------------NEBO----------------------------------

SELECT *
FROM table1
JOIN table2
[ON (join_condition)]
```

- NON EQUI JOIN

  - uses comparison operator instead of the equal sign like >, <, >=, <= along with conditions

- NATURAL JOIN

  - je to typ equi joinu
  - v podstatě to spojí řádky, které mají stejné ID a jsou tam právě jednou
  - jako INNER JOIN, ale nemusím specifikovat, dle jakého id se mají napojit

  <img src="programovani_software/sql/assets/images/natural_join.png"/>

## Agregační funkce - AVG, COUNT, MAX, MIN, SUM

- agregační funkce:
  - **AVG()** - returns the average of a set
  - **COUNT()** - returns the number of items in set
  - **MAX()** - returns the maximum value in a set
  - **MIN()** - returns the minimum value in a set
  - **SUM()** - returns the sum of al or distinct values in a set
- až na COUNT, všechny funkce ignorují NULL
- agregační funkce se většinou používají s GROUP BY
- můžeme zde taky použít ROUND, které zaokrouhluje na námi určený počet míst

```sql
SELECT
	ROUND(price, 2)
FROM
	products;
```

## GROUP BY

- ke spojení více řádků dohromady
- pokud více řádků obsahuje NULL, tak jsou spojeny k sobě, jelikož GROUP BY je vidí jako stejné hodnoty

```sql
SELECT
	manager_id,
	first_name,
	last_name,
	COUNT(employee_id) direct_reports
FROM
	employees
WHERE
	manager_id IS NOT NULL
GROUP BY manager_id
HAVING direct_reports >= 5;
```

## HAVING

- slouží k filtrování skupin z GROUP BY nebo jako WHERE u agregačních funkcí
- zatímco WHERE slouží k filtrování jednotlivých řádků tabulky

<h3><font color="red">Grouping sets</font></h3>

- tomu moc nerozumím, ale asi to v podstatě spojuje několik GROUP BY dohromady bez toho, aniž by se musel používat UNION ALL, protože tomu je pak v kódu lépe rozumět a nevytíží to tolik databázový systém

## ROLLUP

- v podstatě nástavba GROUP BY
- umožňuje přidat "subtotals" řádky (řádek "celkem")

```sql
SELECT
	warehouse, SUM(quantity)
FROM
	inventory
GROUP BY
	warehouse;
```

| warehouse     | SUM(quantity) |
| ------------- | ------------- |
| San Francisco | 560           |
| San Jose      | 650           |

```sql
SELECT
	warehouse, SUM(quantity)
FROM
	inventory
GROUP BY ROLLUP
	warehouse;
```

| warehouse     | SUM(quantity) |
| ------------- | ------------- |
| San Francisco | 560           |
| San Jose      | 650           |
| [null]        | 1210          |

```sql
SELECT
	warehouse, product, SUM(quantity)
FROM
	inventory
GROUP BY
	warehouse, product;
```

| warehouse     | product | SUM(quantity) |
| ------------- | ------- | ------------- |
| San Jose      | iPhone  | 300           |
| San Francisco | iPhone  | 260           |
| San Jose      | Samsung | 350           |
| San Francisco | Samsung | 300           |

```sql
SELECT
	warehouse, product, SUM(quantity)
FROM
	inventory
GROUP BY ROLLUP
	warehouse, product;
```

| warehouse     | product | SUM(quantity) |
| ------------- | ------- | ------------- |
| San Francisco | iPhone  | 260           |
| San Francisco | Samsung | 300           |
| San Francisco | [null]  | 560           |
| San Jose      | iPhone  | 300           |
| San Jose      | Samsung | 350           |
| San Jose      | [null]  | 650           |
| [null]        | [null]  | 1210          |

```sql
SELECT
	COALESCE(warehouse, 'All warehouses') AS warehouse,
	SUM(quantity)
FROM
	inventory
GROUP BY ROLLUP
	warehouse;
```

| warehouse      | SUM(quantity) |
| -------------- | ------------- |
| San Francisco  | 560           |
| San Jose       | 650           |
| All warehouses | 1210          |

## COALESCE

- můžeme použít například k substituci NULL hodnot, ale má i další použití
- funkce, která vrací první ne NULL honotu

```sql
SELECT COALESCE(NULL, 1, 2, 'Ahoj');
```

- tato query by vrátila hodnotu **1**

## CUBE

- podobně jako ROLLUP je CUBE nadstavba funkce GROUP BY, umožňuje vytvořit řádek „celkem“ ale i pro „subsekce“ a hezčí než s ROLLUP

```sql
SELECT
   warehouse,
   product,
   SUM(quantity)
FROM
   inventory
GROUP BY
   CUBE(warehouse,product)
ORDER BY
   warehouse,
   product;
```

| warehouse     | product | SUM(quantity) |
| ------------- | ------- | ------------- |
| San Francisco | Samsung | 300           |
| San Francisco | iPhone  | 260           |
| San Francisco | [null]  | 560           |
| San Jose      | Samsung | 350           |
| San Jose      | iPhone  | 300           |
| San Jose      | [null]  | 650           |
| [null]        | Samsung | 650           |
| [null]        | iPhone  | 560           |
| [null]        | [null]  | 1210          |

``` sql
SELECT
   COALESCE(warehouse, '...All Warehouses') warehouse,
  COALESCE(product, '...All Products') product,
   SUM(quantity) 
FROM
   inventory
GROUP BY
   CUBE(warehouse,product)
ORDER BY
   warehouse,
   product; 
```

| warehouse         | product         | SUM(quantity) |
| ----------------- | --------------- | ------------- |
| ...All Warehouses | ...All Products | 1210          |
| ...All Warehouses | Samsung         | 650           |
| ...All Warehouses | iPhone          | 560           |
| San Francisco     | ...All Products | 560           |
| San Francisco     | Samsung         | 300           |
| San Francisco     | iPhone          | 260           |
| San Jose          | ...All Products | 650           |
| San Jose          | Samsung         | 350           |
| San Jose          | iPhone          | 300           |

## UNION & UNION ALL

- UNION slouží ke spojení více SELECT query v jednu
- pro zanechání duplikátů se používá UNION ALL viz. obrázek

<img src="programovani_software/sql/assets/assets/images/union.png"/>

- rozdílem od JOIN, je že JOIN spojuje sloupečky, zatímco UNION spojuje řádky

## INTERSECT

- slouží k zobrazení „jen“ shodných hodnot ze SELECT query viz. obrázek

<img src="programovani_software/sql/assets/images/intersect.png"/>

## MINUS

- slouží k "odečtení" jedné sady výsledků od druhé

<img src="programovani_software/sql/assets/images/minus.png"/>

```sql
SELECT 
    employee_id
FROM
    employees
    
MINUS

SELECT 
    employee_id
FROM
    dependents;
```

## Subquery

- Subquery je v podstatě query, která je „schovaná“ v další query např. SELECT, INSERT, UPDATE nebo DELETE
- např. chci zjistit všechny zaměstnance s location_id = 1700
- můžu použít 2 nezávislé query:


```sql
SELECT 
    *
FROM
    departments
WHERE
    location_id = 1700;
```

| department_id | department_name | location_id |
| ------------- | --------------- | ----------- |
| 1             | Administration  | 1700        |
| 3             | Purchasing      | 1700        |
| 9             | Executive       | 1700        |
| 10            | Finance         | 1700        |
| 11            | Accounting      | 1700        |

```sql
SELECT 
    employee_id, first_name, last_name
FROM
    employees
WHERE
    department_id IN (1, 3, 9, 10, 11)
ORDER BY first_name, last_name;
```

| employee_id | first_name | last_name |
| ----------- | ---------- | --------- |
| 115         | Alexander  | Khoo      |
| 179         | Charles    | Johnson   |
| 109         | Daniel     | Faviet    |
| 114         | Den        | Raphaely  |
| 118         | Guy        | Himuro    |

  - nebo subquery:

```sql
SELECT 
    employee_id, first_name, last_name
FROM
    employees
WHERE
    department_id IN (SELECT 
            department_id
        FROM
            departments
        WHERE
            location_id = 1700)
ORDER BY first_name, last_name;
```

- použití IN je zde pro porovnání hodnot ze selectu v hlavní query se selectem v subquery (opakem by bylo použití NOT IN)

- dají se použít i s porovnávacím operátorem (=, >, < atd.) nebo EXISTS, NOT EXISTS, ALL a ANY (SOME)

## Corelated subquery

### EXISTS

- slouží k vyběru, kdy v subquery určíme podmínku (např. i z jiné tabulky) a pak vybíráme řádky, které tuto podmínku splňují

```sql
SELECT 
    employee_id, first_name, last_name
FROM
    employees
WHERE
    EXISTS( SELECT 
            1
        FROM
            dependents
        WHERE
            dependents.employee_id = employees.employee_id);
```

### NOT EXISTS

- analogický protějšek k EXISTS

```sql
SELECT 
    employee_id, first_name, last_name
FROM
    employees
WHERE
    NOT EXISTS( SELECT 
            1
        FROM
            dependents
        WHERE
            dependents.employee_id = employees.employee_id);
```

## Recursive query

- the WITH RECURSIVE clause allows a query to refer to its own output

```sql
WITH RECURSIVE query_name (column_name, ...) AS (
	SELECT ...
    FROM query_name -- <= note the self-reference here
) [, ...]
SELECT ...
FROM query_name
```

- example:

```sql
CREATE TABLE employees (
	employee_id serial PRIMARY KEY,
	full_name VARCHAR NOT NULL,
	manager_id INT
);

-------------------------------------------------------------------------

INSERT INTO employees (
	employee_id,
	full_name,
	manager_id
)
VALUES
	(1, 'Michael North', NULL),
	(2, 'Megan Berry', 1),
	(3, 'Sarah Berry', 1),
	(4, 'Zoe Black', 1),
	(5, 'Tim James', 1),
	(6, 'Bella Tucker', 2),
	(7, 'Ryan Metcalfe', 2),
	(8, 'Max Mills', 2),
	(9, 'Benjamin Glover', 2),
	(10, 'Carolyn Henderson', 3),
	(11, 'Nicola Kelly', 3),
	(12, 'Alexandra Climo', 3),
	(13, 'Dominic King', 3),
	(14, 'Leonard Gray', 4),
	(15, 'Eric Rampling', 4),
	(16, 'Piers Paige', 7),
	(17, 'Ryan Henderson', 7),
	(18, 'Frank Tucker', 8),
	(19, 'Nathan Ferguson', 8),
	(20, 'Kevin Rampling', 8);
	
-------------------------------------------------------------------------

WITH RECURSIVE subordinates AS (
	SELECT
		employee_id,
		manager_id,
		full_name
	FROM
		employees
	WHERE
		employee_id = 2
	UNION
		SELECT
			e.employee_id,
			e.manager_id,
			e.full_name
		FROM
			employees e
		INNER JOIN subordinates s ON s.employee_id = e.manager_id
) SELECT
	*
FROM
	subordinates;
```

- the last query returns all subordinates of the manager with the id 2
- how it works:
  - the non-recursive term returns the base result set R0 that is the employee with the id 2


| employee_id | manager_id | full_name |
| ----------- | ---------- | --------- |
| 2         | 1    | Megan Berry   |

- the recursive term returns the direct subordinate(s) of the employee id 2. This is the result of joining between the employees table and the subordinates CTE. The first iteration of the recursive term returns the following result set:

| employee_id | manager_id | full_name |
| ----------- | ---------- | --------- |
| 6         | 2    | Bella Tucker  |
|           7 |          2 | Ryan Metcalfe
|           8 |          2 | Max Mills
|           9 |          2 | Benjamin Glover

- PostgreSQL executes the recursive term repeatedly. The second iteration of the recursive member uses the result set above step as the input value, and returns this result set:

| employee_id | manager_id | full_name |
| ----------- | ---------- | --------- |
|          16 |          7 | Piers Paige
|          17 |          7 | Ryan Henderson
|          18 |          8 | Frank Tucker
|          19 |          8 | Nathan Ferguson
|          20 |          8 | Kevin Rampling

- the third iteration returns an empty result set because there is no employee reporting to the employee with the id 16, 17, 18, 19, and 20
- PostgreSQL returns the final result set that is the union of all result sets in the first and second iterations generated by the non-recursive and recursive terms:

| employee_id | manager_id | full_name |
| ----------- | ---------- | --------- |
|           2 |          1 | Megan Berry
|           6 |          2 | Bella Tucker
|           7 |          2 | Ryan Metcalfe
|           8 |          2 | Max Mills
|           9 |          2 | Benjamin Glover
|          16 |          7 | Piers Paige
|          17 |          7 | Ryan Henderson
|          18 |          8 | Frank Tucker
|          19 |          8 | Nathan Ferguson
|          20 |          8 | Kevin Rampling

## ALL

- logický operátor, který „vrací“ jednu hodnotu se setem jednoho sloupečku hodnot, které „vrací“ subquery

- podmínky:

  - **c > ALL()** - the values in column c must be greater than the biggest value in the set to evaluate to true

  - **c >= ALL()** - the values in column c must be greater than or equal to the biggest value in the se to evaluate to true

  - **c < ALL()** - the values in column c must be less than the lowest value in the set to evaluate to true

  - **c <= ALL()** - the values in column c must be less than or equal to the lowest value in the se to evaluate to true

  - **c < > ALL()** - the values in column c must not be equal to any value in the set to evaluate to true
  - **c = ALL()** - the values in column c must be equal to any value in the set to evalute to true

```sql
SELECT 
    first_name, last_name, salary
FROM
    employees
WHERE
    salary > ALL (SELECT 
            salary
        FROM
            employees
        WHERE
            department_id = 2)
ORDER BY salary;
```

- u ALL bude ta podmínka splněna jen v případě, že všechny hodnoty v setu jí splňují

## ANY (SOME)

- stejné jako ALL, ale stačí, pokud jakákoliv hodnota v setu splňuje námi danou podmínku

- podmínky:

  - **x = ANY()** - the values in column c must match one or more values in the set to evaluate to true

  - **x != ANY()** - the values in column c must not match one or more values in the set to evaluate to true

  - **x > ANY()** - the values in column c must be grater than the smallest value in the se to evaluate to true

  - **x < ANY()** - the values in column c must be smaller than the biggest value in the set to evaluate to true

  - **x >= ANY()** - the values in column c must be greater than or equal to the smallest value in the set to evaluate to true
  - **x <= ANY()** - the values in column c must be smaller than or equal to the biggest value in the set to evaluate to true

```sql
SELECT 
    first_name, 
    last_name, 
    salary
FROM
    employees
WHERE
    salary = ANY (
        SELECT 
            AVG(salary)
        FROM
            employees
        GROUP BY 
            department_id)
ORDER BY 
    first_name, 
    last_name,
    salary;   
```

## INSERT

- slouží k vkládání jednotlivých dat do tabulek

```sql
INSERT INTO table1 (column1, column2,...)
VALUES
	(value1, value2,...);
	
-------------------------------------------------------------------------

INSERT INTO dependents (
	first_name,
	last_name,
	relationship,
	employee_id
)
VALUES
	(
		'Dustin',
		'Johnson',
		'Child',
		178
	);
	
-------------------------------------------------------------------------

INSERT INTO table1
VALUES
	(value1, value2,...),
	(value1, value2,...),
	(value1, value2,...),
	...;
	
-------------------------------------------------------------------------

INSERT INTO dependents (
	first_name,
	last_name,
	relationship,
	employee_id
)
VALUES
	(
		'Cameron',
		'Bell',
		'Child',
		192
	),
	(
		'Michelle',
		'Bell',
		'Child',
		192
	);
```

## UPDATE

- slouží k "aktualizaci" existujících dat v tabulce

```sql
UPDATE table_name
SET column1 = value1,
 column2 = value2
WHERE
	condition;
	
-------------------------------------------------------------------------

UPDATE employees 
SET 
    last_name = 'Lopez'
WHERE
    employee_id = 192;
    
-------------------------------------------------------------------------

SELECT
	*
FROM
	dependents
WHERE
	employee_id = 192;
	
-------------------------------------------------------------------------
    
UPDATE dependents 
SET 
    last_name = 'Lopez'
WHERE
    employee_id = 192;    
   
-------------------------------------------------------------------------
    
UPDATE dependents
SET last_name = (
	SELECT
		last_name
	FROM
		employees
	WHERE
		employee_id = dependents.employee_id
);
```

## DELETE

- slouží k mazání tabulek tak i jednotlivých dat nebo položek z tabulek

```sql
DELETE FROM dependents 
WHERE
    dependent_id = 16;
    
-------------------------------------------------------------------------        
DELETE FROM dependents 
WHERE
    employee_id IN (100 , 101, 102);
```

## CREATE TABLE

- slouží k vytvoření tabulky v databázi

```sql
CREATE TABLE table_name(
     column_name_1 data_type default value column_constraint,
     column_name_2 data_type default value column_constraint,
     ...,
     table_constraint
);
```

- existují různé druhy data (data_type)

<img src="programovani_software/sql/assets/images/data_types.png"/>

```sql
CREATE TABLE trainings (
    employee_id INT,
    course_id INT,
    taken_date DATE,
    PRIMARY KEY (employee_id , course_id)
);
```

<h3><font color="red">DATA_TYPES</font></h3>

https://tableplus.com/blog/2018/06/postgresql-data-types.html

## ALTER TABLE

- slouží k úpravě existující tabulky, např. k přidání nebo přejmenování sloupečku atd.

```sql
ALTER TABLE table_name
ADD new_colum data_type column_constraint [AFTER existing_column];

-------------------------------------------------------------------------

ALTER TABLE table_name
MODIFY column_definition;

-------------------------------------------------------------------------

ALTER TABLE table_name
DROP column_name,
DROP colum_name,
...
```

## DROP TABLE

- slouží k odstranění tabulky z databáze

```sql
DROP TABLE [IF EXISTS] table_name;

-------------------------------------------------------------------------

DROP TABLE table_name1,table_name2,...;
```

## TRUNCATE TABLE

- slouží k odstranění všech dat z tabulky, ale nesmaže tabulku

```sql
TRUNCATE TABLE table_name;

-------------------------------------------------------------------------

TRUNCATE table_name;
```



### TRUNCATE vs. DELETE vs. DROP

- **DELETE** statement removes rows one at a time and records an entry in the transaction log for each deleted row
- **TRUNCATE TABLE** removes the data by deallocating the data pages used to store the table data and records only the page deallocations in the transaction log
- **DROP** removes the entire schema/structure of the table from the database

<h3><font color="red">PRIMARY KEY</font></h3>

https://www.postgresqltutorial.com/postgresql-primary-key/

<h3><font color="red">FOREIGN KEY</font></h3>

https://www.postgresqltutorial.com/postgresql-foreign-key/

<h3><font color="red">UNIQUE</font></h3>

https://www.postgresqltutorial.com/postgresql-unique-constraint/

<h3><font color="red">NOT NULL</font></h3>

https://www.postgresqltutorial.com/postgresql-not-null-constraint/

<h3><font color="red">CHECK</font></h3>

https://www.postgresqltutorial.com/postgresql-check-constraint/

## STRING_AGG

- agregátní funkce, která „spojuje“ seznam stringů a dává mezi ně nějaký separator (oddělovač) 

- většinou se tam pak ještě dává ORDER BY, podle kterého se seřadí ty expressions

```sql
STRING_AGG(expression, separator [ORDER BY ...])
```

## LIKE vs. ILIKE

- slouží k hledání ve stringu, podobné jako (LIKE), procento % nahrazuje jakýkoliv znak

```sql
SELECT
	first_name,
    last_name
FROM
	customer
WHERE
	first_name LIKE 'Jen%';
```

- ILIKE dělá to stejné jako LIKE, ale je case-insensitive (jedno jestli je to velkými nebo malými písmeny)

```sql
SELECT
	first_name,
	last_name
FROM
	customer
WHERE
	first_name ILIKE 'BAR%';
```

## IN

- operator in WHERE clause to check if a value matches any value in a list of values (analogicky funkce **NOT IN**)

```sql
SELECT customer_id,
	   rental_id,
	   return_date
FROM
	   rental
WHERE
	   customer_id IN (1, 2)
ORDER BY
	   return_date DESC;
```

## DATE

- CURRENT_DATE

```sql
SELECT NOW() :: DATE;

-------------------------------------------------------------------------

SELECT CURRENT_DATE;
```

- specific formatting

```sql
SELECT TO_CHAR(NOW() :: DARE, 'MON dd, yyyy');
```

- interval between dates (BETWEEN)

```sql
SELECT
	first_name,
	last_name,
	NOW() - hire_date AS diff
FROM
	employees;
	
-------------------------------------------------------------------------

SELECT
	first_name,
	last_name,
	date
FROM
	employees
WHERE date BETWEEN CURRENT_DATE AND CURRENT_DATE - INTERVAL '7 days'
```

## AGE

```sql
SELECT
	employee_id,
	first_name,
	last_name,
	AGE(birth_date)
FROM
	employees;
	
	
 employee_id | first_name | last_name |           age
-------------+------------+-----------+-------------------------
           1 | Shannon    | Freeman   | 36 years 5 mons 22 days
           2 | Sheila     | Wells     | 38 years 4 mons 18 days
           3 | Ethel      | Webb      | 41 years 5 mons 22 days
(3 rows)
```

## EXTRACT

- to get year, month, day as columns

```sql
SELECT
	employee_id,
	first_name,
	last_name,
	EXTRACT (YEAR FROM birth_date) AS YEAR,
	EXTRACT (MONTH FROM birth_date) AS MONTH,
	EXTRACT (DAY FROM birth_date) AS DAY
FROM
	employees;
	
 employee_id | first_name | last_name | year | month | day
-------------+------------+-----------+------+-------+-----
           1 | Shannon    | Freeman   | 1980 |     1 |   1
           2 | Sheila     | Wells     | 1978 |     2 |   5
           3 | Ethel      | Webb      | 1975 |     1 |   1
(3 rows)
```

## CONCAT_WS

- adds two or more expressions together with a separator

```sql
CONCAT_WS(separator, expression1, expression2, expression3,...)
```

<h3><font color="red">RANK</font></h3>

https://www.sqlshack.com/overview-of-sql-rank-functions/ 

<h3><font color="red">UNNEST</font></h3>

https://count.co/sql-resources/bigquery-standard-sql/unnest

## LOWER

- is used to convert a string from upper case to lower case

## TABLE vs. VIEW vs. MATERIALIZED VIEW

- **Table** is where data is stored. You always start with tables first, and then your usage pattern dictates whether you need views or materialized views.

- **View** is like a stored query for future use, if you're frequently joining or filtering the same tables the same way in multiple places.

- **Materialized view** is like a combination of both: it's a table that is automatically populated and refreshed via a view. You'd use this if you were using views, and want to pre-join or pre-aggregate the rows to speed up queries. à  it allows views to store data physically à materialized views that allow you to store the result of a query physically and update the data periodically

<h3><font color="red">SQL | DDL, DQL, DML, DCL and TCL Commands</font></h3>

<h3><font color="red">Použití *</font></h3>

- používat * lépe v SQL, např. chci vypsat všechny věci z tabulky goods ale budu joinovat nějakou jinou tabulku, která má podobně pojmenované sloupečky, tak pak můžu udělat např. g.*, c.name FROM goods g JOIN companies c....

<h3><font color="red">PERCENTILE_CONT</font></h3>

<h3><font color="red">PIVOT TABLES</font></h3>

- CROSSTAB