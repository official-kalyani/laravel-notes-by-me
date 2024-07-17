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
