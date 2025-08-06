# Ad-Data-Cleaning

**Create Table**
```
DROP TABLE IF EXISTS ads
CREATE TABLE ads (
	age text,
	gender text,
	income text,
	location text,
	ad_type text,
	ad_topic text,
	ad_placement text,
	clicks text,
	click_time text,
	conversion_rate text,
	click_through_rate text
);
```

**Import CSV file**

**Create staging table**
```
CREATE TABLE ads_staging(
LIKE ads
);

```
**Copy data to staging table**
```
INSERT INTO ads_staging(
SELECT *
FROM ads
);
```

# REMOVING THE DUPLICATES

**Set up row number function to find duplicate rows**
```
SELECT *,
ROW_NUMBER() OVER(
	PARTITION BY age, gender, income, click_time
	) AS row_number
FROM ads_staging;
```

**Use a CTE function to isolate duplicate rows**
```
WITH duplicate_cte AS(
SELECT *,
ROW_NUMBER() OVER(
	PARTITION BY age, gender, income, click_time
	) AS row_number
FROM ads_staging
)
SELECT *
FROM duplicate_cte
WHERE row_number > 1;
```

**Copy create table statement into query tool, adding a new title and "row_number" column**
```
CREATE TABLE IF NOT EXISTS public.ads_staging2
(
    age text COLLATE pg_catalog."default",
    gender text COLLATE pg_catalog."default",
    income text COLLATE pg_catalog."default",
    location text COLLATE pg_catalog."default",
    ad_type text COLLATE pg_catalog."default",
    ad_topic text COLLATE pg_catalog."default",
    ad_placement text COLLATE pg_catalog."default",
    clicks text COLLATE pg_catalog."default",
    click_time text COLLATE pg_catalog."default",
    conversion_rate text COLLATE pg_catalog."default",
    click_through_rate text COLLATE pg_catalog."default",
	row_number int
)
TABLESPACE pg_default;
ALTER TABLE IF EXISTS public.ads_staging
    OWNER to postgres;
```

**Copy data to staging table 2**
```
INSERT INTO ads_staging2(
SELECT *
FROM ads
);
```

**Insert values into the row_number column**
```
INSERT INTO ads_staging2
SELECT *,
ROW_NUMBER() OVER(
	PARTITION BY age, gender, income, click_time
	) AS row_number
FROM ads_staging;
```

**Delete duplicate rows**
```
DELETE
FROM ads_staging2
WHERE row_number > 1;
```



# STANDARDIZING THE DATA

**AGE**
```
SELECT DISTINCT age
FROM ads_staging2
ORDER BY 1;
```

```
UPDATE ads_staging2
SET age = NULL
WHERE age IN ('-_', '-__', '0', '1','2','3','4','5','6','7','8','9','10','11','12','13','14','15','16','17');
```

```
SELECT age,
	CASE
	WHEN LOWER(age) LIKE 'fourty-nine' THEN '49'
	WHEN LOWER(age) LIKE 'sixty-three' THEN '63'
	WHEN LOWER(age) LIKE 'twenty' THEN '20'
	ELSE age
	END AS age_cleaned
FROM ads_staging2;
```

```
UPDATE ads_staging2
SET age = CASE
	WHEN LOWER(age) LIKE 'fourty-nine' THEN '49'
	WHEN LOWER(age) LIKE 'sixty-three' THEN '63'
	WHEN LOWER(age) LIKE 'twenty' THEN '20'
	ELSE age
	END;
```

**GENDER**
```
SELECT DISTINCT gender
FROM ads_staging2
ORDER BY 1;
```

```
SELECT gender,
	CASE
	WHEN LOWER(gender) LIKE 'f%' THEN 'Female'
	WHEN LOWER(gender) LIKE 'm%' THEN 'Male'
	WHEN LOWER(gender) LIKE 'o%' THEN 'Other'
	ELSE gender
	END AS gender_cleaned
FROM ads_staging2;
```

```
UPDATE ads_staging2
SET gender = CASE
	WHEN LOWER(gender) LIKE 'f%' THEN 'Female'
	WHEN LOWER(gender) LIKE 'm%' THEN 'Male'
	WHEN LOWER(gender) LIKE 'o%' THEN 'Other'
	ELSE gender
	END;
```

 **Income**
