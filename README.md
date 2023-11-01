## Colin McKinley Portfolio Project 4
# Project: Deforestation Exploration

---

## Project Overview
You’re a data analyst for ForestQuery, a non-profit organization on a mission to reduce deforestation around the world and which raises awareness about this important environmental topic.

Your executive director and her leadership team members are looking to understand which countries and regions around the world seem to have forests that have been shrinking in size and also which countries and regions have the most significant forest area, both in terms of amount and percent of total area. The hope is that these findings can help inform initiatives, communications, and personnel allocation to achieve the largest impact with the precious few resources that the organization has at its disposal.

You’ve been able to find tables of data online dealing with forestation as well as total land area and region groupings, and you’ve brought these tables together into a database that you’d like to query to answer some of the most important questions in preparation for a meeting with the ForestQuery executive team coming up in a few days. Ahead of the meeting, you’d like to prepare and disseminate a report for the leadership team that uses complete sentences to help them understand the global deforestation overview between 1990 and 2016.

Project Instructions

You will be creating a report for the executive team in which you explain your results using complete sentences.

---

## Part 1 - Global Situation

### 1. Create a View called “forestation” by joining all three tables - forest_area, land_area, and regions in the workspace.

```sql
CREATE VIEW forestation
AS
SELECT fa.country_code, 
        fa.country_name, 
        fa.year, fa.forest_area_sqkm, 
        la.total_area_sq_mi, 
        r.region, r.income_group, 
        (fa.forest_area_sqkm / (la.total_area_sq_mi * 2.59) * 100) AS percent_designated_as_forest
FROM forest_area fa
JOIN land_area la
ON fa.country_code = la.country_code AND fa.year = la.year
RIGHT JOIN regions r
ON r.country_code = la.country_code;
```

### 1a. What was the total forest area (in sq km) of the world in 1990? Please keep in mind that you can use the country record denoted as “World" in the region table.

```sql
SELECT fa.country_code, 
        fa.country_name, 
        fa.year, fa.forest_area_sqkm
FROM forest_area fa
JOIN land_area la
ON fa.country_code = la.country_code AND fa.year = la.year
RIGHT JOIN regions r
ON r.country_code = la.country_code
WHERE fa.country_code = 'WLD' AND fa.year = 1990;
```
![](/assets/img/Output1a.png)


### 1b. What was the total forest area (in sq km) of the world in 2016? Please keep in mind that you can use the country record in the table is denoted as “World.

```sql
SELECT fa.country_code, 
      fa.country_name, 
      fa.year, 
      fa.forest_area_sqkm 
FROM forest_area fa
JOIN land_area la
ON fa.country_code = la.country_code AND fa.year = la.year
RIGHT JOIN regions r
ON r.country_code = la.country_code
WHERE fa.country_code = 'WLD' AND fa.year = 2016;
```
![](/assets/img/Output1b.png)


### 1c. What was the change (in sq km) in the forest area of the world from 1990 to 2016?

*** I used the created table (forestation) to create this query, as I needed to show a SELF JOIN query.
```sql
SELECT t1.a_country_code AS country_code, t1.a_country_name AS country_name, t1.b_forest_area_sqkm - t1.a_forest_area_sqkm AS forest_sqkm_change_1990_to_2016
FROM
(SELECT a.country_code AS a_country_code,
        a.country_name AS a_country_name,
        a.year AS a_year,
        a.forest_area_sqkm AS a_forest_area_sqkm,
        b.year AS b_year,
        b.forest_area_sqkm AS b_forest_area_sqkm
FROM forestation a
JOIN forestation b
ON a.country_name = b.country_name
AND b.forest_area_sqkm > a.forest_area_sqkm
AND b.year = 1990
WHERE a.country_code = 'WLD' AND (a.year = 1990 OR a.year = 2016)) t1;
```

### 1d. What was the percent change in forest area of the world between 1990 and 2016?

```sql
SELECT fa.country_code,
        fa.country_name,
        fa.year,
        fa.forest_area_sqkm, 
        LAG(fa.forest_area_sqkm) OVER (ORDER BY fa.year) AS lag,
        ((fa.forest_area_sqkm - lAG(fa.forest_area_sqkm) OVER (ORDER BY fa.year)) / lAG(fa.forest_area_sqkm) OVER (ORDER BY fa.year)) * 100 AS percent_change
FROM forest_area fa
JOIN land_area la
ON fa.country_code = la.country_code AND fa.year = la.year
RIGHT JOIN regions r
ON r.country_code = la.country_code
WHERE fa.country_code = 'WLD' AND (fa.year = 2016 OR fa.year = 1990);
```
![](/assets/img/Output1d.png)


