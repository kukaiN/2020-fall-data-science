
# SQL:  Structured Query Language  Exercise

### Getting Started
1. Go to BigQuery UI https://console.cloud.google.com/bigquery
2. Add in the public data sets.
	3. Click the Add Data icon
	4. Add any dataset
	5. `bigquery-public-data` should become visible and populate in the BigQuery UI.
3. Add your queries where it says [YOUR QUERY HERE].
4. Make sure you add your query in between the triple tick marks.
---

For this section of the exercise we will be using the `bigquery-public-data.austin_311.311_service_requests`  table.

5. Write a query that tells us how many rows are in the table.
	```sql
	SELECT
		COUNT(*) as total_row_counts
	FROM
		`bigquery-public-data.austin_311.311_service_requests` AS T
	```

7. Write a query that tells us how many _distinct_ values there are in the complaint_description column.
	```sql
	SELECT
		complaint_description,
		COUNT(DISTINCT complaint_description) AS distinct_complaints
	FROM
		`bigquery-public-data.austin_311.311_service_requests` AS T
	GROUP BY
		complaint_description
	```

8. Write a query that counts how many times each owning_department appears in the table and orders them from highest to lowest.
	```sql
	SELECT
		owning_department,
		COUNT(owning_department) AS dept_counts
	FROM
		`bigquery-public-data.austin_311.311_service_requests` AS T
	GROUP BY
		owning_department
	ORDER BY
		dept_counts DESC
	```

9. Write a query that lists the top 5 complaint_description that appear most and the amount of times they appear in this table. (hint... limit)
	```sql
	WITH
		T AS(
			SELECT
				complaint_description,
				COUNT(complaint_description) AS distinct_complaints
			FROM
				`bigquery-public-data.austin_311.311_service_requests`
			GROUP BY
				complaint_description )
	SELECT
		*
	FROM
		T
	ORDER BY
		distinct_complaints DESC
	LIMIT
		5
	```
10. Write a query that lists and counts all the complaint_description, just for the where the owning_department is 'Animal Services Office'.
	```sql
	SELECT
		complaint_description,
		COUNT(complaint_description) AS distinct_complaints
	FROM
		`bigquery-public-data.austin_311.311_service_requests`
	WHERE
		owning_department = 'Animal Services Office'
	GROUP BY
		complaint_description
	```

11. Write a query to check if there are any duplicate values in the unique_key column (hint.. There are two was to do this, one is to use a temporary table for the groupby, then filter for values that have more than one count, or, using just one table but including the  `having` function).
	```sql
	SELECT
		unique_key,
		COUNT(unique_key) AS counts
	FROM
		`bigquery-public-data.austin_311.311_service_requests`
	GROUP BY
		unique_key
	HAVING
		counts > 1
	```


### For the next question, use the `census_bureau_usa` tables.

1. Write a query that returns each zipcode and their population for 2000 and 2010.
	```sql
	WITH
		T2000 AS (
			SELECT
				zipcode,
				SUM(population) AS populationOf2000
			FROM
				`bigquery-public-data.census_bureau_usa.population_by_zip_2000`
			GROUP BY
				zipcode ),
		T2010 AS (
			SELECT
				zipcode,
				SUM(population) AS populationOf2010
			FROM
				`bigquery-public-data.census_bureau_usa.population_by_zip_2010`
			GROUP BY
				zipcode )
	SELECT
		A.zipcode,
		A.populationOf2000,
		B.populationOf2010
	FROM
		T2000 AS A
	JOIN
		T2010 AS B
	ON
		A.zipcode = B.zipcode
	```

### For the next section, use the  `bigquery-public-data.google_political_ads.advertiser_weekly_spend` table.
1. Using the `advertiser_weekly_spend` table, write a query that finds the advertiser_name that spent the most in usd.
	```sql
	SELECT
  		advertiser_name,
  		SUM(spend_usd) AS total_usd
	FROM
  		`bigquery-public-data.google_political_ads.advertiser_weekly_spend`
	GROUP BY
		advertiser_name
	ORDER BY
		total_usd DESC
	LIMIT
		1
	```
2. Who was the 6th highest spender? (No need to insert query here, just type in the answer.)
	```
	TOM STEYER 2020 with $8801900
	```

3. What week_start_date had the highest spend? (No need to insert query here, just type in the answer.)
	```
	2020-09-13
	```

