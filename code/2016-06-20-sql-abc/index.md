# SQL 速查

All the resources from [codecademy](https://www.codecademy.com/learn)

<!--more-->

### Basic Operations
celebs:

$$
\begin{array}{c|c|c|c}
\text{id} & \text{name} & \text{age} & \text{twitter_handle} \\\
\hline
1 & \text{Justin Bieber} & 22 &  \\\
2 & \text{Beyonce Knowles} & 33 &  \\\
3 & \text{Jeremy Lin} & 26 &  \\\
\end{array}
$$

1. Table Creation

        CREATE TABLE celebs (id INTEGER, name TEXT, age INTEGER, twitter_handle TEXT);

2. Changing values

        UPDATE celebs SET age = 22 WHERE id = 1;

3. Inserting
 Insert a row:
 
        INSERT INTO celebs (id, name, age) VALUES (1, 'Justin Bieber', 21);
 Insert a column:

        ALTER TABLE celebs ADD COLUMN twitter_handle TEXT; 
       
4. Deletion

        DELETE FROM celebs WHERE twitter_handle IS NULL;


---

### Queries
movies:

$$
\begin{array}{c|c|c|c|c}
\text{id} & \text{name} & \text{genre} & \text{year} & \text{imdb_rating} \\\
\hline
1 & \text{Avatar} & \text{action} & 2009 & 7.9 \\\
2 & \text{Jurassic World} & \text{action} & 2015 & 7.3 \\\
3 & \text{The Avengers} & \text{action} & 2012 & 8.1 \\\
\end{array}
$$

1. Select one or more column:
 
        SELECT * FROM movies;
        SELECT name, imdb_rating FROM movies;

2. Select distinct elements

        SELECT DISTINCT genre FROM movies;

3. Select values specifying rules:
 Sinple rules:
 
        SELECT * FROM movies WHERE imdb_rating > 8;
 Regular expression-like rules:
 
        SELECT * FROM movies WHERE name LIKE 'Se_en';
        SELECT * FROM movies WHERE name LIKE 'A%';
        SELECT * FROM movies WHERE name BETWEEN 'A' AND 'J';
 Combine different rules:
 
        SELECT * FROM movies WHERE genre = 'comedy' OR year < 1980;

4. Ordering:

        SELECT * FROM movies
        ORDER BY imdb_rating DESC;
 DESC is a keyword in SQL that is used with ORDER BY to sort the results in descending order, ASC works similarly

        SELECT * FROM movies
        ORDER BY imdb_rating ASC
        LIMIT 3;
 LIMIT specifies that the result table returned can not have more than three rows

---

### Aggregate Functions
fake_apps:

$$
\begin{array}{c|c|c|c|c}
\text{id} & \text{name} & \text{category} & \text{downloads} & \text{price} \\\
\hline
1 & \text{siliconphase} & \text{Productivity	} & 17193 & 0.0 \\\
2 & \text{Donzolab} & \text{Education} & 4259 & 0.99 \\\
3 & \text{Ittechi} & \text{Reference} & 3874 & 0.0 \\\
\end{array}
$$

1. Counting

        SELECT COUNT(*) FROM fake_apps;
        SELECT price, COUNT(*) FROM fake_apps GROUP BY price;
        SELECT price, COUNT(*) FROM fake_apps WHERE downloads > 20000 GROUP BY price;
 `COUNT()` is a function that takes the name of a column as an argument and counts the number of rows where the column is not NULL. Here, we want to count every row so we pass * as an argument.

2. SUM, MAX, AVG, ROUND
 Return the sum of downloads under each category:
 
        SELECT category, SUM(downloads) FROM fake_apps GROUP BY category;
        
 Return the name and category of the most frequently downloaded apps in each category:

        SELECT name, category, MAX(downloads) FROM fake_apps GROUP BY category;
 Calculate the average number of downloads at each price

        SELECT price, AVG(downloads) FROM fake_apps GROUP BY price;
 Rounding the average number of downloads to two decimal(default is 0):
 
        SELECT price, ROUND(AVG(downloads), 2) FROM fake_apps GROUP BY price;

---

### Multiple Tables
albums:

$$
\begin{array}{c|c|c|c}
\text{id} & \text{name} & \text{artist_id} & \text{year} \\\
\hline
1 & \text{A Hard Days Night} & 1 & 1964 \\\
2 & \text{Elvis Presley} & 2 & 1956 \\\
3 & \text{Unorthodox Jukebox} &  & 2012 \\\
\end{array}
$$

artists:

$$
\begin{array}{c|c}
\text{id} & \text{name} \\\
\hline
1 & \text{The Beatles} \\\
2 & \text{Elvis Presley} \\\
3 & \text{Unorthodox Jukebox} \\\
\end{array}
$$

1. Primary Key

        CREATE TABLE artists(id INTEGER PRIMARY KEY, name TEXT)
 A primary key serves as a unique identifier for each row. By specifying that the id column is the PRIMARY KEY, SQL makes sure that:
  - None of the values in this column are NULL
  - Each value in this column is unique
 
2. Selection from multiple tables:
 
        SELECT albums.name, albums.year, artists.name FROM albums, artists;
   Unfortunately, this operation would return a table which combines every row of the artists table with every row of the albums table.
   
3. Join 2 different tables through rule:

        SELECT * FROM albums JOIN artists ON albums.artist_id = artists.id;
   Here, artist_id is a foreign key in the albums table. A foreign key is a column that contains the primary key of another table in the database.

        SELECT * FROM albums LEFT JOIN artists ON albums.artist_id = artists.id;
 Every row in the left table is returned in the result set, and if the join condition is not met, then NULL values are used to fill. The left table is simply the first table that appears in the statement. 

4. Set alias

        SELECT
          albums.name AS 'Album',
          albums.year,
          artists.name AS 'Artist'
        FROM
          albums
        JOIN artists ON
          albums.artist_id = artists.id
        WHERE
          albums.year > 1980;
   `AS` is a keyword in SQL that allows you to use an alias. The aliases only appear in the result set,  columns have not been renamed
> Written with [StackEdit](https://stackedit.io/).
