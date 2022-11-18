pm566_HW3
================
Yuhong Hu
2022-11-18

# HPC

## Q1

Rewrite the following R functions to make them faster. It is OK (and
recommended) to take a look at Stackoverflow and Google

``` r
# Total row sums
fun1 <- function(mat) {
  n <- nrow(mat)
  ans <- double(n) 
  for (i in 1:n) {
    ans[i] <- sum(mat[i, ])
  }
  ans
}


fun1alt <- function(mat) {
  rowSums(mat)
}


# Cumulative sum by row
fun2 <- function(mat) {
  n <- nrow(mat)
  k <- ncol(mat)
  ans <- mat
  for (i in 1:n) {
    for (j in 2:k) {
      ans[i,j] <- mat[i, j] + ans[i, j - 1]
    }
  }
  ans
}


fun2alt <- function(mat) {
  t(apply(mat, 1, cumsum)) }


# Use the data with this code
set.seed(2315)
dat <- matrix(rnorm(200 * 100), nrow = 200)

# Test for the first
fun1comp <- microbenchmark::microbenchmark(
  fun1(dat),
  fun1alt(dat), check = "equivalent"
)

print(fun1comp,unit = "relative")
```

    ## Unit: relative
    ##          expr      min       lq    mean   median      uq       max neval cld
    ##     fun1(dat) 31.10169 29.99605 12.9998 29.25856 28.3442 0.3137935   100   b
    ##  fun1alt(dat)  1.00000  1.00000  1.0000  1.00000  1.0000 1.0000000   100  a

``` r
# Test for the second
fun2comp <- microbenchmark::microbenchmark(
  fun2(dat),
  fun2alt(dat), check = "equivalent"
)

print(fun2comp,unit = "relative")
```

    ## Unit: relative
    ##          expr      min       lq    mean   median     uq      max neval cld
    ##     fun2(dat) 4.484043 3.783578 3.00285 3.715776 3.6952 1.244529   100   b
    ##  fun2alt(dat) 1.000000 1.000000 1.00000 1.000000 1.0000 1.000000   100  a

The last argument, check = “equivalent”, is included to make sure that
the functions return the same result.

## Q2 Make things run faster with parallel computing

The following function allows simulating PI

``` r
sim_pi <- function(n = 1000, i = NULL) {
  p <- matrix(runif(n*2), ncol = 2)
  mean(rowSums(p^2) < 1) * 4
}

# Here is an example of the run
set.seed(156)
sim_pi(1000) # 3.132
```

    ## [1] 3.132

In order to get accurate estimates, we can run this function multiple
times, with the following code:

``` r
# This runs the simulation a 4,000 times, each with 10,000 points
set.seed(1231)
system.time({
  ans <- unlist(lapply(1:4000, sim_pi, n = 10000))
  print(mean(ans))
})
```

    ## [1] 3.14124

    ##    user  system elapsed 
    ##   0.873   0.167   1.042

Rewrite the previous code using `parLapply()` to make it run faster.
Make sure you set the seed using `clusterSetRNGStream()`:

``` r
# YOUR CODE HERE
system.time({
  cl <- makePSOCKcluster(4L)   
  clusterSetRNGStream(cl, 1231)
  clusterExport(cl, c("sim_pi"),envir = environment())
  ans <- unlist(parLapply(cl,1:4000, sim_pi, n = 10000))
  print(mean(ans))
  stopCluster(cl)
})
```

    ## [1] 3.141578

    ##    user  system elapsed 
    ##   0.005   0.006   0.521

# SQL

Setup a temporary database by running the following chunk

``` r
# Initialize a temporary in memory database
con <- dbConnect(SQLite(), ":memory:")

# Download tables
film <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/film.csv")
film_category <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/film_category.csv")
category <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/category.csv")

# Copy data.frames to database
dbWriteTable(con, "film", film)
dbWriteTable(con, "film_category", film_category)
dbWriteTable(con, "category", category)
```

When you write a new chunk, remember to replace the r with sql,
connection=con. Some of these questions will reqruire you to use an
inner join. Read more about them here
<https://www.w3schools.com/sql/sql_join_inner.asp>

## Q1

How many movies is there avaliable in each rating catagory.

