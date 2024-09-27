# Netflix-Data-Cleaning-and-Analysis using SQL And Python(Pandas)

# Overview
  This project involves data cleaning and analysis of Netflix content using SQL, focusing on various queries to extract meaningful insights.
  
# End-to-End Project Workflow
  1.The Netflix Data is sourced from kaggle, Extracted with Python(Pandas),Perform some operations to get an understanding of data like total columns,its data types,null values availability in columns. 
  2.Load the raw data to sql server and perform Data cleaning and transformation.
  3.Once the data is cleaned then perform some data analysis using SQL.

# Key Analysis:
  1.Director Analysis: Count of movies and TV shows created by directors working in both formats.
  2.Country Insights: Identified the country with the highest number of comedy movies.
  3.Yearly Trends: Determined directors with the most movie releases per year.
  4.Genre Duration: Calculated average movie durations by genre.
  5.Genre-specific Directors: Identified directors who created both comedy and horror movies.
  
# Data Cleaning And Transformation:
1.Handling Foreign Character- To handle this changed data type of title in schema from VARCHAR TO NVARCHAR
      
       [title] [nvarchar](200) NULL

2.Removing Duplicates-

```
  select show_id,COUNT(*) 
  from netflix_raw
  group by show_id 
  having COUNT(*)>1

  select * from netflix_raw
  where concat(upper(title),type)  in (
  select concat(upper(title),type) 
  from netflix_raw
  group by upper(title) ,type
  having COUNT(*)>1
  )
  order by title

 with cte as (
 select * 
 ,ROW_NUMBER() over(partition by title , type order by show_id) as rn
 from netflix_raw
 )
 select show_id,type,title,cast(date_added as date) as date_added,release_year
,rating,case when duration is null then rating else duration end as duration,description
 into netflix
 from cte ;
```  

3.Data Type Coverstion--Changing data type of date added column as date
  
```
select show_id,type,title,cast(date_added as date) as date_added,
release_year
,rating,case when duration is null then rating else duration end as duration,
description
into netflix
from cte;
```

4.New Dimension Tables for countries and listed-in

 ```
  select show_id , trim(value) as genre
  into netflix_genre
  from netflix_raw
  cross apply string_split(listed_in,',');
```
```
 select show_id , trim(value) as countries
 into netflix_countries
 from netflix_raw
 cross apply string_split(country,',');
```



# Netflix data analysis

 1] for each director count the no of movies and tv shows created by them in separate columns 
for directors who have created tv shows and movies both

 ```
 select nd.director 
 ,COUNT(distinct case when n.type='Movie' then n.show_id end) as no_of_movies
 ,COUNT(distinct case when n.type='TV Show' then n.show_id end) as no_of_tvshow
 from netflix n
 inner join netflix_directors nd on n.show_id=nd.show_id
 group by nd.director
 having COUNT(distinct n.type)>1;
```


 2] which country has highest number of comedy movies 
```
select  top 1 nc.country , COUNT(distinct ng.show_id ) as no_of_movies
from netflix_genre ng
inner join netflix_country nc on ng.show_id=nc.show_id
inner join netflix n on ng.show_id=nc.show_id
where ng.genre='Comedies' and n.type='Movie'
group by  nc.country
order by no_of_movies desc;
```


 3] for each year (as per date added to netflix), which director has maximum number of movies released
```
with cte as (
select nd.director,YEAR(date_added) as date_year,count(n.show_id) as no_of_movies
from netflix n
inner join netflix_directors nd on n.show_id=nd.show_id
where type='Movie'
group by nd.director,YEAR(date_added)
)
, cte2 as (
select *
, ROW_NUMBER() over(partition by date_year order by no_of_movies desc, director) as rn
from cte
--order by date_year, no_of_movies desc
)
select * from cte2 where rn=1
```



 4] what is average duration of movies in each genre
```
select ng.genre , avg(cast(REPLACE(duration,' min','') AS int)) as avg_duration
from netflix n
inner join netflix_genre ng on n.show_id=ng.show_id
where type='Movie'
group by ng.genre;
```

 5]  find the list of directors who have created horror and comedy movies both.
    display director names along with number of comedy and horror movies directed by them 
```
select nd.director
, count(distinct case when ng.genre='Comedies' then n.show_id end) as no_of_comedy 
, count(distinct case when ng.genre='Horror Movies' then n.show_id end) as no_of_horror
from netflix n
inner join netflix_genre ng on n.show_id=ng.show_id
inner join netflix_directors nd on n.show_id=nd.show_id
where type='Movie' and ng.genre in ('Comedies','Horror Movies')
group by nd.director
having COUNT(distinct ng.genre)=2;
```


# Conclusion: 

This project demonstrates a robust application of SQL and Python in real-world data analysis, emphasizing practical data cleaning, manipulation, and visualization skills that are crucial for any data-driven decision-making process.


