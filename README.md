# Ad Data Cleaning Project

# Purpose
This project focuses on cleaning and preparing an advertising dataset for further exploration, analysis, and visualization. To simulate real-world messy data scenarios, I intentionally altered a public ads dataset—removing entries, changing formats, and introducing inconsistent or incorrect values. The goal was to identify errors, standardize formats, and produce a final, usable database ready for analysis. This project served as a hands-on practice in data cleaning with SQL.

# Tools & Skills Used
•PostgreSQL for database management and SQL queries
•Data Cleaning Techniques:
•Removing duplicates 
•Standardizing formats (text, numeric, timestamp)
•Handling null values and invalid entries
•Converting data types
•Data Quality Checks to ensure consistency and accuracy

# Prepare the Database

**•Created an ads table and a staging table ads_staging to hold raw imported data.**
```sql
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

**•Import CSV file**

**•Copied the dataset into the staging table for cleaning without altering the original.**
```sql
CREATE TABLE ads_staging(
LIKE ads
);

```
```sql
INSERT INTO ads_staging(
SELECT *
FROM ads
);
```

# Remove Duplicate Rows




**•Used ROW_NUMBER() with a PARTITION BY clause to identify duplicate rows.**
```sql
SELECT *,
ROW_NUMBER() OVER(
	PARTITION BY age, gender, income, click_time
	) AS row_number
FROM ads_staging;
```

**•Use a CTE function to isolate duplicate rows**
```sql
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

**•A new table with a row_number column is needed in order to remove the duplicate rows**
**•Create a duplicate table of ads_staging**
**•In left-hand margin, right click table layoffs_staging in margin •Select "Scripts" > "CREATE Script"**
**•Modify table name to "layoffs_staging2"**
**•Add an integer-type "row_number" column**
```sql
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

**•Stored data in ads_staging2 with a row_number column and deleted all duplicates where row_number > 1.**
```sql
INSERT INTO ads_staging2
SELECT *,
ROW_NUMBER() OVER(
	PARTITION BY age, gender, income, click_time
	) AS row_number
FROM ads_staging;
```
```sql
DELETE
FROM ads_staging2
WHERE row_number > 1;
```



#  Standardize the Data










**Age: Removed invalid values, converted written numbers to integers.**
**•Check validity of listed ages**
```sql
SELECT DISTINCT age
FROM ads_staging2
ORDER BY 1;
```
**•Negative values and ages below 18 are listed**
**•Make these values NULL**
```sql
UPDATE ads_staging2
SET age = NULL
WHERE age IN ('-_', '-__', '0', '1','2','3','4','5','6','7','8','9','10','11','12','13','14','15','16','17');
```
**•Some values are in string form rather than integer form**
**•Change the strings to integers**
```sql
SELECT age,
	CASE
	WHEN LOWER(age) LIKE 'fourty-nine' THEN '49'
	WHEN LOWER(age) LIKE 'sixty-three' THEN '63'
	WHEN LOWER(age) LIKE 'twenty' THEN '20'
	ELSE age
	END AS age_cleaned
FROM ads_staging2;
```
```sql
UPDATE ads_staging2
SET age = CASE
	WHEN LOWER(age) LIKE 'fourty-nine' THEN '49'
	WHEN LOWER(age) LIKE 'sixty-three' THEN '63'
	WHEN LOWER(age) LIKE 'twenty' THEN '20'
	ELSE age
	END;
```

**Gender: Standardized to "Male," "Female," or "Other" with proper capitalization.**
**•Check validity of listed genders**
```sql
SELECT DISTINCT gender
FROM ads_staging2
ORDER BY 1;
```
**•Some genders appear as 'f' or 'm' rather than 'Female' or Male'**
**•Modify this so each gender is spelled out and begun with a capital letter**
```sql
SELECT gender,
	CASE
	WHEN LOWER(gender) LIKE 'f%' THEN 'Female'
	WHEN LOWER(gender) LIKE 'm%' THEN 'Male'
	WHEN LOWER(gender) LIKE 'o%' THEN 'Other'
	ELSE gender
	END AS gender_cleaned
FROM ads_staging2;
```
```sql
UPDATE ads_staging2
SET gender = CASE
	WHEN LOWER(gender) LIKE 'f%' THEN 'Female'
	WHEN LOWER(gender) LIKE 'm%' THEN 'Male'
	WHEN LOWER(gender) LIKE 'o%' THEN 'Other'
	ELSE gender
	END;
