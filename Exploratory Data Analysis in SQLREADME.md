Count missing values
Which column of fortune500 has the most missing values? To find out, you'll need to check each column individually, although here we'll check just three.

Course Note: While you're unlikely to encounter this issue during this exercise, note that if you run a query that takes more than a few seconds to execute, your session may expire or you may be disconnected from the server. You will not have this issue with any of the exercise solutions, so if your session expires or disconnects, there's an error with your query.

First, figure out how many rows are in fortune500 by counting them.
-- Select the count of the number of rows
SELECT count(*)
  FROM fortune500;
  
  Subtract the count of the non-NULL ticker values from the total number of rows; alias the difference as missing.
  
  -- Select the count of ticker, 
-- subtract from the total number of rows, 
-- and alias as missing
SELECT count(*) - count(ticker) AS missing
  FROM fortune500;
  
  Repeat for the profits_change column.
  
  SELECT count(*) - count(profits_change) AS missing
FROM fortune500;  

Repeat for the industry column.
-- Select the count of industry, 
-- subtract from total number of rows, and alias as missing
SELECT count(*) - count(industry) AS missing
FROM fortune500;

Foreign keys
Recall that foreign keys reference another row in the database via a unique ID. Values in a foreign key column are restricted to values in the referenced column OR NULL.

Using what you know about foreign keys, why can't the tag column in the tag_type table be a foreign key that references the tag column in the stackoverflow table?

Remember, you can reference the slides using the icon in the upper right of the screen to review the requirements for a foreign key.
Possible Answers

stackoverflow.tag is not a primary key

tag_type.tag contains NULL values

stackoverflow.tag contains duplicate values

tag_type.tag does not contain all the values in stackoverflow.tag

Read an entity relationship diagram
The information you need is sometimes split across multiple tables in the database.

What is the most common stackoverflow tag_type? What companies have a tag of that type?

To generate a list of such companies, you'll need to join three tables together.

Reference the entity relationship diagram as needed when determining which columns to use when joining tables.

-- Count the number of tags with each type
SELECT type, COUNT(type) AS count
  FROM tag_type
 -- To get the count for each type, what do you need to do?
 GROUP BY type
 -- Order the results with the most common
 -- tag types listed first
 ORDER BY count DESC;
 
 Join the tag_company, company, and tag_type tables, keeping only mutually occurring records.
Select company.name, tag_type.tag, and tag_type.type for tags with the most common type from the previous step.

-- Select the 3 columns desired
SELECT company.name, tag_type.tag, tag_type.type
  FROM company
  	   -- Join to the tag_company table
       INNER JOIN tag_company 
       ON company.id = tag_company.company_id
       -- Join to the tag_type table
       INNER JOIN tag_type
       ON tag_company.tag = tag_type.tag
  -- Filter to most common type
  WHERE type='cloud';
  
  Coalesce
The coalesce() function can be useful for specifying a default or backup value when a column contains NULL values.

coalesce() checks arguments in order and returns the first non-NULL value, if one exists.

coalesce(NULL, 1, 2) = 1
coalesce(NULL, NULL) = NULL
coalesce(2, 3, NULL) = 2
In the fortune500 data, industry contains some missing values. Use coalesce() to use the value of sector as the industry when industry is NULL. Then find the most common industry.

Use coalesce() to select the first non-NULL value from industry, sector, or 'Unknown' as a fallback value.
Alias the result of the call to coalesce() as industry2.
Count the number of rows with each industry2 value.
Find the most common value of industry2.

-- Use coalesce
SELECT coalesce(industry, sector, 'Unknown') AS industry2,
       -- Don't forget to count!
       count(*)
  FROM fortune500 
-- Group by what? (What are you counting by?)
 GROUP BY industry2
-- Order results to see most common first
 ORDER BY count(*) DESC
-- Limit results to get just the one value you want
 LIMIT 1;
 
 Coalesce with a self-join
You previously joined the company and fortune500 tables to find out which companies are in both tables. Now, also include companies from company that are subsidiaries of Fortune 500 companies as well.

To include subsidiaries, you will need to join company to itself to associate a subsidiary with its parent company's information. To do this self-join, use two different aliases for company.

coalesce will help you combine the two ticker columns in the result of the self-join to join to fortune500.

Join company to itself to add information about a company's parent to the original company's information.
Use coalesce to get the parent company ticker if available and the original company ticker otherwise.
INNER JOIN to fortune500 using the ticker.
Select original company name, fortune500 title and rank.

SELECT company_original.name, title, rank
  -- Start with original company information
  FROM company AS company_original
       -- Join to another copy of company with parent
       -- company information
	   LEFT JOIN company AS company_parent
       ON company_original.parent_id = company_parent.id 
       -- Join to fortune500, only keep rows that match
       INNER JOIN fortune500 
       -- Use parent ticker if there is one, 
       -- otherwise original ticker
       ON coalesce(company_parent.ticker, 
                   company_original.ticker) = 
             fortune500.ticker
 -- For clarity, order by rank
 ORDER BY rank; 
 
 Effects of casting
When you cast data from one type to another, information can be lost or changed. See how the casting changes values and practice casting data using the CAST() function and the :: syntax.

SELECT CAST(value AS new_type);

SELECT value::new_type;

-- Select the original value
SELECT profits_change, 
	   -- Cast profits_change
       CAST(profits_change AS integer) AS profits_change_int
  FROM fortune500;
  
Compare the results of casting of dividing the integer value 10 by 3 to the result of dividing the numeric value 10 by 3.
  
  -- Divide 10 by 3
SELECT 10/3, 
       -- Cast 10 as numeric and divide by 3
       10::numeric/3;
       
Now cast numbers that appear as text as numeric.
Note: 1e3 is scientific notation.

SELECT '3.2'::numeric,
       '-123'::numeric,
       '1e3'::numeric,
       '1e-3'::numeric,
       '02314'::numeric,
       '0002'::numeric;
       
Summarize the distribution of numeric values
Was 2017 a good or bad year for revenue of Fortune 500 companies? Examine how revenue changed from 2016 to 2017 by first looking at the distribution of revenues_change and then counting companies whose revenue increased.

Use GROUP BY and count() to examine the values of revenues_change.
Order the results by revenues_change to see the distribution.

-- Select the count of each value of revenues_change
SELECT revenues_change, COUNT(revenues_change)
  FROM fortune500
 GROUP BY revenues_change
 -- order by the values of revenues_change
 ORDER BY revenues_change;
 
 Repeat step 1, but this time, cast revenues_change as an integer to reduce the number of different values.
 
 -- Select the count of each revenues_change integer value
SELECT revenues_change::integer, COUNT(*)
  FROM fortune500
 GROUP BY revenues_change::integer
 -- order by the values of revenues_change
 ORDER BY revenues_change;
 
 How many of the Fortune 500 companies had revenues increase in 2017 compared to 2016? To find out, count the rows of fortune500 where revenues_change indicates an increase.
 
 -- Count rows 
SELECT COUNT(*)
  FROM fortune500
 -- Where...
 WHERE revenues_change > 0;
 