### 1e. If you compare the amount of forest area lost between 1990 and 2016, to which country's total area in 2016 is it closest to?

```sql
SELECT t1.country_code, t1.country_name, t1.year, la.total_area_sq_mi * 2.59 AS total_area_sqkm
FROM(
  SELECT fa.country_code,
         fa.country_name,
         fa.year,
         fa.forest_area_sqkm, 
         LAG(forest_area_sqkm) OVER (ORDER BY forest_area_sqkm) AS lag,
         LAG(forest_area_sqkm) OVER (ORDER BY forest_area_sqkm) - forest_area_sqkm AS forest_sqkm_change
    FROM forest_area fa
    JOIN land_area la
    ON fa.country_code = la.country_code AND fa.year = la.year
    RIGHT JOIN regions r
    ON r.country_code = la.country_code
    WHERE fa.year = 2016
) t1
JOIN land_area la
ON t1.country_code = la.country_code AND t1.year = la.year
WHERE (la.total_area_sq_mi * 2.59) < (SELECT SUM(t2.forest_sqkm_change)
FROM(
  SELECT fa.forest_area_sqkm, 
         LAG(forest_area_sqkm) OVER (ORDER BY forest_area_sqkm DESC) AS lag,
         LAG(forest_area_sqkm) OVER (ORDER BY forest_area_sqkm DESC) - forest_area_sqkm AS forest_sqkm_change
    FROM forest_area fa
    JOIN land_area la
    ON fa.country_code = la.country_code AND fa.year = la.year
    RIGHT JOIN regions r
    ON r.country_code = la.country_code
WHERE fa.country_code = 'WLD')t2)
ORDER BY 4 DESC
LIMIT 1;
```
![](/assets/img/Output1e.png)

---

## 2. Regional Outlook

### 2a. What was the percent forest of the entire world in 2016? Which region had the HIGHEST percent forest in 2016, and which had the LOWEST, to 2 decimal places?

```sql
SELECT r.region,
    fa.year,
    TRUNC(((fa.forest_area_sqkm / (la.total_area_sq_mi * 2.59) * 100)::numeric), 2) AS percent_designated_as_forest
FROM forest_area fa
JOIN land_area la
ON fa.country_code = la.country_code AND fa.year = la.year
RIGHT JOIN regions r
ON r.country_code = la.country_code
WHERE fa.country_code = 'WLD' AND fa.year = 2016
GROUP BY 1, 2, 3;  
```
```sql
SELECT r.region, 
        fa.year, 
        TRUNC(((SUM(fa.forest_area_sqkm) / SUM(la.total_area_sq_mi * 2.59) * 100)::numeric), 2) AS percent_designated_as_forest
FROM forest_area fa
JOIN land_area la
ON fa.country_code = la.country_code AND fa.year = la.year
RIGHT JOIN regions r
ON r.country_code = la.country_code
WHERE fa.year = 2016 AND fa.country_code != 'WLD'
GROUP BY 1, 2
ORDER BY 3 DESC
LIMIT 1;
```
```sql
SELECT r.region, 
        fa.year, 
        TRUNC(((SUM(fa.forest_area_sqkm) / SUM(la.total_area_sq_mi * 2.59) * 100)::numeric), 2) AS percent_designated_as_forest
FROM forest_area fa
JOIN land_area la
ON fa.country_code = la.country_code AND fa.year = la.year
RIGHT JOIN regions r
ON r.country_code = la.country_code
WHERE fa.year = 2016 AND fa.country_code != 'WLD'
GROUP BY 1, 2
ORDER BY 3
LIMIT 1; 
```

### 2b. What was the percent forest of the entire world in 1990? Which region had the HIGHEST percent forest in 1990, and which had the LOWEST, to 2 decimal places?

```sql
SELECT r.region, 
        fa.year, 
        TRUNC(((fa.forest_area_sqkm / (la.total_area_sq_mi * 2.59) * 100)::numeric), 2) AS percent_designated_as_forest
FROM forest_area fa
JOIN land_area la
ON fa.country_code = la.country_code AND fa.year = la.year
RIGHT JOIN regions r
ON r.country_code = la.country_code
WHERE fa.country_code = 'WLD' AND fa.year = 1990
GROUP BY 1, 2, 3; 
```