```

 **Income: Removed negative signs, currency symbols, and commas; converted to decimal type.y**
**•Check validity of listed incomes**
```sql
SELECT DISTINCT income
FROM ads_staging2
ORDER BY 1;
```
**•Some incomes appear as negative values**
**•Remove the '-' symbol in this instance**
```sql
UPDATE ads_staging2
SET income = TRIM(LEADING '-' FROM income)
WHERE income LIKE '-%';
```
**•Some incomes begin with a '$'**
**•Remove this symbol**
```sql
UPDATE ads_staging2
SET income = TRIM(LEADING '$' FROM income)
WHERE income LIKE '$%';
```
**•Some incomes appear with a comma '10,000' rather than '10000'**
**•Remove the ',' in these instances**
```sql
SELECT income FROM ads_staging2
WHERE income LIKE '%,%';
```
```sql
SELECT REPLACE(income, ',', '')
FROM ads_staging2;
```
```sql
UPDATE ads_staging2
SET income = REPLACE(income, ',', '')
WHERE income LIKE '%,%';
```

**Ad Placement: Standardized to proper case (e.g., "Social Media", "Website").**
**•Check validity of ad placements**
```sql
SELECT DISTINCT ad_placement
FROM ads_staging2;
```
**•Some results are in all lowercase text**
**•Modify these results so words begin with capital letters**
```sql
SELECT ad_placement,
	CASE
	WHEN LOWER(ad_placement) LIKE 'social%' THEN 'Social Media'
	WHEN LOWER(ad_placement) LIKE 'web%' THEN 'Website'
	ELSE ad_placement
	END AS ad_placement_cleaned
FROM ads_staging2;
```
```sql
UPDATE ads_staging2
SET ad_placement = CASE
	WHEN LOWER(ad_placement) LIKE 'social%' THEN 'Social Media'
	WHEN LOWER(ad_placement) LIKE 'web%' THEN 'Website'
	ELSE ad_placement
	END;
```

**Clicks: Converted written numbers to integers.**
**•Check validity of clicks**
```sql
SELECT DISTINCT clicks
FROM ads_staging2
ORDER BY 1;
```
**•Some data is listed as a string rather than an integer**
**•Modify this so all data is listed as an integer**
```sql
SELECT clicks,
	CASE
		WHEN LOWER(clicks) LIKE 'six' THEN '6'
		WHEN LOWER(clicks) LIKE 'three' THEN '3'
		WHEN LOWER(clicks) LIKE 'two' THEN '2'
		ELSE clicks
		END AS clicks_cleaned
	FROM ads_staging2;
```
```sql
UPDATE ads_staging2
SET clicks = CASE
	WHEN LOWER(clicks) LIKE 'six' THEN '6'
		WHEN LOWER(clicks) LIKE 'three' THEN '3'
		WHEN LOWER(clicks) LIKE 'two' THEN '2'
		ELSE clicks
		END;
```

**Click Time: Fixed inconsistent timestamp formats.**
**•Check validity of click time data
```sql
SELECT DISTINCT click_time
FROM ads_staging2
ORDER BY 1;
```
**•One timestamp is listed in a MM-DD-YY format rather than a YYYY-MM-DD format**
**•Modify this so the format matches the other timestamps**
```sql
SELECT TO_CHAR(
	TO_TIMESTAMP('04-17-2024 20:45:57', 'MM-DD-YYYY HH24:MI:SS'),
	'YYYY-MM-DD HH24:MI:SS'
);
```
```sql
UPDATE ads_staging2
SET click_time = TO_CHAR(
	TO_TIMESTAMP('04-17-2024 20:45:57', 'MM-DD-YYYY HH24:MI:SS'),
	'YYYY-MM-DD HH24:MI:SS'
)
WHERE click_time LIKE '04-17%';
```

# Finalize the Cleaned Dataset


**•Dropped the row_number column.**
```sql
ALTER TABLE ads_staging2
DROP COLUMN row_number;
```

**•Converted columns to appropriate data types (integer, decimal, timestamp).**
```sql
ALTER TABLE ads_staging2
ALTER COLUMN age
TYPE integer
USING (age::integer);
```
```sql
ALTER TABLE ads_staging2
ALTER COLUMN income
TYPE decimal
USING (income::decimal);
```
**•Change clicks column from text to integer type**
```sql
ALTER TABLE ads_staging2
ALTER COLUMN clicks
TYPE integer
USING (clicks::integer);
```
**•Change click_time from text to timestamp type**
```sql
ALTER TABLE ads_staging2
ALTER COLUMN click_time
TYPE timestamp
USING (click_time::timestamp);
```
**•Change converstion_rate from text to decimal type**
```sql
ALTER TABLE ads_staging2
ALTER COLUMN conversion_rate
TYPE decimal
USING (conversion_rate::decimal);
```
**•Change click through rate column from text to decimal type**
```sql
ALTER TABLE ads_staging2
ALTER COLUMN click_through_rate
TYPE decimal
USING (click_through_rate::decimal);
```

**•Renamed ads_staging2 to ads_final to indicate the cleaned dataset is ready for use.**
```sql
ALTER TABLE ads_staging2
RENAME TO ads_final;
```



# Conclusion
Original dataset: Contained duplicates, inconsistent formatting, invalid values, and mixed data types.
Cleaned dataset: Fully standardized, validated, and stored in ads_final, ready for analysis and visualization.

**This project reinforced the importance of:**
•Creating staging tables to preserve raw data.
•Writing targeted SQL queries to detect and correct inconsistencies.
•Using data type conversions to prepare for accurate analysis.