``` sql
PRAGMA table_info(film)
```

| cid | name                 | type    | notnull | dflt_value |  pk |
|:----|:---------------------|:--------|--------:|:-----------|----:|
| 0   | film_id              | INTEGER |       0 | NA         |   0 |
| 1   | title                | TEXT    |       0 | NA         |   0 |
| 2   | description          | TEXT    |       0 | NA         |   0 |
| 3   | release_year         | INTEGER |       0 | NA         |   0 |
| 4   | language_id          | INTEGER |       0 | NA         |   0 |
| 5   | original_language_id | INTEGER |       0 | NA         |   0 |
| 6   | rental_duration      | INTEGER |       0 | NA         |   0 |
| 7   | rental_rate          | REAL    |       0 | NA         |   0 |
| 8   | length               | INTEGER |       0 | NA         |   0 |
| 9   | replacement_cost     | REAL    |       0 | NA         |   0 |

Displaying records 1 - 10

The numbers of movies in each rating category were shown below.

``` sql
SELECT rating, COUNT(*) AS N
FROM film
GROUP BY rating
```

| rating |   N |
|:-------|----:|
| G      | 180 |
| NC-17  | 210 |
| PG     | 194 |
| PG-13  | 223 |
| R      | 195 |

5 records

## Q2

The average replacement cost and rental rate for each rating category
was shown below.

``` sql
SELECT rating,
       AVG(replacement_cost) AS avg_replacement_cost,
       AVG(rental_rate) AS avg_rental_rate
FROM film
GROUP BY rating
```

| rating | avg_replacement_cost | avg_rental_rate |
|:-------|---------------------:|----------------:|
| G      |             20.12333 |        2.912222 |
| NC-17  |             20.13762 |        2.970952 |
| PG     |             18.95907 |        3.051856 |
| PG-13  |             20.40256 |        3.034843 |
| R      |             20.23103 |        2.938718 |

5 records

## Q3

Use table film_category together with film to find the how many films
there are within each category ID

``` sql
PRAGMA table_info(film_category)
```

| cid | name        | type    | notnull | dflt_value |  pk |
|:----|:------------|:--------|--------:|:-----------|----:|
| 0   | film_id     | INTEGER |       0 | NA         |   0 |
| 1   | category_id | INTEGER |       0 | NA         |   0 |
| 2   | last_update | TEXT    |       0 | NA         |   0 |

3 records

``` sql
SELECT category_id,COUNT(*) AS N
FROM film AS f
  INNER JOIN film_category AS fc
  ON f.film_id=fc.film_id
GROUP BY category_id
```

| category_id |   N |
|:------------|----:|
| 1           |  64 |
| 2           |  66 |
| 3           |  60 |
| 4           |  57 |
| 5           |  58 |
| 6           |  68 |
| 7           |  62 |
| 8           |  69 |
| 9           |  73 |
| 10          |  61 |

Displaying records 1 - 10

The result was the same if we only use table film_category.

``` sql
SELECT category_id,COUNT(*) AS N
FROM film_category 
GROUP BY category_id
```

| category_id |   N |
|:------------|----:|
| 1           |  64 |
| 2           |  66 |
| 3           |  60 |
| 4           |  57 |
| 5           |  58 |
| 6           |  68 |
| 7           |  62 |
| 8           |  69 |
| 9           |  73 |
| 10          |  61 |

Displaying records 1 - 10

## Q4

Incorporate table category into the answer to the previous question to
find the name of the most popular category.

``` sql
SELECT fc.category_id,name,COUNT(*) AS N
FROM film_category AS fc
  INNER JOIN category AS c ON fc.category_id=c.category_id
  INNER JOIN film AS f ON f.film_id=fc.film_id
GROUP BY name
```

| category_id | name        |   N |
|:------------|:------------|----:|
| 1           | Action      |  64 |
| 2           | Animation   |  66 |
| 3           | Children    |  60 |
| 4           | Classics    |  57 |
| 5           | Comedy      |  58 |
| 6           | Documentary |  68 |
| 7           | Drama       |  62 |
| 8           | Family      |  69 |
| 9           | Foreign     |  73 |
| 10          | Games       |  61 |

Displaying records 1 - 10