```sql
SELECT r.region, 
        fa.year, 
        TRUNC(((SUM(fa.forest_area_sqkm) / SUM(la.total_area_sq_mi * 2.59) * 100)::numeric), 2) AS percent_designated_as_forest
FROM forest_area fa
JOIN land_area la
ON fa.country_code = la.country_code AND fa.year = la.year
RIGHT JOIN regions r
ON r.country_code = la.country_code
WHERE fa.year = 1990 AND fa.country_code != 'WLD'
GROUP BY 1, 2
ORDER BY 3 DESC
LIMIT 1;
```

```sql
SELECT r.region, 
        fa.year, 
        TRUNC(((SUM(fa.forest_area_sqkm) / SUM(la.total_area_sq_mi * 2.59) * 100)::numeric), 2) AS percent_designated_as_forest
FROM forest_area fa
JOIN land_area la
ON fa.country_code = la.country_code AND fa.year = la.year
RIGHT JOIN regions r
ON r.country_code = la.country_code
WHERE fa.year = 1990 AND fa.country_code != 'WLD'
GROUP BY 1, 2
ORDER BY 3
LIMIT 1;
```


### 2c. Based on the table you created, which regions of the world DECREASED in forest area from 1990 to 2016?

```sql
SELECT r.region, 
        fa.year, 
        TRUNC(((SUM(fa.forest_area_sqkm) / SUM(la.total_area_sq_mi * 2.59) * 100)::numeric), 2) AS percent_designated_as_forest
FROM forest_area fa
JOIN land_area la
ON fa.country_code = la.country_code AND fa.year = la.year
RIGHT JOIN regions r
ON r.country_code = la.country_code
WHERE fa.year = 1990 AND fa.country_code != 'WLD'
GROUP BY 1, 2
ORDER BY 1;
```

```sql
SELECT r.region, 
        fa.year, 
        TRUNC(((SUM(fa.forest_area_sqkm) / SUM(la.total_area_sq_mi * 2.59) * 100)::numeric), 2) AS percent_designated_as_forest
FROM forest_area fa
JOIN land_area la
ON fa.country_code = la.country_code AND fa.year = la.year
RIGHT JOIN regions r
ON r.country_code = la.country_code
WHERE fa.year = 2016 AND fa.country_code != 'WLD'
GROUP BY 1, 2
ORDER BY 1;  
```

```sql
SELECT r.region, 
        fa.year, 
        TRUNC(((SUM(fa.forest_area_sqkm) / SUM(la.total_area_sq_mi * 2.59) * 100)::numeric), 2) AS percent_forest_area
FROM forest_area fa
JOIN land_area la
ON fa.country_code = la.country_code AND fa.year = la.year
RIGHT JOIN regions r
ON r.country_code = la.country_code
WHERE r.region = 'World' AND (fa.year = 1990 OR fa.year = 2016)
GROUP BY 1, 2
ORDER BY 1, 2; 

```

## 3. Country-level Detail

### Success Stories

```sql
SELECT t1.country_code, t1.country_name, t1.region, t1.year, t1.lag, t1.forest_sqkm_change
FROM(SELECT fa.country_code, fa.country_name, r.region, fa.year, fa.forest_area_sqkm, LAG(forest_area_sqkm) OVER (ORDER BY fa.country_code, fa.year) AS lag, forest_area_sqkm - LAG(forest_area_sqkm) OVER (ORDER BY fa.country_code, fa.year) AS forest_sqkm_change
FROM forest_area fa
JOIN land_area la
ON fa.country_code = la.country_code AND fa.year = la.year
RIGHT JOIN regions r
ON r.country_code = la.country_code
WHERE fa.country_code != 'WLD' AND (fa.year = 1990 OR fa.year = 2016)
ORDER BY fa.year DESC, forest_sqkm_change DESC) t1
WHERE t1.forest_sqkm_change != 0 AND t1.year != 1990
```

```sql
SELECT t1.country_code, t1.country_name, t1.region,  TRUNC(((t1.percent_change)::numeric), 2) AS percent_change
FROM(SELECT fa.country_code,
       fa.country_name,
       r.region,
              fa.year,
       fa.forest_area_sqkm,
       LAG(fa.forest_area_sqkm) OVER (ORDER BY fa.country_code, fa.year) AS lag, ((fa.forest_area_sqkm - LAG(fa.forest_area_sqkm) OVER (ORDER BY fa.country_code, fa.year)) / LAG(fa.forest_area_sqkm) OVER (ORDER BY fa.country_code, fa.year)) * 100 AS percent_change
FROM forest_area fa
JOIN land_area la
ON fa.country_code = la.country_code AND fa.year = la.year
RIGHT JOIN regions r
ON r.country_code = la.country_code
WHERE fa.country_code != 'WLD' AND (fa.year = 1990 OR fa.year = 2016)) t1
WHERE t1.percent_change != 0 AND t1.year != 1990
ORDER BY 4 DESC;
```

