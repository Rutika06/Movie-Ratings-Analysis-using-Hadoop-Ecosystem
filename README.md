# Movie Ratings Analysis using Hadoop Ecosystem

A comprehensive big data analytics project that leverages the Hadoop ecosystem to analyze movie ratings and generate insights through interactive visualizations.

## 📊 Project Overview

This project demonstrates end-to-end big data analytics by processing movie rating datasets using HDFS and Hive, performing analytical queries, and visualizing results through modern BI tools.

## 🛠️ Tech Stack

- **HDFS** - Distributed storage for large datasets
- **Apache Hive** - Data warehousing and SQL-like querying
- **Power BI / Tableau** - Data visualization and dashboard creation
- **Hadoop** - Distributed computing framework

## 📁 Dataset

The project uses publicly available movie datasets:

- **IMDb Dataset** - Comprehensive movie information and ratings
- **MovieLens Dataset** - User ratings and movie metadata (recommended for demos)

Dataset features include:
- Movie titles and IDs
- User ratings
- Genres
- Release years
- Movie metadata

## 🎯 Project Objectives

Analyze movie ratings data to extract meaningful insights:

1. Identify top-rated movies
2. Analyze rating trends across genres
3. Track movie production trends over time
4. Create visual dashboards for stakeholder reporting

## 🚀 Getting Started

### Prerequisites

```bash
# Required installations
- Hadoop 3.x or higher
- Apache Hive 3.x or higher
- Java 8 or higher
- Power BI Desktop / Tableau Desktop
```

### Installation Steps

1. **Clone the repository**
```bash
git clone https://github.com/yourusername/Movie-Ratings-Analysis-using-Hadoop-Ecosystem.git
cd Movie-Ratings-Analysis-using-Hadoop-Ecosystem
```

2. **Download the dataset**
```bash
# Download MovieLens dataset
wget http://files.grouplens.org/datasets/movielens/ml-latest-small.zip
unzip ml-latest-small.zip
```

3. **Start Hadoop services**
```bash
start-dfs.sh
start-yarn.sh
```

4. **Start Hive**
```bash
hive --service metastore &
hive
```

## 📋 Implementation Steps

### Step 1: Upload Dataset to HDFS

Create HDFS directories and upload data:

```bash
# Create directory structure
hdfs dfs -mkdir -p /user/hadoop/movie_analysis/input

# Upload datasets to HDFS
hdfs dfs -put movies.csv /user/hadoop/movie_analysis/input/
hdfs dfs -put ratings.csv /user/hadoop/movie_analysis/input/

# Verify upload
hdfs dfs -ls /user/hadoop/movie_analysis/input/
```

### Step 2: Create Hive Tables

Launch Hive and create database:

```sql
-- Create database
CREATE DATABASE IF NOT EXISTS movie_db;
USE movie_db;

-- Create movies table
CREATE EXTERNAL TABLE IF NOT EXISTS movies (
    movieId INT,
    title STRING,
    genres STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/user/hadoop/movie_analysis/input/'
TBLPROPERTIES ("skip.header.line.count"="1");

-- Create ratings table
CREATE EXTERNAL TABLE IF NOT EXISTS ratings (
    userId INT,
    movieId INT,
    rating DOUBLE,
    timestamp BIGINT
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/user/hadoop/movie_analysis/input/'
TBLPROPERTIES ("skip.header.line.count"="1");
```

### Step 3: Run Analytical Queries

#### Query 1: Top 10 Movies by Average Rating

```sql
CREATE TABLE top_movies AS
SELECT 
    m.title,
    AVG(r.rating) as avg_rating,
    COUNT(r.rating) as rating_count
FROM movies m
JOIN ratings r ON m.movieId = r.movieId
GROUP BY m.title
HAVING COUNT(r.rating) >= 50  -- Filter movies with at least 50 ratings
ORDER BY avg_rating DESC
LIMIT 10;
```

#### Query 2: Average Rating per Genre

```sql
CREATE TABLE genre_ratings AS
SELECT 
    genre,
    AVG(rating) as avg_rating,
    COUNT(*) as total_ratings
FROM (
    SELECT 
        r.rating,
        genre_split.genre
    FROM ratings r
    JOIN movies m ON r.movieId = m.movieId
    LATERAL VIEW explode(split(m.genres, '\\|')) genre_split as genre
) genre_data
GROUP BY genre
ORDER BY avg_rating DESC;
```

#### Query 3: Year-wise Number of Movies Released

```sql
CREATE TABLE movies_by_year AS
SELECT 
    CAST(regexp_extract(title, '\\((\\d{4})\\)', 1) AS INT) as release_year,
    COUNT(*) as movie_count
FROM movies
WHERE regexp_extract(title, '\\((\\d{4})\\)', 1) != ''
GROUP BY regexp_extract(title, '\\((\\d{4})\\)', 1)
ORDER BY release_year;
```

### Step 4: Export Results for Visualization

Export query results to local filesystem:

```sql
-- Export from Hive
INSERT OVERWRITE LOCAL DIRECTORY '/home/hadoop/output/top_movies'
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
SELECT * FROM top_movies;

INSERT OVERWRITE LOCAL DIRECTORY '/home/hadoop/output/genre_ratings'
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
SELECT * FROM genre_ratings;

INSERT OVERWRITE LOCAL DIRECTORY '/home/hadoop/output/movies_by_year'
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
SELECT * FROM movies_by_year;
```

Or export directly from HDFS:

```bash
hdfs dfs -getmerge /user/hive/warehouse/movie_db.db/top_movies top_movies.csv
hdfs dfs -getmerge /user/hive/warehouse/movie_db.db/genre_ratings genre_ratings.csv
hdfs dfs -getmerge /user/hive/warehouse/movie_db.db/movies_by_year movies_by_year.csv
```

### Step 5: Create Visualizations

#### Using Power BI:

1. Open Power BI Desktop
2. Click **Get Data** → **Text/CSV**
3. Import the exported CSV files
4. Create visualizations:
   - **Bar Chart**: Top 10 movies by rating
   - **Horizontal Bar Chart**: Average rating by genre
   - **Line Chart**: Movies released year-wise
   - **Cards**: Total movies, total ratings, average rating

#### Using Tableau:

1. Open Tableau Desktop
2. Connect to **Text File** data source
3. Import CSV files
4. Build dashboard with:
   - Bar charts for top movies
   - Tree map for genre analysis
   - Time series for yearly trends
   - KPI indicators

## 📈 Expected Insights

- **Top Performers**: Identify highest-rated movies with significant user engagement
- **Genre Trends**: Understand which genres consistently receive better ratings
- **Production Trends**: Track how movie production volume changes over decades
- **Rating Patterns**: Discover rating distribution and user behavior


