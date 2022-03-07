Getting information about your database
As we saw in the video, PostgreSQL has a system database called INFORMATION_SCHEMA that allows us to extract information about objects, including tables, in our database.

In this exercise we will look at how to query the tables table of the INFORMATION_SCHEMA database to discover information about tables in the DVD Rentals database including the name, type, schema, and catalog of all tables and views and then how to use the results to get additional information about columns in our tables.

Select all columns from the INFORMATION_SCHEMA.TABLES system database. Limit results that have a public table_schema.
 -- Select all columns from the TABLES system database
 SELECT * 
 FROM INFORMATION_SCHEMA.TABLES
 -- Filter by schema
 WHERE table_schema = 'public';
 
 Select all columns from the INFORMATION_SCHEMA.COLUMNS system database. Limit by table_name to actor
 
  -- Select all columns from the COLUMNS system database
 SELECT * 
 FROM INFORMATION_SCHEMA.COLUMNS
 WHERE table_name = 'actor';
 
 Determining data types
The columns table of the INFORMATION_SCHEMA database also allows us to extract information about the data types of columns in a table. We can extract information like the character or string length of a CHAR or VARCHAR column or the precision of a DECIMAL or NUMERIC floating point type.

Using the techniques you learned in the lesson, let's explore the customer table of our DVD Rental database.

Select the column name and data type from the INFORMATION_SCHEMA.COLUMNS system database.
Limit results to only include the customer table.

-- Get the column name and data type
SELECT
 	column_name, 
    data_type
-- From the system database information schema
FROM INFORMATION_SCHEMA.COLUMNS 
-- For the customer table
WHERE table_name = 'customer';

Interval data types
INTERVAL data types provide you with a very useful tool for performing arithmetic on date and time data types. For example, let's say our rental policy requires a DVD to be returned within 3 days. We can calculate the expected_return_date for a given DVD rental by adding an INTERVAL of 3 days to the rental_date from the rental table. We can then compare this result to the actual return_date to determine if the DVD was returned late.

Let's try this example in the exercise.

