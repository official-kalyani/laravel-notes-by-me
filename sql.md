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
```SELECT * from movies where imdb_rating IS NOT NULL;```



