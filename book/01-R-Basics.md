# R Basics

## 1 R Markdown

In this book we will work on R Markdowns, which is a document format to embed code chunks (of R or other languages) in documents. But most important, it allows to print (knitr) in other authoring languages including LaTeX, HTML, Text, among others. 

For See more markdown content at: [@R-Markdown]. 


## 2 Coding basics

The entities that we can create and manipulate in R are called objects. These may be variables, arrays of numbers, character strings, functions, or more general structures. Those objects are created applying the assignment operator (‘<-’). It consists of the two characters ‘<’ (“less than”) and ‘-’ (“minus”) occurring strictly side-by-side and it ‘points’ to the object receiving the value of the expression [@introduction-R]. 

We also could  apply the operator ‘=’, however, in my experience, some functions use the "=" operator inside, and could be confused. 

For example, we create the object "a", wich has assigned the value 4. 

```r
a<- 4 
```


To delete an object from the environment. We can also use the function rm(). However, I suggest using R-studio. For an introduction to RStudio i suggest reviewing chapter 1 of the book [@rstudio]

```r
rm(a)
```

## Objet types: "atomic" structures

Types of objects frequently used in finance applications are numeric, character, vectors and  logical. These are known as "atomic" structures since their components are all of the same type. The rest of the objects, like matrix and Data frames, are built on these atomic objects.