Select the rental date and return date from the rental table.
Add an INTERVAL of 3 days to the rental_date to calculate the expected return date`.
SELECT
 	-- Select the rental and return dates
	rental_date,
	return_date,
 	-- Calculate the expected_return_date
	rental_date + INTERVAL '3 days' AS expected_return_date
FROM rental;

Accessing data in an ARRAY
In our DVD Rentals database, the film table contains an ARRAY for special_features which has a type of TEXT[]. Much like any ARRAY data type in PostgreSQL, a TEXT[] array can store an array of TEXT values. This comes in handy when you want to store things like phone numbers or email addresses as we saw in the lesson.

Let's take a look at the special_features column and also practice accessing data in the ARRAY.

Select the title and special features from the film table and compare the results between the two columns.

-- Select the title and special features column 
SELECT 
  title, 
  special_features 
FROM film;

Select all films that have a special feature Trailers by filtering on the first index of the special_features ARRAY.

-- Select the title and special features column 
SELECT 
  title, 
  special_features 
FROM film
-- Use the array index of the special_features column
WHERE special_features[1] = 'Trailers';

Now let's select all films that have Deleted Scenes in the second index of the special_features ARRAY.

-- Select the title and special features column 
SELECT 
  title, 
  special_features 
FROM film
-- Use the array index of the special_features column
WHERE special_features[2] = 'Deleted Scenes';

Searching an ARRAY with ANY
As we saw in the video, PostgreSQL also provides the ability to filter results by searching for values in an ARRAY. The ANY function allows you to search for a value in any index position of an ARRAY. Here's an example.

WHERE 'search text' = ANY(array_name)
When using the ANY function, the value you are filtering on appears on the left side of the equation with the name of the ARRAY column as the parameter in the ANY function.

Match 'Trailers' in any index of the special_features ARRAY regardless of position.

SELECT
  title, 
  special_features 
FROM film 
-- Modify the query to use the ANY function 
WHERE 'Trailers' = ANY (special_features);

Searching an ARRAY with @>
The contains operator @> operator is alternative syntax to the ANY function and matches data in an ARRAY using the following syntax.

WHERE array_name @> ARRAY['search text'] :: type[]
So let's practice using this operator in the exercise.

Use the contains operator to match the text Deleted Scenes in the special_features column.

SELECT 
  title, 
  special_features 
FROM film 
-- Filter where special_features contains 'Deleted Scenes'
WHERE special_features @> ARRAY['Deleted Scenes'];

Adding and subtracting date and time values
In this exercise, you will calculate the actual number of days rented as well as the true expected_return_date by using the rental_duration column from the film table along with the familiar rental_date from the rental table.

This will require that you dust off the skills you learned from prior courses on how to join two or more tables together. To select columns from both the film and rental tables in a single query, we'll need to use the inventory table to join these two tables together since there is no explicit relationship between them. Let's give it a try!

Subtract the rental_date from the return_date to calculate the number of days_rented.

SELECT f.title, f.rental_duration,
    -- Calculate the number of days rented
    r.return_date - r.rental_date AS days_rented
FROM film AS f
     INNER JOIN inventory AS i ON f.film_id = i.film_id
     INNER JOIN rental AS r ON i.inventory_id = r.inventory_id
ORDER BY f.title;

Now use the AGE() function to calculate the days_rented.

SELECT f.title, f.rental_duration,
    -- Calculate the number of days rented
	AGE(r.return_date, r.rental_date) AS days_rented
FROM film AS f
	INNER JOIN inventory AS i ON f.film_id = i.film_id
	INNER JOIN rental AS r ON i.inventory_id = r.inventory_id
ORDER BY f.title;

INTERVAL arithmetic
If you were running a real DVD Rental store, there would be times when you would need to determine what film titles were currently out for rental with customers. In the previous exercise, we saw that some of the records in the results had a NULL value for the return_date. This is because the rental was still outstanding.

Each rental in the film table has an associated rental_duration column which represents the number of days that a DVD can be rented by a customer before it is considered late. In this example, you will exclude films that have a NULL value for the return_date and also convert the rental_duration to an INTERVAL type. Here's a reminder of one method for performing this conversion.

SELECT INTERVAL '1' day * timestamp '2019-04-10 12:34:56'

Convert rental_duration by multiplying it with a 1 day INTERVAL
Subtract the rental_date from the return_date to calculate the number of days_rented.
Exclude rentals with a NULL value for return_date.

SELECT
	f.title,
 	-- Convert the rental_duration to an interval
    INTERVAL '1' day * f.rental_duration,
 	-- Calculate the days rented as we did previously
    r.return_date - r.rental_date AS days_rented
FROM film AS f
    INNER JOIN inventory AS i ON f.film_id = i.film_id
    INNER JOIN rental AS r ON i.inventory_id = r.inventory_id
-- Filter the query to exclude outstanding rentals
WHERE r.return_date IS NOT NULL
ORDER BY f.title;

Calculating the expected return date
So now that you've practiced how to add and subtract timestamps and perform relative calculations using intervals, let's use those new skills to calculate the actual expected return date of a specific rental. As you've seen in previous exercises, the rental_duration is the number of days allowed for a rental before it's considered late. To calculate the expected_return_date you will want to use the rental_duration and add it to the rental_date.

Convert rental_duration by multiplying it with a 1-day INTERVAL.
Add it to the rental date.

SELECT
    f.title,
	r.rental_date,
    f.rental_duration,
    -- Add the rental duration to the rental date
    INTERVAL '1' day * f.rental_duration + r.rental_date AS expected_return_date,
    r.return_date
FROM film AS f
    INNER JOIN inventory AS i ON f.film_id = i.film_id
    INNER JOIN rental AS r ON i.inventory_id = r.inventory_id
ORDER BY f.title;

Working with the current date and time
Because the Sakila database is a bit dated and most of the date and time values are from 2005 or 2006, you are going to practice using the current date and time in our queries without using Sakila. You'll get back into working with this database in the next video and throughout the remainder of the course. For now, let's practice the techniques you learned about so far in this chapter to work with the current date and time.

As you learned in the video, NOW() and CURRENT_TIMESTAMP can be used interchangeably.

Now, let's use the CAST() function to eliminate the timezone from the current timestamp.

--Select the current timestamp without a timezone
SELECT CAST( NOW() AS timestamp )

Finally, let's select the current date.
Use CAST() to retrieve the same result from the NOW() function.

SELECT 
	-- Select the current date
	CURRENT_DATE,
    -- CAST the result of the NOW() function to a date
    CAST( NOW() AS date )
    
    Manipulating the current date and time
Most of the time when you work with the current date and time, you will want to transform, manipulate, or perform operations on the value in your queries. In this exercise, you will practice adding an INTERVAL to the current timestamp as well as perform some more advanced calculations.

Let's practice retrieving the current timestamp. For this exercise, please use CURRENT_TIMESTAMP instead of the NOW() function and if you need to convert a date or time value to a timestamp data type, please use the PostgreSQL specific casting rather than the CAST() function.

Select the current timestamp without timezone and alias it as right_now.

--Select the current timestamp without timezone
SELECT CURRENT_TIMESTAMP ::timestamp AS right_now;

Now select a timestamp five days from now and alias it as five_days_from_now.

SELECT
	CURRENT_TIMESTAMP::timestamp AS right_now,
    INTERVAL '5 days' + CURRENT_TIMESTAMP AS five_days_from_now;
    
    Finally, let's use a second-level precision with no fractional digits for both the right_now and five_days_from_now fields.
    
    	CURRENT_TIMESTAMP(2)::timestamp AS right_now,
    interval '5 days' + CURRENT_TIMESTAMP(2) AS five_days_from_now;
    
    Using EXTRACT
You can use EXTRACT() and DATE_PART() to easily create new fields in your queries by extracting sub-fields from a source timestamp field.

Now suppose you want to produce a predictive model that will help forecast DVD rental activity by day of the week. You could use the EXTRACT() function with the dow field identifier in our query to create a new field called dayofweek as a sub-field of the rental_date column from the rental table.

You can COUNT() the number of records in the rental table for a given date range and aggregate by the newly created dayofweek column.

Get the day of the week from the rental_date column.

SELECT 
  -- Extract day of week from rental_date
  Extract(dow FROM rental_date) AS dayofweek 
FROM rental 
LIMIT 100;

Count the total number of rentals by day of the week.

-- Extract day of week from rental_date
SELECT 
  EXTRACT(dow FROM rental_date) AS dayofweek, 
  -- Count the number of rentals
  COUNT(rental_date) as rentals 
FROM rental 
GROUP BY 1;

Using DATE_TRUNC
The DATE_TRUNC() function will truncate timestamp or interval data types to return a timestamp or interval at a specified precision. The precision values are a subset of the field identifiers that can be used with the EXTRACT() and DATE_PART() functions. DATE_TRUNC() will return an interval or timestamp rather than a number. For example

SELECT DATE_TRUNC('month', TIMESTAMP '2005-05-21 15:30:30');
Result: 2005-05-01 00;00:00

Now, let's experiment with different precisions and ultimately modify the queries from the previous exercises to aggregate rental activity.

Truncate the rental_date field by year.

-- Truncate rental_date by year
SELECT DATE_TRUNC('year', rental_date) AS rental_year
FROM rental;

Now modify the previous query to truncate the rental_date by month.

-- Truncate rental_date by month
SELECT DATE_TRUNC('month', rental_date) AS rental_month
FROM rental;

Let's see what happens when we truncate by day of the month.

-- Truncate rental_date by day of the month 
SELECT DATE_TRUNC('day', rental_date) AS rental_day 
FROM rental;

Finally, count the total number of rentals by rental_day and alias it as rentals.


SELECT 
  DATE_TRUNC('day', rental_date) AS rental_day,
  -- Count total number of rentals 
  COUNT(rental_date) AS rentals 
FROM rental
GROUP BY 1;

Putting it all together
Many of the techniques you've learned in this course will be useful when building queries to extract data for model training. Now let's use some date/time functions to extract and manipulate some DVD rentals data from our fictional DVD rental store.

In this exercise, you are going to extract a list of customers and their rental history over 90 days. You will be using the EXTRACT(), DATE_TRUNC(), and AGE() functions that you learned about during this chapter along with some general SQL skills from the prerequisites to extract a data set that could be used to determine what day of the week customers are most likely to rent a DVD and the likelihood that they will return the DVD late.

Extract the day of the week from the rental_date column using the alias dayofweek.
Use an INTERVAL in the WHERE clause to select records for the 90 day period starting on 5/1/2005.

SELECT 
  -- Extract the day of week date part from the rental_date
  EXTRACT(dow FROM rental_date) AS dayofweek,
  AGE(return_date, rental_date) AS rental_days
FROM rental AS r 
WHERE 
  -- Use an INTERVAL for the upper bound of the rental_date 
  rental_date BETWEEN CAST('2005-05-01' AS timestamp)
   AND CAST('2005-05-01' AS timestamp) + INTERVAL '90 day';
   
   Finally, use a CASE statement and DATE_TRUNC() to create a new column called past_due which will be TRUE if the rental_days is greater than the rental_duration otherwise, it will be FALSE.
   
   SELECT 
  c.first_name || ' ' || c.last_name AS customer_name,
  f.title,
  r.rental_date,
  -- Extract the day of week date part from the rental_date
  EXTRACT(dow FROM r.rental_date) AS dayofweek,
  AGE(r.return_date, r.rental_date) AS rental_days,
  -- Use DATE_TRUNC to get days from the AGE function
  CASE WHEN DATE_TRUNC('day', AGE(r.return_date, r.rental_date)) > 
  -- Calculate number of d
    f.rental_duration * INTERVAL '1' day 
  THEN TRUE 
  ELSE FALSE END AS past_due 
FROM 
  film AS f 
  INNER JOIN inventory AS i 
  	ON f.film_id = i.film_id 
  INNER JOIN rental AS r 
  	ON i.inventory_id = r.inventory_id 
  INNER JOIN customer AS c 
  	ON c.customer_id = r.customer_id 
WHERE 
  -- Use an INTERVAL for the upper bound of the rental_date 
  r.rental_date BETWEEN CAST('2005-05-01' AS DATE) 
  AND CAST('2005-05-01' AS DATE) + INTERVAL '90 day';


