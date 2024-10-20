# Netflix-Content-Analysis-with-SQL
[link to the dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

********************
## SQL Functions and Clauses Overview
- **CREATE TABLE**: Defines a new table structure for storing Netflix data.
- **SELECT**: Retrieves data from a database.
- **COUNT()**: Counts the number of rows that match a specified condition.
- **DISTINCT**: Returns unique values from a specified column.
- **GROUP BY**: Groups rows that have the same values in specified columns into summary rows.
- **ORDER BY**: Sorts the result set based on specified column(s).
- **ILIKE**: Case-insensitive pattern matching in PostgreSQL.
- **EXTRACT()**: Retrieves subparts of a date (year, month, day).
- **TO_DATE()**: Converts a string to a date based on the specified format.
- **SPLIT_PART()**: Splits a string into parts based on a specified delimiter.
- **UNNEST()**: Expands an array into a set of rows.

## Queries

### Create new table
```sql
CREATE TABLE netflix
(
 show_id	  VARCHAR(6),
 type	      VARCHAR(10),
 title	     VARCHAR(150),
 director	 VARCHAR(208),
 casts	     VARCHAR(1000),
 country	 VARCHAR(150),
 date_added	 VARCHAR(50),
 release_year  INT,	
 rating	  VARCHAR(10),
 duration	VARCHAR(15),
 listed_in	 VARCHAR(100),
 description  VARCHAR(250)
);
```
### 1. Display All Content
```sql
SELECT * FROM netflix;

```
### 2. Count Total Rows in the Database
```sql
SELECT COUNT(*) AS total_rows
FROM netflix;
```
<img width="105" alt="image" src="https://github.com/user-attachments/assets/2798140b-48a7-40ed-a5c9-726f93e2c286">

### 3. Distinct Content Types
```sql
SELECT DISTINCT type
FROM netflix;
```
<img width="149" alt="image" src="https://github.com/user-attachments/assets/e3f5ce69-d68b-4df2-85ea-8133f7bab111">

### 4. Count Total Content by Type
```sql
SELECT type, COUNT(*) AS total_content
FROM netflix
GROUP BY type;
```
<img width="222" alt="image" src="https://github.com/user-attachments/assets/bc348f22-7337-4b28-980e-49487e338dc4">

### 5. Most Common Rating for Movies and TV Shows
```sql
SELECT DISTINCT ON (type) type, rating, COUNT(*) AS rating_count
FROM netflix
GROUP BY type, rating
ORDER BY type, rating_count DESC;
```
<img width="335" alt="image" src="https://github.com/user-attachments/assets/9b30a038-3bcb-4a18-babc-9fffdf5d6402">

### 6. Handle Ties for Ratings
```sql
WITH RankedRatings AS (
    SELECT type, rating, COUNT(*) AS rating_count,
           RANK() OVER (PARTITION BY type ORDER BY COUNT(*) DESC) AS rank
    FROM netflix
    GROUP BY type, rating
)
SELECT type, rating, rating_count
FROM RankedRatings
WHERE rank = 1;
```
<img width="332" alt="image" src="https://github.com/user-attachments/assets/bb762233-9be8-4191-b2d5-d5a0cc5e2a47">

### 7. Display Movies Released in 2020
```sql
SELECT show_id, type, title, release_year
FROM netflix
WHERE type = 'Movie' AND release_year = 2020;
```
<img width="661" alt="image" src="https://github.com/user-attachments/assets/84d0f892-5d23-4150-be28-4bab634564f2">

### 8. Top 5 Countries with Most Content on Netflix
```sql
SELECT
       UNNEST(string_to_array(country, ', ')) AS individual_country,
       COUNT(show_id) AS total_content
FROM netflix
GROUP BY individual_country
ORDER BY total_content DESC
LIMIT 5;
```
<img width="212" alt="image" src="https://github.com/user-attachments/assets/2dfd59ab-efd8-4f89-b088-f4e2eb97743f">

### 9. Identify the Longest Movie
```sql
SELECT show_id, type, title, duration, listed_in
FROM netflix
WHERE type = 'Movie' AND duration IS NOT NULL
ORDER BY CAST(REGEXP_REPLACE(duration, '[^0-9]', '', 'g') AS INTEGER) DESC
LIMIT 1;
```
<img width="703" alt="image" src="https://github.com/user-attachments/assets/237fe625-4a74-4ab9-9ed0-a12af5e5fb63">

### 10. Content Added in the Last 5 Years
```sql
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
```
### 11. Display All Movies and TV Shows by Director name
```sql
SELECT *
FROM netflix
WHERE director IS NOT NULL AND director ILIKE '%Vince Gilligan%';
```
<img width="770" alt="image" src="https://github.com/user-attachments/assets/68179d23-9b28-466c-abc7-d6483219256c">

### 12. Display All Movies or TV Shows with Specific Cast member
```sql
SELECT type, title, date_added, release_year, listed_in
FROM netflix
WHERE casts ILIKE '%Anthony Hopkins%';
```
<img width="701" alt="image" src="https://github.com/user-attachments/assets/f4962e11-ed25-4570-8564-e8b408339a2e">

### 13. List All TV Shows with More Than 5 Seasons
```sql
SELECT show_id, type, title, country, date_added, duration
FROM netflix
WHERE type = 'TV Show' AND CAST(SPLIT_PART(duration, ' ', 1) AS INTEGER) > 5
ORDER BY CAST(SPLIT_PART(duration, ' ', 1) AS INTEGER) DESC;
```
<img width="797" alt="image" src="https://github.com/user-attachments/assets/1d0b5db9-e456-4307-8349-fd6e4646aef1">

### 14. Number of Content Items in Each Genre
```sql
SELECT UNNEST(string_to_array(listed_in, ', ')) AS Genre,
       COUNT(show_id) AS Total_items
FROM netflix
GROUP BY Genre
ORDER BY Total_items DESC;
```
<img width="264" alt="image" src="https://github.com/user-attachments/assets/30eb21e9-5547-42c2-b0e8-ebbb3908e02d">

### 15. Number of Netflix Content Added by Year related to United States
```sql
SELECT CAST(SPLIT_PART(date_added, ', ', 2) AS INTEGER) AS year_added,
       COUNT(show_id) AS total_content_added
FROM netflix
WHERE country ILIKE '%United States%'
GROUP BY year_added
ORDER BY year_added;
```
<img width="212" alt="image" src="https://github.com/user-attachments/assets/3fbe4c52-92e7-4881-a777-71bafc21bcc3">

### 16. Number of Netflix Content Added by Year in the Last 5 Years related to US
```sql
SELECT EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD, YYYY')) AS year_added,
       COUNT(show_id) AS total_content_added
FROM netflix
WHERE country ILIKE '%United States%'
      AND EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD, YYYY')) >= EXTRACT(YEAR FROM CURRENT_DATE) - 5
GROUP BY year_added
ORDER BY year_added;

```
<img width="217" alt="image" src="https://github.com/user-attachments/assets/4f1f62e6-d5c2-4b0c-9370-8528e6929557">

### 17. List All Documentaries
```sql
SELECT show_id, title, listed_in
FROM netflix
WHERE listed_in ILIKE '%Documentaries%'
   AND type = 'Movie';
```
<img width="803" alt="image" src="https://github.com/user-attachments/assets/762f3e0e-b51d-42d6-9989-9a9cb3b35e61">

### 18. List of Content Without a Director
```sql
SELECT *
FROM netflix
WHERE director IS NULL;
```
### 19. Number of Movies with Morgan Freeman in the Past Decade
```sql
SELECT COUNT(*)
FROM netflix
WHERE casts ILIKE '%Morgan Freeman%' AND type = 'Movie'
      AND EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD, YYYY')) >= EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```
### 20. Actors Appearing in Netflix Movies Associated with Egypt
```sql
SELECT UNNEST(string_to_array(casts, ', ')) AS Actor,
       COUNT(*) AS total_appearance_on_netflix
FROM netflix
WHERE country ILIKE '%Egypt%' AND type = 'Movie'
GROUP BY Actor
ORDER BY total_appearance_on_netflix DESC;

```
<img width="358" alt="image" src="https://github.com/user-attachments/assets/54b80faf-512e-42ed-a564-a3796d5ba834">

### 21. Kid-Friendliness Labeling Based on Description
```sql
SELECT
  CASE
    WHEN description ILIKE '%family%' OR description ILIKE '%children%' OR description ILIKE '%adventure%' 
         OR description ILIKE '%animated%' OR description ILIKE '%fun%' OR description ILIKE '%wholesome%' THEN 'Recommended for Kids'
    WHEN description ILIKE '%violence%' OR description ILIKE '%death%' OR description ILIKE '%mature%' 
         OR description ILIKE '%kill%' OR description ILIKE '%fear%' OR description ILIKE '%gore%' 
         OR description ILIKE '%blood%' OR description ILIKE '%sex%' OR description ILIKE '%abuse%' 
         OR description ILIKE '%screams%' THEN 'Not Recommended for Kids'
    ELSE 'Unlabeled'
  END AS Kid_Friendliness_Label,
  COUNT(*) AS total_content
FROM netflix
GROUP BY Kid_Friendliness_Label
ORDER BY total_content DESC;
```
<img width="212" alt="image" src="https://github.com/user-attachments/assets/32a0af25-2f8b-4c42-81e2-e7f50a506165">


________________________________________

NAME : Hamza Thamlaoui

Email : hamzathamleoui@gmail.com

Linkedin : www.linkedin.com/in/hamza-thamlaoui-51220a264