Character strings are entered using either matching double (") or single (') quotes. For example:


```r
ticker<-"APPL" 
```

to print the object we use the function print or just typing the object name.


```r
ticker
#> [1] "APPL"
# or 
print(ticker)
#> [1] "APPL"
```


To review the object class we use function "class":

```r
class(ticker)
#> [1] "character"
```
Numeric objects. 

```r
num<-4
print(num)
#> [1] 4
print(num)
#> [1] 4
print(class(num))
#> [1] "numeric"
```


A vector consist of an ordered collection of numbers or characters. To build a vector we apply the function concatenate "c()". For example,

```r
v1<-c(160,165,167,145,145)
print(v1)
#> [1] 160 165 167 145 145
class(v1)
#> [1] "numeric"
```
As we can see, the object class is numeric, because the vector is taking the class of the atomic objects, in this case numeric. An example of a character vector: 


```r
v2<-c("Apple","Meta","Amazon")
print(v2)
#> [1] "Apple"  "Meta"   "Amazon"
class(v2)
#> [1] "character"
```

Selecting an element of a vector. To select an element we use brackets: "[]". For example, to select the first element of vector "v2":

```r
v2[1]
#> [1] "Apple"
```
Also we could select a sub sample:

```r
v2[1:2]
#> [1] "Apple" "Meta"
```
In the former example we just select a sub sample, but the object "v2" hasn't changed. If we want to change the object we need to create a new one. For example, if we want to delete an element, we use the minus sign "-".  For example the 2nd element of "v2":

```r
v2<-v2[-2]
v2
#> [1] "Apple"  "Amazon"
```
In this case the object "v2" has changed. Also, we could change an element of the vector, for example, changing the element "Amazon" by "Meta":


```r
v2[2]<-"Meta"
v2
#> [1] "Apple" "Meta"
```
If we would like to add a new element, for example "Amazon_new", we need to apply again the "c" function:

```r
v2<-c(v2,"Amazon_new")
v2
#> [1] "Apple"      "Meta"       "Amazon_new"
```

## Data frames

In Finance is common and practical using Data Frames, which are tabular-form data objects where each column can be of different form, that is, numeric or character. 

For this example we are going to use an already created data frame, from library Wooldridge. Get the data frame k401k from the library Wooldridge 

Remember that a library is a set of functions that someone created. The Wooldridge library has many data sets from the econometrics book of the author [@wooldridge]. 

To import the library, apply the function library()

```r
library(wooldridge) 
```
To import a databases from the library, the library must be imported, and just calling the data set name, in this case "k401k". 


```r
k4<-k401k
class(k4)
#> [1] "data.frame"
```

As we can see the object class is Data Frame. 


The function "colnames" shows the names of each column of the data frame. in this case, it is a character vector:


```r
colnames(k4)
#> [1] "prate"   "mrate"   "totpart" "totelg"  "age"     "totemp"  "sole"   
#> [8] "ltotemp"
```

Some times is convenient to change the column or rows names of a data frame, for example, change the name of the first column, to ""prate_1". In this case we use the function "colnames" and select, in brackets, the column number we want to change. Because we are changing the column names vector, we need to establish it with the assignment operator "<-".


```r
colnames(k4)[1]<-"prate_1" 
colnames(k4)
#> [1] "prate_1" "mrate"   "totpart" "totelg"  "age"     "totemp"  "sole"   
#> [8] "ltotemp"
```

To show or change a row name, we use the "rownames" function. For convenience, we select the first 5 rows.

```r
rownames(k4)[1:5]
#> [1] "1" "2" "3" "4" "5"
```
We could apply the same procedure we made in the "colnames" function to modify a row of a data frame. 


Selecting rows or columns

There are many ways to select a column or a row of a data frame. 


Selecting rows or columns by position, for example selecting the first row, column 5. A data frame has two dimensions, rows and columns, for selecting we also use brackets, separating the rows and columns by a 

```r
k4[1,5]
#> [1] 8
```

Selecting columns by $ symbol

```r
k4$age[1:10]
#>  [1]  8  6 10  7 28  7 31 13 21 10
```


Merging two data frames by its columns.

Suppose you have the following Data Frame:

```r
df1<-k4[1:6,c("prate_1","totpart","age")]
df2<-k4[1:6,c("age","totemp")]

df3<-cbind(df1,df2)
head(df3,10) 
#>   prate_1 totpart age age totemp
#> 1    26.1    1653   8   8   8709
#> 2   100.0     262   6   6    315
#> 3    97.6     166  10  10    275
#> 4   100.0     257   7   7    500
#> 5    82.5     591  28  28    933
#> 6   100.0      92   7   7    143
```


Print the the dimension of each data frame, applying the function paste, print and dim:


```r
dim<-dim(df3) 
dim
#> [1] 6 5
```

Applying the function cbind to merge the two data frames and call the object df3

```r
df1<-k4[1:6,c("prate_1","totpart","age")]
df2<-k4[1:6,c("age","totemp")]
df3<-cbind(df1,df2)
df3
#>   prate_1 totpart age age totemp
#> 1    26.1    1653   8   8   8709
#> 2   100.0     262   6   6    315
#> 3    97.6     166  10  10    275
#> 4   100.0     257   7   7    500
#> 5    82.5     591  28  28    933
#> 6   100.0      92   7   7    143
```

Note that the method is duplicating the column age. 

To takeoff one of the columns, select by number position adding minus symbol

```r
df3<-df3[,-3]
df3
#>   prate_1 totpart age totemp
#> 1    26.1    1653   8   8709
#> 2   100.0     262   6    315
#> 3    97.6     166  10    275
#> 4   100.0     257   7    500
#> 5    82.5     591  28    933
#> 6   100.0      92   7    143
```


Create a new variable, tot_part_age (totpart/age) and a variable that is the row names or index, call it index, of the data frame.  Insert both into the object df3.


```r
df3[" tot_part_age"]<-(df3[,"totpart"]/df3[,"age"])
df3["index"]<-rownames(df3)
```

Eliminate the 2nd row of object df3 and call it df4. 

```r
df4<-df3[-2,]
```


Apply again the function cbind to merge the df3 and df4

```r
#df5<-cbind(df3,df4)
```

It is goig to show a debug "Error in data.frame(..., check.names = FALSE) : 
arguments imply differing number of rows: 6, 5", which means that the number of rows is not the same. 

Careful: if the number of rows of a data frames is a multiple of another, by coincidence the cbind function would do the merge. However, R is going to fill the missing values repeating the values of a data frame. 

Try now with the function merge(x,y,by.x=,by.y=,all=T or F, all.x=T or F, all.y=T or F)

The merge function needs a pivot or a reference variable to make the merge. In this case the column index or identification id (both share the same variable). That id must be a unique value for each row and must be present in both data frames.  Also, we need to specify if we want to keep all the data in data frame x or y. 


```r
df5<-merge(df3,df4,by.x="age",by.y="age")
df5
#>   age prate_1.x totpart.x totemp.x  tot_part_age.x index.x prate_1.y totpart.y
#> 1   7     100.0       257      500        36.71429       4     100.0       257
#> 2   7     100.0       257      500        36.71429       4     100.0        92
#> 3   7     100.0        92      143        13.14286       6     100.0       257
#> 4   7     100.0        92      143        13.14286       6     100.0        92
#> 5   8      26.1      1653     8709       206.62500       1      26.1      1653
#> 6  10      97.6       166      275        16.60000       3      97.6       166
#> 7  28      82.5       591      933        21.10714       5      82.5       591
#>   totemp.y  tot_part_age.y index.y
#> 1      500        36.71429       4
#> 2      143        13.14286       6
#> 3      500        36.71429       4
#> 4      143        13.14286       6
#> 5     8709       206.62500       1
#> 6      275        16.60000       3
#> 7      933        21.10714       5
```


## xts objects

An xts class of object provide for uniform handling of R's different time-based data classes. Also, some API´s, such as quantmod, download the data in xts format. For example,  from the library xts I write into xlsx file the data set "sample_matrix".


```r
#data(sample_matrix)
#sm<-get("sample_matrix")
#data_df<-data.frame(sample_matrix)
#date<-rownames(data_df)
#data_df<-cbind(date,data_df)
#write.xlsx(data_df,"data/data_df.xlsx")
```

In the next section I covered how to read an "xlsx" file. 

```r
library(openxlsx)
data_df<-read.xlsx("data/data_df.xlsx")
```

By default the object class is a data frame. A feature of the "xts" objects is that the rownames are dates objects. Then first I replace the numerical rownames by the dates inside the object. 

```r
date<-data_df[,1]
rownames(data_df)<-date
# Also I eliminate the dates in row one. 
data_df_2<-data_df[,-1]
```



```r
library(xts)
data_xts<- as.xts(data_df_2)
```

There are some useful functions that we can use with xts objects, for example transforming into weekly, monthly, quarterly, yearly ,etc. 

```r
data_xt_m<-apply.monthly(data_xts,mean)
```

Making a sub sample:

```r
sub_set<-subset(data_xts,
  +index(data_xts)>="2007-05-01" &
  +index(data_xts)<="2007-06-30")
```


Note that in this two examples, we could use the apply.monthly to an object like data_df_2 and it will work, because it has as rownames the dates, but we can´t apply the subset function to that object, it would generate an empty object. 


## Reading and writing csv and xlsx

There are some libraries to write and  open an xlsx or csv files. I suggest using "openxlsx". 

```r
write.xlsx(df5,"data/df5.xlsx")
write.csv(df5,"data/df5.csv")
```

To open a file use, the File must be in the same directory or we need to specify the directory location; otherwise, it would be an error:

```r
library(openxlsx)
fdf5_x<-read.xlsx("data/df5.xlsx")
fdf5_c<-read.csv("data/df5.csv")
```