4. Using the `advertiser_weekly_spend` table, write a query that returns the sum of spend by week (using week_start_date) in usd for the month of August only.
	```sql
	WITH
		T AS (
			SELECT
				week_start_date,
				SUM(spend_usd) AS total_usd
			FROM
				`bigquery-public-data.google_political_ads.advertiser_weekly_spend`
			GROUP BY
				week_start_date),
		TT AS(
			SELECT
				EXTRACT(MONTH FROM week_start_date) AS the_month,
				*
			FROM
				T)
	SELECT
		week_start_date,
		total_usd
	FROM
		TT
	WHERE
		the_month = 8
	ORDER BY
		week_start_date DESC
	```
6.  How many ads did the 'TOM STEYER 2020' campaign run? (No need to insert query here, just type in the answer.)
	```
	50
	```
7. Write a query that has, in the US region only, the total spend in usd for each advertiser_name and how many ads they ran. (Hint, you're going to have to join tables for this one).
	```sql
	WITH
		T AS (
			SELECT
				advertiser_name, sum(spend_usd) AS total_spending
			FROM
				`bigquery-public-data.google_political_ads.advertiser_weekly_spend`
			WHERE
			election_cycle = "US-Federal-2018"
			group by advertiser_name),
		TT AS (
			SELECT
				advertiser_name, count(advertiser_id) as ad_counts
			FROM
				`bigquery-public-data.google_political_ads.advertiser_weekly_spend`
			WHERE
				election_cycle = "US-Federal-2018"
				group by advertiser_name
			)
	SELECT
		A.advertiser_name, A.total_spending, B.ad_counts spend_eur
	FROM
		T AS A join TT AS B
	ON
	 A.advertiser_name = B.advertiser_name

	```
8. For each advertiser_name, find the average spend per ad.
	```sql
	WITH
	T AS (
		SELECT
			advertiser_name,
			SUM(spend_usd) AS total_spending,
			count(advertiser_id) as ad_count
		FROM
			`bigquery-public-data.google_political_ads.advertiser_weekly_spend`
		WHERE
			election_cycle = "US-Federal-2018"
		GROUP BY
			advertiser_name )
	SELECT
		advertiser_name, (total_spending/ad_count) as ave_spend_per_ad
	FROM
		T
	```
10. Which advertiser_name had the lowest average spend per ad that was at least above 0.
	```
	AMERICAN JOBS AND GROWTH PAC with 25000.0 USD
	```
## For this next section, use the `new_york_citibike` datasets.

1. Who went on more bike trips, Males or Females?
	```sql
	SELECT
		gender,
		COUNT(gender) AS counts
	FROM
		`bigquery-public-data.new_york_citibike.citibike_trips`
	GROUP BY
		gender
	ORDER BY
		counts DESC
	LIMIT
		1
	```
2. What was the average, shortest, and longest bike trip taken in minutes?
	```sql
	with T as
		(SELECT
			tripduration,
			(tripduration/60) as trip_minute
		FROM
			`bigquery-public-data.new_york_citibike.citibike_trips`
		where
			tripduration is not Null
		)
	Select
		-- trip_minute
		avg(trip_minute)
	FROM
		T
	/* -- used to find max and min
	ORDER BY
		trip_minute DESC --ASC
	*/
	```

	```
	longest: 325167.48333333334
	shortest: 1.0 (smallest number excluding Null)
	Average: 16.041516425648233 (average excluding Null)
	```

3. Write a query that, for every station_name, has the amount of trips that started there and the amount of trips that ended there. (Hint, use two temporary tables, one that counts the amount of starts, the other that counts the number of ends, and then join the two.)
	```sql
	WITH
		T AS (
			SELECT
				start_station_name,
				COUNT(start_station_name) AS start_count
			FROM
				`bigquery-public-data.new_york_citibike.citibike_trips`
			GROUP BY
				start_station_name),
	TT AS (
		SELECT
			end_station_name,
			COUNT(end_station_name) AS end_count
		FROM
			`bigquery-public-data.new_york_citibike.citibike_trips`
		GROUP BY
			end_station_name)
	SELECT
		T.start_station_name AS station_name,
		T.start_count,
		TT.end_count
	FROM
		T
	JOIN
		TT
	ON
		T.start_station_name = TT.end_station_name
	```
# The next section is the Google Colab section.
1. Open up this [this Colab notebook](https://colab.research.google.com/drive/1kHdTtuHTPEaMH32GotVum41YVdeyzQ74?usp=sharing).
2. Save a copy of it in your drive.
3. Rename your saved version with your initials.
4. Click the 'Share' button on the top right.
5. Change the permissions so anyone with link can view.
6. Copy the link and paste it right below this line.
	* YOUR LINK:  [hyperlink to colab](https://colab.research.google.com/drive/1rxeF4wLQV9SbgzXE7OBUmqkpS6KDoGge?usp=sharing)
7. Complete the two questions in the colab notebook file.
