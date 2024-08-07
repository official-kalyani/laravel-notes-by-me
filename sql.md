# Chapter:- SQL Basics: Data Retrieval - Single Table
### Module: Retrieve data using text query(SELECT, WHERE, DISTINCT, LIKE)

> Simply print all the movies 
```
	SELECT * from movies;
```
>Get movie title and industry for all the movies
```
SELECT title, industry from movies;

```
>Print all moves from Hollywood 
```
SELECT * from movies where industry="Hollywood";
```
>Print all moves from Bollywood 
```
SELECT * from movies where industry="Bollywood";
```
>Get all the unique industries in the movies database
```
SELECT DISTINCT industry from movies;
```
>Select all movies that starts with THOR
```
SELECT * from movies where title LIKE 'THOR%';
```
>Select all movies that have 'America' word in it. That means to select all captain America movies
```
SELECT * from movies where title LIKE '%America%';

```
>How many hollywood movies are present in the database?
```
SELECT COUNT(*) from movies where industry="Hollywood";
```
>Print all  movies where we don't know the value of the studio
```
 SELECT * FROM movies WHERE studio='';

```
### Module: Retrieve data using numeric query (BETWEEN, IN, ORDER BY, LIMIT, OFFSET)
> Which movies had greater than 9 imdb rating?
```
	SELECT * from movies where imdb_rating>9;

```
>Movies with rating between 6 and 8
```
SELECT * from movies where imdb_rating>=6 and imdb_rating <=8;
SELECT * from movies where imdb_rating BETWEEN 6 AND 8;

```
>Select all movies whose release year can be 2018 or 2019 or 2022
```
-- Approach1:
	SELECT * from movies where release_year=2022 
	or release_year=2019 or release_year=2018;

	-- Approach2:
	SELECT * from movies where release_year IN (2018,2019,2022);


```
> All movies where imdb rating is not available (imagine the movie is just released)
```
SELECT * from movies where imdb_rating IS NULL;
```
> All movies where imdb rating is available 
```
SELECT * from movies where imdb_rating IS NOT NULL;
```
> Print all bollywood movies ordered by their imdb rating
``` 
SELECT *  from movies WHERE industry = "bollywood" ORDER BY imdb_rating ASC;
```
>Print first 5 bollywood movies with highest rating
```
SELECT *  from movies WHERE industry = "bollywood" ORDER BY imdb_rating DESC LIMIT 5;
```
>Select movies starting from second highest rating movie till next 5 movies for bollywood
```
SELECT * 
        from movies WHERE industry = "bollywood"
        ORDER BY imdb_rating DESC LIMIT 5 OFFSET 1;
```
### Module: Summary Analytics (COUNT, MAX, MIN, AVG, GROUP BY)
>How many total movies do we have in our movies table?
```
SELECT COUNT(*) from movies;
```
>Select highest imdb rating for bollywood movies
```
SELECT MAX(imdb_rating) from movies where industry="Bollywood";
```
>Select lowest imdb rating for bollywood movies
```
SELECT MIN(imdb_rating) from movies where industry="Bollywood";
```
>Print average rating of Marvel Studios movies
```
SELECT AVG(imdb_rating) from movies where studio="Marvel Studios";
SELECT ROUND(AVG(imdb_rating),2) from movies where studio="Marvel Studios";
```
>Print min, max, avg rating of Marvel Studios movies
```
SELECT 
	MIN(imdb_rating) as min_rating, 
	MAX(imdb_rating) as max_rating, 
	ROUND(AVG(imdb_rating),2) as avg_rating
FROM movies 
WHERE studio="Marvel Studios";
```
>Print count of movies by industry
```
SELECT 
           industry, count(industry) 
        FROM movies
        GROUP BY industry;
```
>Same thing but add average rating
```
SELECT 
            industry, 
            count(industry) as movie_count,
            avg(imdb_rating) as avg_rating
	FROM movies
	GROUP BY industry;
```
>Count number of movies released by a given production studio
```
SELECT 
	    studio, count(studio) as movies_count 
        from movies WHERE studio != ''
	GROUP BY studio
	ORDER BY movies_count DESC;

```
>What is the average rating of movies per studio and also order them by average rating in descending format?
```
SELECT 
	   studio, 
	   count(studio) as cnt, 
	   round(avg(imdb_rating), 1) as avg_rating 
	from movies WHERE studio != ''
	GROUP BY studio
        order by avg_rating DESC;
```
### Module: HAVING Clause
>Print all the years where more than 2 movies were released
```
SELECT 
           release_year, 
           count(*) as movies_count
	FROM movies    
	GROUP BY release_year
	HAVING movies_count>2
	ORDER BY movies_count DESC;
	
```