### 3a. Which 5 countries saw the largest amount decrease in forest area from 1990 to 2016? What was the difference in forest area for each?

```sql
SELECT fa.country_code,
       fa.country_name,
       r.region,
        fa.year,
       fa.forest_area_sqkm,
       LAG(forest_area_sqkm) OVER (ORDER BY fa.country_code, fa.year) AS lag,
        forest_area_sqkm - LAG(forest_area_sqkm) OVER (ORDER BY fa.country_code, fa.year) AS forest_sqkm_change
FROM forest_area fa
JOIN land_area la
ON fa.country_code = la.country_code AND fa.year = la.year
RIGHT JOIN regions r
ON r.country_code = la.country_code
WHERE fa.country_code != 'WLD' AND (fa.year = 1990 OR fa.year = 2016)
ORDER BY fa.year DESC, forest_sqkm_change
LIMIT 5;
```

### 3b. Which 5 countries saw the largest percent decrease in forest area from 1990 to 2016? What was the percent change to 2 decimal places for each?

```sql
SELECT fa.country_code,
       fa.country_name,
       r.region,
        fa.year,
       fa.forest_area_sqkm, LAG(fa.forest_area_sqkm) OVER (ORDER BY fa.country_code, fa.year) AS "1990_forest_area_sqkm", ((fa.forest_area_sqkm - LAG(fa.forest_area_sqkm) OVER (ORDER BY fa.country_code, fa.year)) / LAG(fa.forest_area_sqkm) OVER (ORDER BY fa.country_code, fa.year)) * 100 AS percent_change
FROM forest_area fa
JOIN land_area la
ON fa.country_code = la.country_code AND fa.year = la.year
RIGHT JOIN regions r
ON r.country_code = la.country_code
WHERE fa.country_code != 'WLD' AND (fa.year = 1990 OR fa.year = 2016)
ORDER BY fa.year DESC, percent_change
LIMIT 5;
```

### 3c. If countries were grouped by percent forestation in quartiles, which group had the most countries in it in 2016?

```sql
SELECT t1.quartile_group, COUNT(t1.quartile_group)
FROM
(SELECT fa.country_name, fa.year, fa.forest_area_sqkm / (la.total_area_sq_mi * 2.59) * 100 AS percent_forest_area,
CASE WHEN fa.forest_area_sqkm / (la.total_area_sq_mi * 2.59) * 100 < 25 THEN 'Quartile 1'
WHEN fa.forest_area_sqkm / (la.total_area_sq_mi * 2.59) * 100 BETWEEN 25 AND 50 THEN 'Quartile 2'
WHEN fa.forest_area_sqkm / (la.total_area_sq_mi * 2.59) * 100 BETWEEN 50 AND 75 THEN 'Quartile 3'
WHEN fa.forest_area_sqkm / (la.total_area_sq_mi * 2.59) * 100 > 75 THEN 'Quartile 4' ELSE 'Quartile 1' END AS quartile_group
    FROM forest_area fa
    JOIN land_area la
    ON fa.country_code = la.country_code AND fa.year = la.year
    RIGHT JOIN regions r
    ON r.country_code = la.country_code
    WHERE fa.year = 2016 AND fa.country_name != 'World' AND fa.forest_area_sqkm / (la.total_area_sq_mi * 2.59) * 100 != 0
    GROUP BY 1, 2, 3) t1
GROUP BY 1
ORDER BY 1;
```

### 3d. List all of the countries that were in the 4th quartile (percent forest > 75%) in 2016.

```sql
SELECT t1.country_name, 
        t1.year, 
        t1.region, TRUNC(((t1.percent_forest_area)::numeric), 2) AS percent_forest_area, t1.quartile
FROM(SELECT fa.country_name, 
            fa.year, 
            r.region, 
            fa.forest_area_sqkm / (la.total_area_sq_mi * 2.59) * 100 AS percent_forest_area,
            NTILE(4) OVER (ORDER BY (fa.forest_area_sqkm) / (la.total_area_sq_mi * 2.59) * 100) AS quartile
    FROM forest_area fa
    JOIN land_area la
    ON fa.country_code = la.country_code AND fa.year = la.year
    JOIN regions r
    ON r.country_code = la.country_code
    WHERE fa.year = 2016
    GROUP BY 1, 2, 3, 4
    )t1
WHERE t1.quartile = 4 AND t1.percent_forest_area >75;
```









