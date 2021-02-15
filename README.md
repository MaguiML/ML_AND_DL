# Movie Recomendation system
*This project was taken from [DataFlair](https://data-flair.training/blogs/data-science-r-movie-recommendation/)* for my own learning. I have added some comments in the coding cells, added some explanations and definitions.

A recommendation system provides suggestions to the users based on their preferences. The main package we are going to use is *recommenderlab*. This [paper](https://cran.r-project.org/web/packages/recommenderlab/vignettes/recommenderlab.pdf) writen by Hashler  contains all about this package. We are goint to use it in this notebook.

First, we are going to install and load the proper libraries.

```
install.packages("recommenderlab")
install.packages("data.table")
install.packages("reshape2")
library(recommenderlab)
library(ggplot2)
library(data.table)
library(reshape2)
library(data.table)
```

## 0. Importing datasets

Use read.csv function to import the datasets. Use head() to look at the first entries of both:

```
movie_data <- read.csv("movies.csv", stringsAsFactors = FALSE)
rating_data <- read.csv("ratings.csv")
str(movie_data) 
str(rating_data)
head(movie_data, 4)
head(rating_data, 3)
```
## 1. Cleaning the dataset

We have to split the genre column into multiple columns. For the recomendation system , we want a column for each genre filled with 1 if the movie has that genre and 0 if it is not the case. 

First , we separate the genre column in a data frame

```
#Take the genre column as a data frame:
movie_genre <- as.data.frame(movie_data$genres, stringsAsFactors=FALSE) 
#This library will help us :

#The function tstrsplit will break the genre column in multiple each time it enc
#ters with teh separator |
movie_genre2 <- as.data.frame(tstrsplit(movie_genre[,1], '[|]', 
                                   type.convert=TRUE), 
                         stringsAsFactors=FALSE) #DataFlair
#Change the names of the new columns: 
colnames(movie_genre2) <- c(1:10)
#lets see how movie_genre2 is 
head(movie_genre2)
```
We are more closer to get the matrix. Now, we want the columns names to be the genres, and each row filled with 0 or 1. 

The code below construct a zero matrix of 10,330 rows ( movie_data has 10,229 rows, the additional is for the column names, the genres) and 18 columns (one for each genre). 

The first for loop will fill the matrix with ones whenever it be the case. Let's brake the steps in the for loop:


1.   The for loop goes from 1 to 10,229. It starts with index = 1.
2.   Inside it finds other foor loop that runs across the number of columns. We have index = 1 and col = 1.
3.   [*The which() function in R returns the position or the index of the value which satisfies the given condition.*](https://www.journaldev.com/45274/which-function-in-r#:~:text=The%20which()%20function%20in%20R%20returns%20the%20position%20or,and%20even%20vector%20as%20well.) In this case, gen_col contains the position where movie_genre2[1, 1] (= Adventure) == genre_mat[1,]. So, It will look for in the complete row, and check in which position is Adventure. In this case, gen_col = 2.
4.   Then, it fills with 1, the genre_mat1[1 + 1, 2] (the +1 is because the first row of genre_mat1 are the column names, we dont want to change it)
5.   Still inside the first for, index = 1, the steps 2 to 4 will be repeated until col = 18.

```
#Create the vector containing the names of 
list_genre <- c("Action", "Adventure", "Animation", "Children", 
                "Comedy", "Crime","Documentary", "Drama", "Fantasy",
                "Film-Noir", "Horror", "Musical", "Mystery","Romance",
                "Sci-Fi", "Thriller", "War", "Western")
#Create a matrix filled with zeros, with 10330 rows and 18 columns
genre_mat1 <- matrix(0,10330,18)
genre_mat1[1,] <- list_genre
colnames(genre_mat1) <- list_genre
#The for loops:
for (index in 1:nrow(movie_genre2)) {
  for (col in 1:ncol(movie_genre2)) {
    gen_col = which(genre_mat1[1,] == movie_genre2[index,col]) #Author DataFlair
    genre_mat1[index+1,gen_col] <- 1
}
}

genre_mat2 <- as.data.frame(genre_mat1[-1,], stringsAsFactors=FALSE) #remove first row, which was the genre list
#For loop convert to integers
for (col in 1:ncol(genre_mat2)) {
  genre_mat2[,col] <- as.integer(genre_mat2[,col]) #convert from characters to integers
} 
#See the first rows:
head(genre_mat2)
```
Recommenderlab uses the abstract ratingMatrix to provide a common interface for rating data.Now, lets create a new data frame, joining each movie (from movie_data) to its codified genres (genre_mat2)

```
SearchMatrix <- cbind(movie_data[,1:2], genre_mat2[])
head(SearchMatrix)    #DataFlair
```
We must convert the data frame *rating_data* to *realRatingMatrix*, a special form of *ratingMatrix*, only non-NA values are stored explicitly; NA values are represented
by a dot.

The dcast function puts userId in rows and movieId in columns. The NA entries in each row mean that the user did not rate the movie.

```
ratingMatrix <- dcast(rating_data, userId~movieId, value.var = "rating", na.rm=FALSE)
ratingMatrix <- as.matrix(ratingMatrix[,-1]) #remove userIds

#Convert rating matrix into a recommenderlab sparse matrix
ratingMatrix <- as(ratingMatrix, "realRatingMatrix")
ratingMatrix
```
## 2. Preparing data

*   **Select useful data:** 

We have to select useful data for the recommendation system. We can put a threshold for the minimum number of users who have rated a film and for the minimum numbers of views per film. The project set the threshold 50 for both numbers:

``` 
#Filter for movies by number of views and number of ratings
movie_ratings <- ratingMatrix[rowCounts(ratingMatrix) >50,colCounts(ratingMatrix) >50]
#Movie ratings
movie_ratings
```
*   **Data Normalization**

Normalizing data to remove rating bias. Check if mean  is close to zero in rows:

```
normalized_ratings <- normalize(movie_ratings)
sum(rowMeans(normalized_ratings))

```
*    **Data binarization**
We will use binarize() to transform into a 0-1 matrix. It divides data into two groups, then assign one of two values to all members of the same group. One has to define a threshold: the value 0 will be assigned to all data points below, and 1 to those above it.










