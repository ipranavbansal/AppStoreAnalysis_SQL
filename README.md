## AppStoreAnalysis_SQL

## Table of Contents

- [Project Overview](#project-overview)
- [Data Sources](#data-sources)
- [Results/Findings](#results)
---

### Project Overview

Our stakeholder is an app developer, we are looking for some insights in the App Store database which may help them building an app.

Target Insights could be like -
1. Is there any effect of length of app description on average user rating?
2. Is there any effect of no. of supported languages on average user rating?
3. Is there any effect of paid or unapid version of apps on average user ratings?

---

### Data Sources

- Keep in mind, SQL online Editor doesnt allow to import file > 4MB, so we have break down the file apple_description.csv into 4 parts and after importing, with the help of UNION ALL we can combine it in a single table.

---

### Tools 

- [SQL Online Editor](https://sqliteonline.com) - for data analysis
---
### Results

1. App description has positive correlation with app ratings.
2. Best average user rating came for the languages around 10-15 bucket.
3. Paid apps has better average user rating of 3.7, whereas free ones are around 3.4.
4. Finance and Books have least average user rating.

---
### Recommendations

1. Write longer app descriptions, greater than 2000 characters.
2. NO noeed to maximise on supported languages, keep it around 10-15.
3. You can go with paid apps as they have bettter user base and reviews.
4. There is chance of category penetration as user needs are not met in these categories.
---

### Limitations

##### NA
---
### References

##### NA
---
### Codebase

```SQL
create table applestore_description_combined AS

select * from appleStore_description1
Union all
SELECT	* From appleStore_description2
Union all
SELECT	* From appleStore_description3
Union all
SELECT	* From appleStore_description4

** EDA **

-- to check the missing data in two tables

select count(distinct id) from AppleStore

SELECT count (DISTINCT id) FROM applestore_description_combined

--same result 

-- to check for missing values in some of the key fieldsAppleStore
SELECT count(*) FROM AppleStore
where track_name is NULL OR prime_genre is NULL 

-- similarly check for the other tables as well

-- to find the trending categories out there
SELECT prime_genre, count(prime_genre) AS apps
from AppleStore 
GROUP BY prime_genre
ORDER BY apps DESC

--to find the stats about the ratingsAppleStore

select min(user_rating) as min_rating, max(user_rating) AS max_rating, avg(user_rating) As avg_rating
FROM AppleStore

--determine whether paid apps have higher rating than free appsAppleStore

-- started with no. of free and paid apps
select count(id) from AppleStore where price == 0
select count(id) from AppleStore where price != 0


select avg(user_rating) from AppleStore
where price != 0
select avg(user_rating) from AppleStore
where price == 0

-- other and much better methodAppleStore

select case 
			WHEN price > 0 then 'PAID'
			ELSE 'FREE'
            END AS app_type,
            avg(user_rating) AS avg_user_rating
From AppleStore
GROUP BY app_type

-- check if app that supports more languages have higher ratings 


SELECT DISTINCT lang_num as SL, avg(user_rating) as avg_user_rating 
FROM AppleStore
GROUP BY lang_num
HAVING avg_user_rating >=4
ORDER BY avg_user_rating DESC


SELECT CASE
			WHEN lang_num < 8 THEN "< 8 "
            WHEN lang_num BETWEEN 12 AND 15 THEN "12 - 15 "
            WHEN lang_num BETWEEN 10 AND 15 THEN "10 - 15 "
            WHEN lang_num BETWEEN 15 AND 25 THEN "15 - 25 "
            WHEN lang_num BETWEEN 25 AND 35 THEN "25 - 35 "
            WHEN lang_num BETWEEN 35 AND 50 THEN "35 - 50 "
            ELSE "> 50"
            END AS lang_bucket,
			avg(user_rating) as avg_user_rating 
FROM AppleStore
GROUP BY lang_bucket
--HAVING avg_user_rating >=4
ORDER BY avg_user_rating DESC




-- check genres with low ratings

SELECT prime_genre, avg(user_rating) 
FROM AppleStore
GROUP BY prime_genre 
HAVING avg(user_rating) <3
order BY avg(user_rating) DESC


-- check if there is a correlation between app description length and user ratingAppleStore


SELECT length(B.app_desc) as b1, avg(A.user_rating) as a1
FROM 
	AppleStore as A 
	JOIN applestore_description_combined AS B
	ON A.id = B.id 
GROUP BY b1
having a1>3
ORder BY b1 DESC



select case 
			when length(B.app_desc) <1000 THEN 'SHORT'
            WHEn length(B.app_desc) BETWEEN 1000 and 2000 then 'medium'
            else 'longer'
            end as b1,
            avg(A.user_rating) as a1
                        
FROM	
                        AppleStore as A
JOIN 					applestore_description_combined AS B
	
ON 
                        A.id = B.id 
GROUP BY b1
ORDER BY a1
