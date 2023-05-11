# Python Basics

This section covers the topics required in the following chapters. We suggest covering this section for someone who has yet to gain previous knowledge of Python programming.

## 1 Jupyter Notebook

In this book, we will work on Jupyter Notebook, which is the original web application for creating and sharing computational documents. It offers a simple, streamlined, document-centric experience.



See more about in <https://jupyter.org/>.

## Data types, numerical and text objects

Python is a programming language that lets our work quickly and integrate systems more effectively contains many data types as part of the core language [@py].

The entities that we can create and manipulate in python are called objects. We could create those objects by applying the assignment operator ('=').

For example, we create the object "a"; winch has assigned the value 4.


```python

x=4
x
#> 4
```

Each object in Python has tho characteristics, object type and object value.

Object type tells Python what kind of an object it's dealing with. A type could be a number, or a string, or a list, or something else. In this book we will use those type of objects. Also we will cover more complex data structures such as dictionaries, arrays and data frames.

The function type() give us the object type. For example, the object x is an integer(int):


```python
type(x)
#> <class 'int'>
```

In this example, the object type is integer(int) and the object value is 4.

Besides integers, Python provides other numeric types, floating point numbers, and complex numbers (for example (5j). For example:


```python
type(1.23)
#> <class 'float'>
```

An example of string (str) would be:


```python
y="Apple"
type(y)
#> <class 'str'>
```

Look that we are adding the " " to tell Python that Apple is a string.

## List and object attributes

A list is another useful object in Python, which is a vector of integers, strings, or both.


```python
liste=[1,2,3]
liste
#> [1, 2, 3]
```


```python
type(liste)
#> <class 'list'>
```

Objects whose value can change called mutable objects, whereas objects whose value is unchangeable after they've been created are called immutable.


```python
4 # is inmutable

# but liste is mutable
#> 4
liste=["a","b"]
```

Most Python objects have either data or functions or both associated with them. These are known as attributes. The name of the attribute follows the name of the object. And these two are separated by a dot in between them. The two types of attributes are called either data attributes or methods.

The list data type has some more methods. Here are all of the methods of list objects:


```python
s=[1,2,3,4]
dir(s)[1:10]
#> ['__class__', '__class_getitem__', '__contains__', '__delattr__', '__delitem__', '__dir__', '__doc__', '__eq__', '__format__']
```

We printed only the first 10 methods. But in the list you could find more methods. In this section we will cover some examples. For instance the method append, which appends an object to the end of the list.


```python
s.append(6)
s
#> [1, 2, 3, 4, 6]
```

A data atribute contains information about the object. For example, to get the number of elements of a list, we use the data method len. 


```python
len(s)
#> 5
```

There are other ways to manipulate a list, without using a method. For instance, to concatenate two different lists:


```python
t=[12,14,16]
s=s+t
s
#> [1, 2, 3, 4, 6, 12, 14, 16]
```

To select an element of a list. for example selecting the second element of the list:


```python
s[1]
#> 2
```

As you can see, instead of typing the s[2], we type number one, because Python starts counting in zero.


```python
s[0]
#> 1
```

To replace an element of a list, for example replace number 3 in lis "s" by number 100


```python
s[2]=100
s
#> [1, 2, 100, 4, 6, 12, 14, 16]
```

To remove an element, for example number 2:


```python
s.remove(2)
s
#> [1, 100, 4, 6, 12, 14, 16]
```

## Dictionaries

Dictionaries are useful objects for performing fast look-ups on underscored data.


```python
grades_dict={"Arturo": 100, "Fátima":120} 
grades_dict
#> {'Arturo': 100, 'Fátima': 120}
```

Has two components, the keys:


```python
grades_dict.keys()
#> dict_keys(['Arturo', 'Fátima'])
```

And values:


```python
grades_dict.values()
#> dict_values([100, 120])
```

Dictionaries are mutable, then we can modify its contents, adding a new element:


```python
grades_dict["Karina"]=200
grades_dict
#> {'Arturo': 100, 'Fátima': 120, 'Karina': 200}
```

Or modifying an element:


```python
# modify an element
grades_dict["Arturo"]=grades_dict["Arturo"]-10
grades_dict
#> {'Arturo': 90, 'Fátima': 120, 'Karina': 200}
```

We could ask for an elemnt in the dctionary:


```python
# members

"Karina" in grades_dict
#> True
```

We could combine two or more lists into a dictionary, having two keys and four values each:


```python
letters=["A","B","C","D"]
colors=["Blue","Red","Black","Orange"]
combine = {"Letthers":letters,"Colors":colors}
combine
#> {'Letthers': ['A', 'B', 'C', 'D'], 'Colors': ['Blue', 'Red', 'Black', 'Orange']}
```

Or we may want to use one list for the keys and the other for the values:


```python
region_colors = dict(zip(letters, colors))
region_colors
#> {'A': 'Blue', 'B': 'Red', 'C': 'Black', 'D': 'Orange'}
```

## Python modules.

Python also contains building functions that can be used by all Python programs. Functions could be made by the user or could be developed by someone else in a library.

The Python modules are libraries of code and you can import Python modules using the import statements. Two of the mos popular libraries in Python are Pandas and numpy.

### Pandas data frames

Is a Python package providing fast, flexible, and expressive data structures designed to make working with "relational" or "labeled" data both easy and intuitive. It aims to be the fundamental high-level building block for doing practical, real-world data analysis in Python. Additionally, it has the broader goal of becoming the most powerful and flexible open source data analysis/manipulation tool available in any language. It is already well on its way toward this goal.

We will use it to create and manipulate data frames, which are two-dimensional, size-mutable, potentially heterogeneous tabular data.

To install the Pandas module, in the Terminal prompt or in a cell in the Jupyter note book type: pip install pandas.

```         
pip install pandas
```

To import the library we use the "import" statement:


```python
import pandas as pd
```

Look that after the name pandas we use the "as" statement, to give pandas a short name. Then, when we use it is easier to call pd than pandas. For example, to create a data frame we have to call the pd. and the method DataFrame, referring that we are calling the the function DataFrame from the library pandas:

```{}
colors=["Blue","Red","Black","Orange"]
df = pd.DataFrame(colors,columns=["Column 1"])
df
```


```{=html}
<style type="text/css">
</style>
<table id="T_6a241">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_6a241_level0_col0" class="col_heading level0 col0" >Column 1</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_6a241_level0_row0" class="row_heading level0 row0" >0</th>
      <td id="T_6a241_row0_col0" class="data row0 col0" >Blue</td>
    </tr>
    <tr>
      <th id="T_6a241_level0_row1" class="row_heading level0 row1" >1</th>
      <td id="T_6a241_row1_col0" class="data row1 col0" >Red</td>
    </tr>
    <tr>
      <th id="T_6a241_level0_row2" class="row_heading level0 row2" >2</th>
      <td id="T_6a241_row2_col0" class="data row2 col0" >Black</td>
    </tr>
    <tr>
      <th id="T_6a241_level0_row3" class="row_heading level0 row3" >3</th>
      <td id="T_6a241_row3_col0" class="data row3 col0" >Orange</td>
    </tr>
  </tbody>
</table>

```

In the previous example, an argument of the function is columns, winch is the column name of the data frame. To knowing more about the function and its arguments, we could ask like this:

```         
help(pd.DataFrame) # or like this: pd.DataFrame?
```

In the previous example, by default the index, the left column without a title, are numbers from zero to 3, but if we want to change the index:


```{}
letters=["A","B","C","D"]
df = pd.DataFrame(colors,columns=["Column 1"],index=letters)
df
```



```{=html}
<style type="text/css">
</style>
<table id="T_b9b49">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_b9b49_level0_col0" class="col_heading level0 col0" >Column 1</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_b9b49_level0_row0" class="row_heading level0 row0" >A</th>
      <td id="T_b9b49_row0_col0" class="data row0 col0" >Blue</td>
    </tr>
    <tr>
      <th id="T_b9b49_level0_row1" class="row_heading level0 row1" >B</th>
      <td id="T_b9b49_row1_col0" class="data row1 col0" >Red</td>
    </tr>
    <tr>
      <th id="T_b9b49_level0_row2" class="row_heading level0 row2" >C</th>
      <td id="T_b9b49_row2_col0" class="data row2 col0" >Black</td>
    </tr>
    <tr>
      <th id="T_b9b49_level0_row3" class="row_heading level0 row3" >D</th>
      <td id="T_b9b49_row3_col0" class="data row3 col0" >Orange</td>
    </tr>
  </tbody>
</table>

```


We could take advantage of the creation of a dictionary and transform it into a data frame:

```{}
my_dict=dict(zip(letters, colors))
pd.DataFrame(my_dict,index=[0])
```


```{=html}
<style type="text/css">
</style>
<table id="T_766b3">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_766b3_level0_col0" class="col_heading level0 col0" >A</th>
      <th id="T_766b3_level0_col1" class="col_heading level0 col1" >B</th>
      <th id="T_766b3_level0_col2" class="col_heading level0 col2" >C</th>
      <th id="T_766b3_level0_col3" class="col_heading level0 col3" >D</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_766b3_level0_row0" class="row_heading level0 row0" >0</th>
      <td id="T_766b3_row0_col0" class="data row0 col0" >Blue</td>
      <td id="T_766b3_row0_col1" class="data row0 col1" >Red</td>
      <td id="T_766b3_row0_col2" class="data row0 col2" >Black</td>
      <td id="T_766b3_row0_col3" class="data row0 col3" >Orange</td>
    </tr>
  </tbody>
</table>

```
```{}
letters=["A","B","C","D"]
colors=["Blue","Red","Black","Orange"]
numbers=[10,20,30,40]

# We use two list to create the dictionary
combine = {"Letters":letters,"Colors":colors,"Numbers":numbers}

# and from the dictionary we create the  data frame
df=pd.DataFrame(combine)
df
```


```{=html}
<style type="text/css">
</style>
<table id="T_6ded0">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_6ded0_level0_col0" class="col_heading level0 col0" >Letters</th>
      <th id="T_6ded0_level0_col1" class="col_heading level0 col1" >Colors</th>
      <th id="T_6ded0_level0_col2" class="col_heading level0 col2" >Numbers</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_6ded0_level0_row0" class="row_heading level0 row0" >0</th>
      <td id="T_6ded0_row0_col0" class="data row0 col0" >A</td>
      <td id="T_6ded0_row0_col1" class="data row0 col1" >Blue</td>
      <td id="T_6ded0_row0_col2" class="data row0 col2" >10</td>
    </tr>
    <tr>
      <th id="T_6ded0_level0_row1" class="row_heading level0 row1" >1</th>
      <td id="T_6ded0_row1_col0" class="data row1 col0" >B</td>
      <td id="T_6ded0_row1_col1" class="data row1 col1" >Red</td>
      <td id="T_6ded0_row1_col2" class="data row1 col2" >20</td>
    </tr>
    <tr>
      <th id="T_6ded0_level0_row2" class="row_heading level0 row2" >2</th>
      <td id="T_6ded0_row2_col0" class="data row2 col0" >C</td>
      <td id="T_6ded0_row2_col1" class="data row2 col1" >Black</td>
      <td id="T_6ded0_row2_col2" class="data row2 col2" >30</td>
    </tr>
    <tr>
      <th id="T_6ded0_level0_row3" class="row_heading level0 row3" >3</th>
      <td id="T_6ded0_row3_col0" class="data row3 col0" >D</td>
      <td id="T_6ded0_row3_col1" class="data row3 col1" >Orange</td>
      <td id="T_6ded0_row3_col2" class="data row3 col2" >40</td>
    </tr>
  </tbody>
</table>

```


To apply a method (function) to the data frame, we must type the pandas object and a dot, before the method. A useful a useful method is shape, which give us the number of rows and columns.


```python
df.shape 
#> (4, 3)
```

To rename a column:
```{}
df=df.rename(columns={"Letters": "let", "Colors": "col"})
df
```


```{=html}
<style type="text/css">
</style>
<table id="T_d5078">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_d5078_level0_col0" class="col_heading level0 col0" >let</th>
      <th id="T_d5078_level0_col1" class="col_heading level0 col1" >col</th>
      <th id="T_d5078_level0_col2" class="col_heading level0 col2" >Numbers</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_d5078_level0_row0" class="row_heading level0 row0" >0</th>
      <td id="T_d5078_row0_col0" class="data row0 col0" >A</td>
      <td id="T_d5078_row0_col1" class="data row0 col1" >Blue</td>
      <td id="T_d5078_row0_col2" class="data row0 col2" >10</td>
    </tr>
    <tr>
      <th id="T_d5078_level0_row1" class="row_heading level0 row1" >1</th>
      <td id="T_d5078_row1_col0" class="data row1 col0" >B</td>
      <td id="T_d5078_row1_col1" class="data row1 col1" >Red</td>
      <td id="T_d5078_row1_col2" class="data row1 col2" >20</td>
    </tr>
    <tr>
      <th id="T_d5078_level0_row2" class="row_heading level0 row2" >2</th>
      <td id="T_d5078_row2_col0" class="data row2 col0" >C</td>
      <td id="T_d5078_row2_col1" class="data row2 col1" >Black</td>
      <td id="T_d5078_row2_col2" class="data row2 col2" >30</td>
    </tr>
    <tr>
      <th id="T_d5078_level0_row3" class="row_heading level0 row3" >3</th>
      <td id="T_d5078_row3_col0" class="data row3 col0" >D</td>
      <td id="T_d5078_row3_col1" class="data row3 col1" >Orange</td>
      <td id="T_d5078_row3_col2" class="data row3 col2" >40</td>
    </tr>
  </tbody>
</table>

```
It also applies for index:
```{}
df=df.rename(index={0: "x", 1: "y", 2: "z"})
df
```


```{=html}
<style type="text/css">
</style>
<table id="T_04813">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_04813_level0_col0" class="col_heading level0 col0" >let</th>
      <th id="T_04813_level0_col1" class="col_heading level0 col1" >col</th>
      <th id="T_04813_level0_col2" class="col_heading level0 col2" >Numbers</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_04813_level0_row0" class="row_heading level0 row0" >x</th>
      <td id="T_04813_row0_col0" class="data row0 col0" >A</td>
      <td id="T_04813_row0_col1" class="data row0 col1" >Blue</td>
      <td id="T_04813_row0_col2" class="data row0 col2" >10</td>
    </tr>
    <tr>
      <th id="T_04813_level0_row1" class="row_heading level0 row1" >y</th>
      <td id="T_04813_row1_col0" class="data row1 col0" >B</td>
      <td id="T_04813_row1_col1" class="data row1 col1" >Red</td>
      <td id="T_04813_row1_col2" class="data row1 col2" >20</td>
    </tr>
    <tr>
      <th id="T_04813_level0_row2" class="row_heading level0 row2" >z</th>
      <td id="T_04813_row2_col0" class="data row2 col0" >C</td>
      <td id="T_04813_row2_col1" class="data row2 col1" >Black</td>
      <td id="T_04813_row2_col2" class="data row2 col2" >30</td>
    </tr>
    <tr>
      <th id="T_04813_level0_row3" class="row_heading level0 row3" >3</th>
      <td id="T_04813_row3_col0" class="data row3 col0" >D</td>
      <td id="T_04813_row3_col1" class="data row3 col1" >Orange</td>
      <td id="T_04813_row3_col2" class="data row3 col2" >40</td>
    </tr>
  </tbody>
</table>

```

### Selecting rows and columns in a data frame

To select a column we could type the column name:



```python
df["col"]
#> x      Blue
#> y       Red
#> z     Black
#> 3    Orange
#> Name: col, dtype: object
```

The resulting object is a pandas series, which is a one dimensional object, such as a list, but with and index, in this case the data frame index.


```python
type(df["col"])
#> <class 'pandas.core.series.Series'>
```

If we want to keep the data frame type, we should add the swuare brackets twice:

```{}
df[["col"]]
```


```{=html}
<style type="text/css">
</style>
<table id="T_fa6d0">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_fa6d0_level0_col0" class="col_heading level0 col0" >col</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_fa6d0_level0_row0" class="row_heading level0 row0" >x</th>
      <td id="T_fa6d0_row0_col0" class="data row0 col0" >Blue</td>
    </tr>
    <tr>
      <th id="T_fa6d0_level0_row1" class="row_heading level0 row1" >y</th>
      <td id="T_fa6d0_row1_col0" class="data row1 col0" >Red</td>
    </tr>
    <tr>
      <th id="T_fa6d0_level0_row2" class="row_heading level0 row2" >z</th>
      <td id="T_fa6d0_row2_col0" class="data row2 col0" >Black</td>
    </tr>
    <tr>
      <th id="T_fa6d0_level0_row3" class="row_heading level0 row3" >3</th>
      <td id="T_fa6d0_row3_col0" class="data row3 col0" >Orange</td>
    </tr>
  </tbody>
</table>

```

or two columns at once:


```{}
df[["col","let"]]
```



```{=html}
<style type="text/css">
</style>
<table id="T_b78d7">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_b78d7_level0_col0" class="col_heading level0 col0" >col</th>
      <th id="T_b78d7_level0_col1" class="col_heading level0 col1" >let</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_b78d7_level0_row0" class="row_heading level0 row0" >x</th>
      <td id="T_b78d7_row0_col0" class="data row0 col0" >Blue</td>
      <td id="T_b78d7_row0_col1" class="data row0 col1" >A</td>
    </tr>
    <tr>
      <th id="T_b78d7_level0_row1" class="row_heading level0 row1" >y</th>
      <td id="T_b78d7_row1_col0" class="data row1 col0" >Red</td>
      <td id="T_b78d7_row1_col1" class="data row1 col1" >B</td>
    </tr>
    <tr>
      <th id="T_b78d7_level0_row2" class="row_heading level0 row2" >z</th>
      <td id="T_b78d7_row2_col0" class="data row2 col0" >Black</td>
      <td id="T_b78d7_row2_col1" class="data row2 col1" >C</td>
    </tr>
    <tr>
      <th id="T_b78d7_level0_row3" class="row_heading level0 row3" >3</th>
      <td id="T_b78d7_row3_col0" class="data row3 col0" >Orange</td>
      <td id="T_b78d7_row3_col1" class="data row3 col1" >D</td>
    </tr>
  </tbody>
</table>

```

To select rows we use the method .loc

```{}
df.loc[["x"]]
```


```{=html}
<style type="text/css">
</style>
<table id="T_52dbe">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_52dbe_level0_col0" class="col_heading level0 col0" >let</th>
      <th id="T_52dbe_level0_col1" class="col_heading level0 col1" >col</th>
      <th id="T_52dbe_level0_col2" class="col_heading level0 col2" >Numbers</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_52dbe_level0_row0" class="row_heading level0 row0" >x</th>
      <td id="T_52dbe_row0_col0" class="data row0 col0" >A</td>
      <td id="T_52dbe_row0_col1" class="data row0 col1" >Blue</td>
      <td id="T_52dbe_row0_col2" class="data row0 col2" >10</td>
    </tr>
  </tbody>
</table>

```
```{}
df.loc[["x","z"]]# .styl
```


```{=html}
<style type="text/css">
</style>
<table id="T_c0a5c">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_c0a5c_level0_col0" class="col_heading level0 col0" >let</th>
      <th id="T_c0a5c_level0_col1" class="col_heading level0 col1" >col</th>
      <th id="T_c0a5c_level0_col2" class="col_heading level0 col2" >Numbers</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_c0a5c_level0_row0" class="row_heading level0 row0" >x</th>
      <td id="T_c0a5c_row0_col0" class="data row0 col0" >A</td>
      <td id="T_c0a5c_row0_col1" class="data row0 col1" >Blue</td>
      <td id="T_c0a5c_row0_col2" class="data row0 col2" >10</td>
    </tr>
    <tr>
      <th id="T_c0a5c_level0_row1" class="row_heading level0 row1" >z</th>
      <td id="T_c0a5c_row1_col0" class="data row1 col0" >C</td>
      <td id="T_c0a5c_row1_col1" class="data row1 col1" >Black</td>
      <td id="T_c0a5c_row1_col2" class="data row1 col2" >30</td>
    </tr>
  </tbody>
</table>

```

The loc method works also for selecting columns:
```{}
df.loc[:, ("col","Numbers")]
```


The loc method works also for selecting columns:

```{=html}
<style type="text/css">
</style>
<table id="T_d1bba">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_d1bba_level0_col0" class="col_heading level0 col0" >col</th>
      <th id="T_d1bba_level0_col1" class="col_heading level0 col1" >Numbers</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_d1bba_level0_row0" class="row_heading level0 row0" >x</th>
      <td id="T_d1bba_row0_col0" class="data row0 col0" >Blue</td>
      <td id="T_d1bba_row0_col1" class="data row0 col1" >10</td>
    </tr>
    <tr>
      <th id="T_d1bba_level0_row1" class="row_heading level0 row1" >y</th>
      <td id="T_d1bba_row1_col0" class="data row1 col0" >Red</td>
      <td id="T_d1bba_row1_col1" class="data row1 col1" >20</td>
    </tr>
    <tr>
      <th id="T_d1bba_level0_row2" class="row_heading level0 row2" >z</th>
      <td id="T_d1bba_row2_col0" class="data row2 col0" >Black</td>
      <td id="T_d1bba_row2_col1" class="data row2 col1" >30</td>
    </tr>
    <tr>
      <th id="T_d1bba_level0_row3" class="row_heading level0 row3" >3</th>
      <td id="T_d1bba_row3_col0" class="data row3 col0" >Orange</td>
      <td id="T_d1bba_row3_col1" class="data row3 col1" >40</td>
    </tr>
  </tbody>
</table>

```

Some times is useful selecting by position. in this case we use the method .iloc[ rows, columns ].  For example to select the second and third columns:
```{}
df.iloc[: , 1:]
```


```{=html}
<style type="text/css">
</style>
<table id="T_c5e77">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_c5e77_level0_col0" class="col_heading level0 col0" >col</th>
      <th id="T_c5e77_level0_col1" class="col_heading level0 col1" >Numbers</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_c5e77_level0_row0" class="row_heading level0 row0" >x</th>
      <td id="T_c5e77_row0_col0" class="data row0 col0" >Blue</td>
      <td id="T_c5e77_row0_col1" class="data row0 col1" >10</td>
    </tr>
    <tr>
      <th id="T_c5e77_level0_row1" class="row_heading level0 row1" >y</th>
      <td id="T_c5e77_row1_col0" class="data row1 col0" >Red</td>
      <td id="T_c5e77_row1_col1" class="data row1 col1" >20</td>
    </tr>
    <tr>
      <th id="T_c5e77_level0_row2" class="row_heading level0 row2" >z</th>
      <td id="T_c5e77_row2_col0" class="data row2 col0" >Black</td>
      <td id="T_c5e77_row2_col1" class="data row2 col1" >30</td>
    </tr>
    <tr>
      <th id="T_c5e77_level0_row3" class="row_heading level0 row3" >3</th>
      <td id="T_c5e77_row3_col0" class="data row3 col0" >Orange</td>
      <td id="T_c5e77_row3_col1" class="data row3 col1" >40</td>
    </tr>
  </tbody>
</table>

```

The left side of the comma is for selecting rows and the right for columns. Another example:
```{}
df.iloc[: , 0:2]
```


```{=html}
<style type="text/css">
</style>
<table id="T_e84e7">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_e84e7_level0_col0" class="col_heading level0 col0" >let</th>
      <th id="T_e84e7_level0_col1" class="col_heading level0 col1" >col</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_e84e7_level0_row0" class="row_heading level0 row0" >x</th>
      <td id="T_e84e7_row0_col0" class="data row0 col0" >A</td>
      <td id="T_e84e7_row0_col1" class="data row0 col1" >Blue</td>
    </tr>
    <tr>
      <th id="T_e84e7_level0_row1" class="row_heading level0 row1" >y</th>
      <td id="T_e84e7_row1_col0" class="data row1 col0" >B</td>
      <td id="T_e84e7_row1_col1" class="data row1 col1" >Red</td>
    </tr>
    <tr>
      <th id="T_e84e7_level0_row2" class="row_heading level0 row2" >z</th>
      <td id="T_e84e7_row2_col0" class="data row2 col0" >C</td>
      <td id="T_e84e7_row2_col1" class="data row2 col1" >Black</td>
    </tr>
    <tr>
      <th id="T_e84e7_level0_row3" class="row_heading level0 row3" >3</th>
      <td id="T_e84e7_row3_col0" class="data row3 col0" >D</td>
      <td id="T_e84e7_row3_col1" class="data row3 col1" >Orange</td>
    </tr>
  </tbody>
</table>

```
For rows:

```{}
df.iloc[2: , ]
```


```{=html}
<style type="text/css">
</style>
<table id="T_34c9f">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_34c9f_level0_col0" class="col_heading level0 col0" >let</th>
      <th id="T_34c9f_level0_col1" class="col_heading level0 col1" >col</th>
      <th id="T_34c9f_level0_col2" class="col_heading level0 col2" >Numbers</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_34c9f_level0_row0" class="row_heading level0 row0" >z</th>
      <td id="T_34c9f_row0_col0" class="data row0 col0" >C</td>
      <td id="T_34c9f_row0_col1" class="data row0 col1" >Black</td>
      <td id="T_34c9f_row0_col2" class="data row0 col2" >30</td>
    </tr>
    <tr>
      <th id="T_34c9f_level0_row1" class="row_heading level0 row1" >3</th>
      <td id="T_34c9f_row1_col0" class="data row1 col0" >D</td>
      <td id="T_34c9f_row1_col1" class="data row1 col1" >Orange</td>
      <td id="T_34c9f_row1_col2" class="data row1 col2" >40</td>
    </tr>
  </tbody>
</table>

```

To inset a new column
```{}
num_2=list(range(4))

df["Numbers_2"]=num_2
df
```


```{=html}
<style type="text/css">
</style>
<table id="T_ff7a7">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_ff7a7_level0_col0" class="col_heading level0 col0" >let</th>
      <th id="T_ff7a7_level0_col1" class="col_heading level0 col1" >col</th>
      <th id="T_ff7a7_level0_col2" class="col_heading level0 col2" >Numbers</th>
      <th id="T_ff7a7_level0_col3" class="col_heading level0 col3" >Numbers_2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_ff7a7_level0_row0" class="row_heading level0 row0" >x</th>
      <td id="T_ff7a7_row0_col0" class="data row0 col0" >A</td>
      <td id="T_ff7a7_row0_col1" class="data row0 col1" >Blue</td>
      <td id="T_ff7a7_row0_col2" class="data row0 col2" >10</td>
      <td id="T_ff7a7_row0_col3" class="data row0 col3" >0</td>
    </tr>
    <tr>
      <th id="T_ff7a7_level0_row1" class="row_heading level0 row1" >y</th>
      <td id="T_ff7a7_row1_col0" class="data row1 col0" >B</td>
      <td id="T_ff7a7_row1_col1" class="data row1 col1" >Red</td>
      <td id="T_ff7a7_row1_col2" class="data row1 col2" >20</td>
      <td id="T_ff7a7_row1_col3" class="data row1 col3" >1</td>
    </tr>
    <tr>
      <th id="T_ff7a7_level0_row2" class="row_heading level0 row2" >z</th>
      <td id="T_ff7a7_row2_col0" class="data row2 col0" >C</td>
      <td id="T_ff7a7_row2_col1" class="data row2 col1" >Black</td>
      <td id="T_ff7a7_row2_col2" class="data row2 col2" >30</td>
      <td id="T_ff7a7_row2_col3" class="data row2 col3" >2</td>
    </tr>
    <tr>
      <th id="T_ff7a7_level0_row3" class="row_heading level0 row3" >3</th>
      <td id="T_ff7a7_row3_col0" class="data row3 col0" >D</td>
      <td id="T_ff7a7_row3_col1" class="data row3 col1" >Orange</td>
      <td id="T_ff7a7_row3_col2" class="data row3 col2" >40</td>
      <td id="T_ff7a7_row3_col3" class="data row3 col3" >3</td>
    </tr>
  </tbody>
</table>

```
The range method return an object that produces a sequence of integers from start (inclusive)  to stop (exclusive) by step.  range(i, j) produces i, i+1, i+2, ..., j-1.


To drooping colum(s):
```{}
df.drop(columns=['col'])
```


```{=html}
<style type="text/css">
</style>
<table id="T_d79a0">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_d79a0_level0_col0" class="col_heading level0 col0" >let</th>
      <th id="T_d79a0_level0_col1" class="col_heading level0 col1" >Numbers</th>
      <th id="T_d79a0_level0_col2" class="col_heading level0 col2" >Numbers_2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_d79a0_level0_row0" class="row_heading level0 row0" >x</th>
      <td id="T_d79a0_row0_col0" class="data row0 col0" >A</td>
      <td id="T_d79a0_row0_col1" class="data row0 col1" >10</td>
      <td id="T_d79a0_row0_col2" class="data row0 col2" >0</td>
    </tr>
    <tr>
      <th id="T_d79a0_level0_row1" class="row_heading level0 row1" >y</th>
      <td id="T_d79a0_row1_col0" class="data row1 col0" >B</td>
      <td id="T_d79a0_row1_col1" class="data row1 col1" >20</td>
      <td id="T_d79a0_row1_col2" class="data row1 col2" >1</td>
    </tr>
    <tr>
      <th id="T_d79a0_level0_row2" class="row_heading level0 row2" >z</th>
      <td id="T_d79a0_row2_col0" class="data row2 col0" >C</td>
      <td id="T_d79a0_row2_col1" class="data row2 col1" >30</td>
      <td id="T_d79a0_row2_col2" class="data row2 col2" >2</td>
    </tr>
    <tr>
      <th id="T_d79a0_level0_row3" class="row_heading level0 row3" >3</th>
      <td id="T_d79a0_row3_col0" class="data row3 col0" >D</td>
      <td id="T_d79a0_row3_col1" class="data row3 col1" >40</td>
      <td id="T_d79a0_row3_col2" class="data row3 col2" >3</td>
    </tr>
  </tbody>
</table>

```
```{}
df.drop(columns=['col',"let"])
```


```{=html}
<style type="text/css">
</style>
<table id="T_2410c">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_2410c_level0_col0" class="col_heading level0 col0" >Numbers</th>
      <th id="T_2410c_level0_col1" class="col_heading level0 col1" >Numbers_2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_2410c_level0_row0" class="row_heading level0 row0" >x</th>
      <td id="T_2410c_row0_col0" class="data row0 col0" >10</td>
      <td id="T_2410c_row0_col1" class="data row0 col1" >0</td>
    </tr>
    <tr>
      <th id="T_2410c_level0_row1" class="row_heading level0 row1" >y</th>
      <td id="T_2410c_row1_col0" class="data row1 col0" >20</td>
      <td id="T_2410c_row1_col1" class="data row1 col1" >1</td>
    </tr>
    <tr>
      <th id="T_2410c_level0_row2" class="row_heading level0 row2" >z</th>
      <td id="T_2410c_row2_col0" class="data row2 col0" >30</td>
      <td id="T_2410c_row2_col1" class="data row2 col1" >2</td>
    </tr>
    <tr>
      <th id="T_2410c_level0_row3" class="row_heading level0 row3" >3</th>
      <td id="T_2410c_row3_col0" class="data row3 col0" >40</td>
      <td id="T_2410c_row3_col1" class="data row3 col1" >3</td>
    </tr>
  </tbody>
</table>

```

### Numpy

NumPy is the fundamental package for scientific computing in Python. It is a Python library that provides a multidimensional array object, various derived objects (such as masked arrays and matrices), and an assortment of routines for fast operations on arrays, including mathematical, logical, shape manipulation, sorting, selecting, I/O, discrete Fourier transforms, basic linear algebra, basic statistical operations, random simulation and much more.

In machine learnig models is useful to work with numpy arrays.
NumPy’s main object is the homogeneous multidimensional array.
For example, we could define variable x and y as an array. 

```python
import numpy as np

X=np.array([[1,2,3],[4,5,6]])
X
#> array([[1, 2, 3],
#>        [4, 5, 6]])
```
Or we can create a matrix for several variables. 

```python
Y=np.array([[2,4,6],[8,10,12]])
Y
#> array([[ 2,  4,  6],
#>        [ 8, 10, 12]])
```
Sometimes is useful to simulate an missing value:

```python
np.nan
#> nan
```


