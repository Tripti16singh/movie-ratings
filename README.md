# movie-ratings
"SQL analysis of movie ratings on BigQuery console revealed interesting insights. Using data from multiple sources, queries were performed to identify top-rated genres, highest-grossing movies, and user demographics. Results were visualized through interactive dashboards for easy interpretation."
select * from `Project_Movie.Movie_Review`;
 -- 1.	Which movies have the highest ratings overall?

select max(t.Ratings) Max_ratings,t.MovieName from
 (select count(rating) Ratings,  MovieName from `Project_Movie.Movie_Review`
 group by MovieName)t
 group by MovieName
 order by Max_ratings desc
 limit 5;

 -- 2.	What is the average rating for each genre? Which genre has the highest average rating?
 
select max(t.avg_rating) Max_avg_highest_rating , moviename from 
 (select floor(avg(rating)) avg_rating, moviename from `Project_Movie.Movie_Review`
 group by MovieName)t
 group by moviename
 order by  Max_avg_highest_rating 
 desc
 limit 5;

 -- 3.	Which customers have rated the most movies?
SELECT cust_id, COUNT(DISTINCT movie_id) as num_ratings
FROM `Project_Movie.Movie_Review`
GROUP BY cust_id
ORDER BY num_ratings DESC;

-- 4.	What is the distribution of ratings for each movie? 
select  rating ,count(rating) no_of_rating, moviename from `Project_Movie.Movie_Review`
group by MovieName,Rating
order by no_of_rating desc;

-- Which movies have the most variability in their ratings? - Higher the standard deviation higher the variabiltiy.
SELECT  moviename, round(STDDEV(rating),2) as rating_stddev
FROM `Project_Movie.Movie_Review`
GROUP BY moviename
ORDER BY rating_stddev DESC;

-- 5.	Are there any relationships between the movie genre and the customer's ratings?

SELECT genre, AVG(rating) as avg_rating
FROM `Project_Movie.Movie_Review`
GROUP BY genre
order by avg_rating desc;

-- 7.	Which movies have the most polarizing ratings, i.e., the highest variance in ratings between customers?

SELECT movie_Id, genre, moviename, round(VARIANCE(rating),2) AS rating_variance 
FROM `Project_Movie.Movie_Review` 
GROUP BY 
  movie_Id, genre, moviename 
ORDER BY rating_variance DESC;

-- 8.	What are the top-rated movies in each genre?
select max(rating) as most_rated_movies,genre,moviename from `Project_Movie.Movie_Review`
group by Genre,MovieName
having most_rated_movies = max(Rating);

 --9. Which is the top rated genre?
select count(t.most_Rated) as most_rated_movie_genre ,t.genre from
(select max(rating) as most_Rated, genre, MovieName from `Project_Movie.Movie_Review`
group by Genre,MovieName)t
group by t.genre
order by most_rated_movie_genre desc;

-- 9.	Which genre has the highest average rating?
select round(avg(rating),2) highest_rated , genre from `Project_Movie.Movie_Review`
group by Genre
order by highest_rated desc
limit 1;

-- 10.What is the average rating of each movie, and how does it compare to the average rating of movies in its genre?
select round(avg(m.rating),2) avg_movie_rating, m.moviename,avg_genre_rate, m.Genre from `Project_Movie.Movie_Review` m 
inner join (select round(avg(rating),2) avg_genre_rate , genre from `Project_Movie.Movie_Review`
group by Genre) g
on m.genre = g.genre
group by m.MovieName,m.Genre,g.avg_genre_rate
order by avg_movie_rating desc;

-- 11.	What is the most popular genre among customers who have rated the highest number of movies?
with cte as 
(
    SELECT cust_id, COUNT(*) as num_ratings, genre
    FROM `Project_Movie.Movie_Review`
    GROUP BY cust_id, genre),
ranked_cte AS (
    SELECT cust_id, genre, cte.num_ratings,
        RANK() OVER (PARTITION BY cust_id ORDER BY num_ratings DESC) as rating_rank
    FROM cte)
    
SELECT genre, COUNT(DISTINCT cust_id) as num_customers
FROM ranked_cte
WHERE rating_rank = 1
GROUP BY genre
ORDER BY num_customers DESC; 

-- 12.	How many movies have been rated by each customer, and what is the average rating given by those customers?
select cust_id, count(distinct movie_id) no_movie_rated, round(avg(rating),2) as avg_rating from `Project_Movie.Movie_Review`
group by Cust_Id
order by no_movie_rated desc;

-- 13.	What are the top-rated movies overall, and which genres do they belong to?
select max(rating) max_rated, genre, MovieName from `Project_Movie.Movie_Review`
group by Genre,MovieName;

-- 14.	Which customers have rated the most movies, and what are their top-rated genres?

select cust_id, count(distinct movie_id) most_movies, Genre from `Project_Movie.Movie_Review`
group by Cust_Id, Genre
order by most_movies desc;

-- custmore who have given top ratings to which genre the most number of times.

SELECT cust_id, genre,COUNT(*) AS num_top_ratings
FROM `Project_Movie.Movie_Review`
WHERE rating >= 4
group by cust_id, genre
ORDER BY num_top_ratings DESC
LIMIT 10;


-- 15.	What is the average rating for each genre of movie, and which genres have the highest and lowest ratings?
select genre,round(avg(rating),2) avg_rate from `Project_Movie.Movie_Review`
group by Genre
having  AVG(rating) = (SELECT MAX(avg_rating) FROM ( SELECT genre, AVG(rating) AS avg_rating FROM `Project_Movie.Movie_Review` GROUP BY genre) AS avg_ratings)
or
AVG(rating) = (SELECT min(avg_rating) FROM ( SELECT genre, AVG(rating) AS avg_rating FROM `Project_Movie.Movie_Review` GROUP BY genre) AS avg_ratings)
ORDER BY
  avg_rate DESC;

-- 16.	Are there any outliers in the data, such as movies that have extremely high or low ratings compared to others in their genre? 
-- max the rating_diff max is the variance hence, outlier

SELECT m.Genre,m.moviename,m.rating,avg_rating, round(r.avg_rating - m.rating,2)  rating_diff
FROM `Project_Movie.Movie_Review` m
INNER JOIN ( SELECT genre, round(AVG(rating),2) AS avg_rating FROM `Project_Movie.Movie_Review`
GROUP BY genre) r ON m.genre = r.genre
ORDER BY ABS(r.avg_rating - m.rating) desc;