```
SELECT DISTINCT income
FROM ads_staging2
ORDER BY 1;
```

```
UPDATE ads_staging2
SET income = TRIM(LEADING '-' FROM income)
WHERE income LIKE '-%';
```

```
UPDATE ads_staging2
SET income = TRIM(LEADING '$' FROM income)
WHERE income LIKE '$%';
```

```
SELECT income FROM ads_staging2
WHERE income LIKE '%,%';
```

```
SELECT REPLACE(income, ',', '')
FROM ads_staging2;
```

```
UPDATE ads_staging2
SET income = REPLACE(income, ',', '')
WHERE income LIKE '%,%';
```

**Ad Placement**
```
SELECT DISTINCT ad_placement
FROM ads_staging2;
```
```
SELECT ad_placement,
	CASE
	WHEN LOWER(ad_placement) LIKE 'social%' THEN 'Social Media'
	WHEN LOWER(ad_placement) LIKE 'web%' THEN 'Website'
	ELSE ad_placement
	END AS ad_placement_cleaned
FROM ads_staging2;
```

```
UPDATE ads_staging2
SET ad_placement = CASE
	WHEN LOWER(ad_placement) LIKE 'social%' THEN 'Social Media'
	WHEN LOWER(ad_placement) LIKE 'web%' THEN 'Website'
	ELSE ad_placement
	END;
```

**Clicks**

```
SELECT DISTINCT clicks
FROM ads_staging2
ORDER BY 1;
```

```
SELECT clicks,
	CASE
		WHEN LOWER(clicks) LIKE 'six' THEN '6'
		WHEN LOWER(clicks) LIKE 'three' THEN '3'
		WHEN LOWER(clicks) LIKE 'two' THEN '2'
		ELSE clicks
		END AS clicks_cleaned
	FROM ads_staging2;
```

```
UPDATE ads_staging2
SET clicks = CASE
	WHEN LOWER(clicks) LIKE 'six' THEN '6'
		WHEN LOWER(clicks) LIKE 'three' THEN '3'
		WHEN LOWER(clicks) LIKE 'two' THEN '2'
		ELSE clicks
		END;
```

**Click Time**
```
SELECT DISTINCT click_time
FROM ads_staging2
ORDER BY 1;
```

```
SELECT TO_CHAR(
	TO_TIMESTAMP('04-17-2024 20:45:57', 'MM-DD-YYYY HH24:MI:SS'),
	'YYYY-MM-DD HH24:MI:SS'
);
```

```
UPDATE ads_staging2
SET click_time = TO_CHAR(
	TO_TIMESTAMP('04-17-2024 20:45:57', 'MM-DD-YYYY HH24:MI:SS'),
	'YYYY-MM-DD HH24:MI:SS'
)
WHERE click_time LIKE '04-17%';
```

# REMOVING IRRELEVANT COLUMNS & ALTERING DATA TYPES FROM TEXT

**Delete row_number column**
```
ALTER TABLE ads_staging2
DROP COLUMN row_number;
```

**Change data from text to a more suitable type**
```
ALTER TABLE ads_staging2
ALTER COLUMN age
TYPE integer
USING (age::integer);
```

```
ALTER TABLE ads_staging2
ALTER COLUMN income
TYPE decimal
USING (income::decimal);
```

```
ALTER TABLE ads_staging2
ALTER COLUMN clicks
TYPE integer
USING (clicks::integer);
```

```
ALTER TABLE ads_staging2
ALTER COLUMN click_time
TYPE timestamp
USING (click_time::timestamp);
```

```
ALTER TABLE ads_staging2
ALTER COLUMN conversion_rate
TYPE decimal
USING (conversion_rate::decimal);
```

```
ALTER TABLE ads_staging2
ALTER COLUMN click_through_rate
TYPE decimal
USING (click_through_rate::decimal);
```

**Change title of table to show data has been cleaned**
```
ALTER TABLE ads_staging2
RENAME TO ads_final;
```
