# Python Basics

This section covers the topics required in the following chapters. We suggest covering this section for someone who has yet to gain previous knowledge of Python programming.

## 1 Jupyter Notebook

In this book, we will work on Jupyter Notebook, which is the original web application for creating and sharing computational documents. It offers a simple, streamlined, document-centric experience.



See more about in <https://jupyter.org/>.

## Data types, numerical and text objects

Python is a programming language that lets us work quickly and integrate systems more effectively and contains many data types as part of the core language [@py].

The entities that we can create and manipulate in Python are called objects. We could make those objects by applying the assignment operator ('=').

For example, we create the object "a"; winch has assigned the value 4.


```python

x=4
x
#> 4
```

Each object in Python has tho characteristics, object type and object value.

Object type tells Python what kind of an object it's dealing with. A type could be a number, a string, a list, or something else. In this book, we will use those types of objects. Also, we will cover more complex data structures such as dictionaries, arrays and data frames.

The function type() shows us the object type. For example, the object x is an integer(int):


```python
type(x)
#> <class 'int'>
```

In this example, the object type is an integer(int), and the value is 4.

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

We are adding the " " to tell Python that Apple is a string.

## List and object attributes

A list is another useful object in Python, a vector of integers, strings, or both.


```python
liste=[1,2,3]
liste
#> [1, 2, 3]
```


```python
type(liste)
#> <class 'list'>
```

Objects whose value can change are called mutable objects, whereas objects whose value is unchangeable after they've been created are called immutable.


```python
4 # is inmutable

# but liste is mutable
#> 4
liste=["a","b"]
```

Most Python objects have either data or functions or both associated with them. These are known as attributes. The name of the attribute follows the name of the object. And these two are separated by a dot in between them. The two types of attributes are called either data attributes or methods.

The "list" data type has some more methods. Here are all of the methods of "list" objects:


```python
s=[1,2,3,4]
dir(s)[1:10]
#> ['__class__', '__class_getitem__', '__contains__', '__delattr__', '__delitem__', '__dir__', '__doc__', '__eq__', '__format__']
```

We printed only the first ten methods. But in the list, you can find more methods. In this section, we will cover some examples—the method append, which appends an object to the end of the list.


```python
s.append(6)
s
#> [1, 2, 3, 4, 6]
```

A data attribute contains information about the object. For example, to get the number of elements of a list, we use the data method "len." 


```python
len(s)
#> 5
```

There are other ways to manipulate a "list" without a method. For instance, to concatenate two different lists:


```python
t=[12,14,16]
s=s+t
s
#> [1, 2, 3, 4, 6, 12, 14, 16]
```

To select an element of a list, for example, selecting the second element of the list:


```python
s[1]
#> 2
```

As you can see, instead of typing the s[2], we write the number one because Python starts counting to zero.


```python
s[0]
#> 1
```

For example, replace the number 3 in lis "s" with the number 100 to replace an element of a list.


```python
s[2]=100
s
#> [1, 2, 100, 4, 6, 12, 14, 16]
```

To remove an element, for example, number 2:


```python
s.remove(2)
s
#> [1, 100, 4, 6, 12, 14, 16]
```

## Dictionaries

Dictionaries are useful objects for performing fast look-ups on underscored data.


```python
grades_dict={"Paulina": 100, "Coral":95} 
grades_dict
#> {'Paulina': 100, 'Coral': 95}
```

The "dictionaries" have two components, keys and values. The keys:


```python
grades_dict.keys()
#> dict_keys(['Paulina', 'Coral'])
```

And values:


```python
grades_dict.values()
#> dict_values([100, 95])
```

Dictionaries are mutable, then we can modify their content, by adding a new element:


```python
grades_dict["Alejandra"]=120
grades_dict
#> {'Paulina': 100, 'Coral': 95, 'Alejandra': 120}
```

Or modifying an element:


```python
# modify an element
grades_dict["Alejandra"]=grades_dict["Alejandra"]-30
grades_dict
#> {'Paulina': 100, 'Coral': 95, 'Alejandra': 90}
```

We could ask for an element in the dictionary:

```python
# members

"Karina" in grades_dict
#> False
```

We could combine two or more lists into a dictionary, having two keys and four values each:


```python
names=["Eugenio","Luis","Isa","Gisell"]
numbers=[9,17,80,79]
combine = {"Letthers":names,"Colors":numbers}
combine
#> {'Letthers': ['Eugenio', 'Luis', 'Isa', 'Gisell'], 'Colors': [9, 17, 80, 79]}
```

Or we may want to use one list for the keys and the other for the values:


```python
stud= dict(zip(names, numbers))
stud
#> {'Eugenio': 9, 'Luis': 17, 'Isa': 80, 'Gisell': 79}
```

## Python modules

Python also contains building functions that all Python programs can use. The user could make functions or could be developed by someone else in a library.

The Python modules are code libraries, and you can import Python modules using the import statements. Two of the most popular libraries in Python are Pandas and Numpy.

### Pandas data frames

It is a Python package that provides fast, flexible, and expressive data structures designed to make working with "relational" or "labeled" data easy and intuitive. It is the fundamental high-level building block for practical, real-world data analysis in Python. Additionally, it aims to become the most powerful and flexible open-source data analysis/manipulation tool available in any language. It is already well on its way toward this goal.

We will use it to create and manipulate data frames, which are two-dimensional, size-mutable, potentially heterogeneous tabular data.

To install the Pandas module, we write in a notebook cell or in the Conda terminal prompt "pip install pandas" in the Terminal prompt.

```         
pip install pandas
```

To import the library we use the "import" statement:


```python
import pandas as pd
```

After the name Pandas, we use the "as" statement to give pandas a short name. Then, when we use it, it is easier to call pd than pandas. For example, to create a data frame, we have to call the "pd". and the method DataFrame, referring to that we are calling the method DataFrame from the library pandas:

```{}
name=["Vale","Diana","Ivan","Vivi"]
df = pd.DataFrame(name,columns=["Column 1"])
df
```



```{=html}
<style type="text/css">
</style>
<table id="T_b5dd9">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_b5dd9_level0_col0" class="col_heading level0 col0" >Column 1</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_b5dd9_level0_row0" class="row_heading level0 row0" >0</th>
      <td id="T_b5dd9_row0_col0" class="data row0 col0" >Vale</td>
    </tr>
    <tr>
      <th id="T_b5dd9_level0_row1" class="row_heading level0 row1" >1</th>
      <td id="T_b5dd9_row1_col0" class="data row1 col0" >Diana</td>
    </tr>
    <tr>
      <th id="T_b5dd9_level0_row2" class="row_heading level0 row2" >2</th>
      <td id="T_b5dd9_row2_col0" class="data row2 col0" >Ivan</td>
    </tr>
    <tr>
      <th id="T_b5dd9_level0_row3" class="row_heading level0 row3" >3</th>
      <td id="T_b5dd9_row3_col0" class="data row3 col0" >Vivi</td>
    </tr>
  </tbody>
</table>

```

In the previous example, an argument of the function is columns; winch is the column name of the data frame. To know more about the function and its arguments, we could ask like this:

```         
help(pd.DataFrame) # or like this: pd.DataFrame?
```

In the previous example, by default, the index, the left column without a title, is numbered from zero to 3, but if we want to change the index:


```{}
name=["Vale","Diana","Ivan","Vivi"]
numbers=[51,11,511,50]
df = pd.DataFrame(name,columns=["Column 1"],index=numbers)
df
```



```
#>     Column 1
#> 51      Vale
#> 11     Diana
#> 511     Ivan
#> 50      Vivi
```

We could take advantage of the creation of a dictionary and transform it into a data frame:

```{}
my_dict=dict(zip(name, numbers))
sn=pd.DataFrame(my_dict,index=["Student_number"])
sn
```


```{=html}
<style type="text/css">
</style>
<table id="T_c3953">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_c3953_level0_col0" class="col_heading level0 col0" >Vale</th>
      <th id="T_c3953_level0_col1" class="col_heading level0 col1" >Diana</th>
      <th id="T_c3953_level0_col2" class="col_heading level0 col2" >Ivan</th>
      <th id="T_c3953_level0_col3" class="col_heading level0 col3" >Vivi</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_c3953_level0_row0" class="row_heading level0 row0" >Student_number</th>
      <td id="T_c3953_row0_col0" class="data row0 col0" >51</td>
      <td id="T_c3953_row0_col1" class="data row0 col1" >11</td>
      <td id="T_c3953_row0_col2" class="data row0 col2" >511</td>
      <td id="T_c3953_row0_col3" class="data row0 col3" >50</td>
    </tr>
  </tbody>
</table>

```
If we want to have the student_number as columns instead of a row, we could transpose the data frame:

```{}
sn.transpose()
```


```{=html}
<style type="text/css">
</style>
<table id="T_3d3b5">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_3d3b5_level0_col0" class="col_heading level0 col0" >Student_number</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_3d3b5_level0_row0" class="row_heading level0 row0" >Vale</th>
      <td id="T_3d3b5_row0_col0" class="data row0 col0" >51</td>
    </tr>
    <tr>
      <th id="T_3d3b5_level0_row1" class="row_heading level0 row1" >Diana</th>
      <td id="T_3d3b5_row1_col0" class="data row1 col0" >11</td>
    </tr>
    <tr>
      <th id="T_3d3b5_level0_row2" class="row_heading level0 row2" >Ivan</th>
      <td id="T_3d3b5_row2_col0" class="data row2 col0" >511</td>
    </tr>
    <tr>
      <th id="T_3d3b5_level0_row3" class="row_heading level0 row3" >Vivi</th>
      <td id="T_3d3b5_row3_col0" class="data row3 col0" >50</td>
    </tr>
  </tbody>
</table>

```


```{}
name=["Estefanía","Laura Yanet","María Guadalupe","Karla Lizette"]
nick=["Estef","Yanet","Lupita","Karla"]
number=[1,2,3,4]

# We use two list to create the dictionary
combine = {"Nick_name":nick,"Name":name}

# and from the dictionary we create the  data frame
df=pd.DataFrame(combine,index=number)
df
```


```
#>   Nick_name             name
#> 1     Estef        Estefanía
#> 2     Yanet      Laura Yanet
#> 3    Lupita  María Guadalupe
#> 4     Karla    Karla Lizette
```

To apply a method (function) to the data frame, we must type the Pandas object and a dot before the method. A useful method is "shape," which gives us the number of rows and columns.


```python
df.shape 
#> (4, 2)
```

To rename a data frame column:
```{}
df=df.rename(columns={"Nick_name": "Nick", "name": "Names"})
df#
```



```
#>      Nick            Names
#> 1   Estef        Estefanía
#> 2   Yanet      Laura Yanet
#> 3  Lupita  María Guadalupe
#> 4   Karla    Karla Lizette
```
It also applies for index:
```{}
df=df.rename(index={1: "x", 2: "y", 3: "z",4:"w"})
df
```


```
#>      Nick            Names
#> x   Estef        Estefanía
#> y   Yanet      Laura Yanet
#> z  Lupita  María Guadalupe
#> w   Karla    Karla Lizette
```

### Selecting rows and columns in a data frame

To select a column, we could type the column name:



```python
df["Nick"]
#> x     Estef
#> y     Yanet
#> z    Lupita
#> w     Karla
#> Name: Nick, dtype: object
```

The resulting object is a pandas series, a one-dimensional object, such as a list, but with an index, in this case, the data frame index.


```python
type(df["Nick"])
#> <class 'pandas.core.series.Series'>
```

If we want to keep the data frame type, we should add the square brackets twice:

```{}
df[["Nick"]]
```


```{=html}
<style type="text/css">
</style>
<table id="T_1f064">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_1f064_level0_col0" class="col_heading level0 col0" >Nick</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_1f064_level0_row0" class="row_heading level0 row0" >x</th>
      <td id="T_1f064_row0_col0" class="data row0 col0" >Estef</td>
    </tr>
    <tr>
      <th id="T_1f064_level0_row1" class="row_heading level0 row1" >y</th>
      <td id="T_1f064_row1_col0" class="data row1 col0" >Yanet</td>
    </tr>
    <tr>
      <th id="T_1f064_level0_row2" class="row_heading level0 row2" >z</th>
      <td id="T_1f064_row2_col0" class="data row2 col0" >Lupita</td>
    </tr>
    <tr>
      <th id="T_1f064_level0_row3" class="row_heading level0 row3" >w</th>
      <td id="T_1f064_row3_col0" class="data row3 col0" >Karla</td>
    </tr>
  </tbody>
</table>

```

Or two columns at once:

```{}
df[["Nick","Names"]]
```



```{=html}
<style type="text/css">
</style>
<table id="T_ff950">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_ff950_level0_col0" class="col_heading level0 col0" >Nick</th>
      <th id="T_ff950_level0_col1" class="col_heading level0 col1" >Names</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_ff950_level0_row0" class="row_heading level0 row0" >x</th>
      <td id="T_ff950_row0_col0" class="data row0 col0" >Estef</td>
      <td id="T_ff950_row0_col1" class="data row0 col1" >Estefanía</td>
    </tr>
    <tr>
      <th id="T_ff950_level0_row1" class="row_heading level0 row1" >y</th>
      <td id="T_ff950_row1_col0" class="data row1 col0" >Yanet</td>
      <td id="T_ff950_row1_col1" class="data row1 col1" >Laura Yanet</td>
    </tr>
    <tr>
      <th id="T_ff950_level0_row2" class="row_heading level0 row2" >z</th>
      <td id="T_ff950_row2_col0" class="data row2 col0" >Lupita</td>
      <td id="T_ff950_row2_col1" class="data row2 col1" >María Guadalupe</td>
    </tr>
    <tr>
      <th id="T_ff950_level0_row3" class="row_heading level0 row3" >w</th>
      <td id="T_ff950_row3_col0" class="data row3 col0" >Karla</td>
      <td id="T_ff950_row3_col1" class="data row3 col1" >Karla Lizette</td>
    </tr>
  </tbody>
</table>

```

To select rows, we use the method ".loc".

```{}
df.loc[["y"]]
```


```{=html}
<style type="text/css">
</style>
<table id="T_68ba0">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_68ba0_level0_col0" class="col_heading level0 col0" >Nick</th>
      <th id="T_68ba0_level0_col1" class="col_heading level0 col1" >Names</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_68ba0_level0_row0" class="row_heading level0 row0" >y</th>
      <td id="T_68ba0_row0_col0" class="data row0 col0" >Yanet</td>
      <td id="T_68ba0_row0_col1" class="data row0 col1" >Laura Yanet</td>
    </tr>
  </tbody>
</table>

```
```{}
df.loc[["y","z"]]
```


```{=html}
<style type="text/css">
</style>
<table id="T_1275f">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_1275f_level0_col0" class="col_heading level0 col0" >Nick</th>
      <th id="T_1275f_level0_col1" class="col_heading level0 col1" >Names</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_1275f_level0_row0" class="row_heading level0 row0" >y</th>
      <td id="T_1275f_row0_col0" class="data row0 col0" >Yanet</td>
      <td id="T_1275f_row0_col1" class="data row0 col1" >Laura Yanet</td>
    </tr>
    <tr>
      <th id="T_1275f_level0_row1" class="row_heading level0 row1" >z</th>
      <td id="T_1275f_row1_col0" class="data row1 col0" >Lupita</td>
      <td id="T_1275f_row1_col1" class="data row1 col1" >María Guadalupe</td>
    </tr>
  </tbody>
</table>

```


The "loc" method also works for selecting a column:
```{}
df.loc[:, ("Nick","Names")]
```


Or more than one column:

```{=html}
<style type="text/css">
</style>
<table id="T_8abea">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_8abea_level0_col0" class="col_heading level0 col0" >Nick</th>
      <th id="T_8abea_level0_col1" class="col_heading level0 col1" >Names</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_8abea_level0_row0" class="row_heading level0 row0" >x</th>
      <td id="T_8abea_row0_col0" class="data row0 col0" >Estef</td>
      <td id="T_8abea_row0_col1" class="data row0 col1" >Estefanía</td>
    </tr>
    <tr>
      <th id="T_8abea_level0_row1" class="row_heading level0 row1" >y</th>
      <td id="T_8abea_row1_col0" class="data row1 col0" >Yanet</td>
      <td id="T_8abea_row1_col1" class="data row1 col1" >Laura Yanet</td>
    </tr>
    <tr>
      <th id="T_8abea_level0_row2" class="row_heading level0 row2" >z</th>
      <td id="T_8abea_row2_col0" class="data row2 col0" >Lupita</td>
      <td id="T_8abea_row2_col1" class="data row2 col1" >María Guadalupe</td>
    </tr>
    <tr>
      <th id="T_8abea_level0_row3" class="row_heading level0 row3" >w</th>
      <td id="T_8abea_row3_col0" class="data row3 col0" >Karla</td>
      <td id="T_8abea_row3_col1" class="data row3 col1" >Karla Lizette</td>
    </tr>
  </tbody>
</table>

```

Sometimes is useful to select by position. We use the method .iloc[ rows, columns ] in this case. For example, to select the second and third columns:

```{}
df.iloc[: , 1:]
```


```{=html}
<style type="text/css">
</style>
<table id="T_1c847">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_1c847_level0_col0" class="col_heading level0 col0" >Names</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_1c847_level0_row0" class="row_heading level0 row0" >x</th>
      <td id="T_1c847_row0_col0" class="data row0 col0" >Estefanía</td>
    </tr>
    <tr>
      <th id="T_1c847_level0_row1" class="row_heading level0 row1" >y</th>
      <td id="T_1c847_row1_col0" class="data row1 col0" >Laura Yanet</td>
    </tr>
    <tr>
      <th id="T_1c847_level0_row2" class="row_heading level0 row2" >z</th>
      <td id="T_1c847_row2_col0" class="data row2 col0" >María Guadalupe</td>
    </tr>
    <tr>
      <th id="T_1c847_level0_row3" class="row_heading level0 row3" >w</th>
      <td id="T_1c847_row3_col0" class="data row3 col0" >Karla Lizette</td>
    </tr>
  </tbody>
</table>

```

The left side of the comma is for selecting rows, and the right is for columns. Another example:
```{}
df.iloc[: , 0:2]
```



```{=html}
<style type="text/css">
</style>
<table id="T_59ae4">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_59ae4_level0_col0" class="col_heading level0 col0" >Nick</th>
      <th id="T_59ae4_level0_col1" class="col_heading level0 col1" >Names</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_59ae4_level0_row0" class="row_heading level0 row0" >x</th>
      <td id="T_59ae4_row0_col0" class="data row0 col0" >Estef</td>
      <td id="T_59ae4_row0_col1" class="data row0 col1" >Estefanía</td>
    </tr>
    <tr>
      <th id="T_59ae4_level0_row1" class="row_heading level0 row1" >y</th>
      <td id="T_59ae4_row1_col0" class="data row1 col0" >Yanet</td>
      <td id="T_59ae4_row1_col1" class="data row1 col1" >Laura Yanet</td>
    </tr>
    <tr>
      <th id="T_59ae4_level0_row2" class="row_heading level0 row2" >z</th>
      <td id="T_59ae4_row2_col0" class="data row2 col0" >Lupita</td>
      <td id="T_59ae4_row2_col1" class="data row2 col1" >María Guadalupe</td>
    </tr>
    <tr>
      <th id="T_59ae4_level0_row3" class="row_heading level0 row3" >w</th>
      <td id="T_59ae4_row3_col0" class="data row3 col0" >Karla</td>
      <td id="T_59ae4_row3_col1" class="data row3 col1" >Karla Lizette</td>
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
<table id="T_d03eb">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_d03eb_level0_col0" class="col_heading level0 col0" >Nick</th>
      <th id="T_d03eb_level0_col1" class="col_heading level0 col1" >Names</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_d03eb_level0_row0" class="row_heading level0 row0" >z</th>
      <td id="T_d03eb_row0_col0" class="data row0 col0" >Lupita</td>
      <td id="T_d03eb_row0_col1" class="data row0 col1" >María Guadalupe</td>
    </tr>
    <tr>
      <th id="T_d03eb_level0_row1" class="row_heading level0 row1" >w</th>
      <td id="T_d03eb_row1_col0" class="data row1 col0" >Karla</td>
      <td id="T_d03eb_row1_col1" class="data row1 col1" >Karla Lizette</td>
    </tr>
  </tbody>
</table>

```

To insert a new column:
```{}
num_2=list(range(4))

df["Numbers_2"]=num_2
df
```


```{=html}
<style type="text/css">
</style>
<table id="T_3df28">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_3df28_level0_col0" class="col_heading level0 col0" >Nick</th>
      <th id="T_3df28_level0_col1" class="col_heading level0 col1" >Names</th>
      <th id="T_3df28_level0_col2" class="col_heading level0 col2" >Numbers_2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_3df28_level0_row0" class="row_heading level0 row0" >x</th>
      <td id="T_3df28_row0_col0" class="data row0 col0" >Estef</td>
      <td id="T_3df28_row0_col1" class="data row0 col1" >Estefanía</td>
      <td id="T_3df28_row0_col2" class="data row0 col2" >0</td>
    </tr>
    <tr>
      <th id="T_3df28_level0_row1" class="row_heading level0 row1" >y</th>
      <td id="T_3df28_row1_col0" class="data row1 col0" >Yanet</td>
      <td id="T_3df28_row1_col1" class="data row1 col1" >Laura Yanet</td>
      <td id="T_3df28_row1_col2" class="data row1 col2" >1</td>
    </tr>
    <tr>
      <th id="T_3df28_level0_row2" class="row_heading level0 row2" >z</th>
      <td id="T_3df28_row2_col0" class="data row2 col0" >Lupita</td>
      <td id="T_3df28_row2_col1" class="data row2 col1" >María Guadalupe</td>
      <td id="T_3df28_row2_col2" class="data row2 col2" >2</td>
    </tr>
    <tr>
      <th id="T_3df28_level0_row3" class="row_heading level0 row3" >w</th>
      <td id="T_3df28_row3_col0" class="data row3 col0" >Karla</td>
      <td id="T_3df28_row3_col1" class="data row3 col1" >Karla Lizette</td>
      <td id="T_3df28_row3_col2" class="data row3 col2" >3</td>
    </tr>
  </tbody>
</table>

```
The range method return an object that produces a sequence of integers from start (inclusive)  to stop (exclusive) by step.  range(i, j) produces i, i+1, i+2, ..., j-1.


To drooping colum(s):
```{}
df.drop(columns=['Names'])
```


```{=html}
<style type="text/css">
</style>
<table id="T_c0611">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_c0611_level0_col0" class="col_heading level0 col0" >Nick</th>
      <th id="T_c0611_level0_col1" class="col_heading level0 col1" >Numbers_2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_c0611_level0_row0" class="row_heading level0 row0" >x</th>
      <td id="T_c0611_row0_col0" class="data row0 col0" >Estef</td>
      <td id="T_c0611_row0_col1" class="data row0 col1" >0</td>
    </tr>
    <tr>
      <th id="T_c0611_level0_row1" class="row_heading level0 row1" >y</th>
      <td id="T_c0611_row1_col0" class="data row1 col0" >Yanet</td>
      <td id="T_c0611_row1_col1" class="data row1 col1" >1</td>
    </tr>
    <tr>
      <th id="T_c0611_level0_row2" class="row_heading level0 row2" >z</th>
      <td id="T_c0611_row2_col0" class="data row2 col0" >Lupita</td>
      <td id="T_c0611_row2_col1" class="data row2 col1" >2</td>
    </tr>
    <tr>
      <th id="T_c0611_level0_row3" class="row_heading level0 row3" >w</th>
      <td id="T_c0611_row3_col0" class="data row3 col0" >Karla</td>
      <td id="T_c0611_row3_col1" class="data row3 col1" >3</td>
    </tr>
  </tbody>
</table>

```

```{}
df.drop(columns=['Names',"Numbers_2"])
```


```{=html}
<style type="text/css">
</style>
<table id="T_27177">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_27177_level0_col0" class="col_heading level0 col0" >Nick</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_27177_level0_row0" class="row_heading level0 row0" >x</th>
      <td id="T_27177_row0_col0" class="data row0 col0" >Estef</td>
    </tr>
    <tr>
      <th id="T_27177_level0_row1" class="row_heading level0 row1" >y</th>
      <td id="T_27177_row1_col0" class="data row1 col0" >Yanet</td>
    </tr>
    <tr>
      <th id="T_27177_level0_row2" class="row_heading level0 row2" >z</th>
      <td id="T_27177_row2_col0" class="data row2 col0" >Lupita</td>
    </tr>
    <tr>
      <th id="T_27177_level0_row3" class="row_heading level0 row3" >w</th>
      <td id="T_27177_row3_col0" class="data row3 col0" >Karla</td>
    </tr>
  </tbody>
</table>

```


## Reading Excel and csv files

You can download the Excel file by copying and pasting and pasting to a browser through the following link:

https://github.com/abernal30/ML_python/blob/main/df.xlsx


I stored the file in a sub-directory named "data," and I called "df.xlsx"

To verify the names of the Sheets, we use the following code:

```python
import pandas as pd
sheets=pd.ExcelFile("data/df.xlsx").sheet_names
sheets
#> ['Sheet1', 'Sheet2', 'Sheet3']
```
I use the function read_excel of the Pandas library to read the Excel file. In this case, I use the argument sheet_name=sheets[0], equivalent to sheet_name="Sheet1".

```{}
data=pd.read_excel("data/df.xlsx",sheet_name=sheets[0])
data
```


```{=html}
<style type="text/css">
</style>
<table id="T_0fc8e">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_0fc8e_level0_col0" class="col_heading level0 col0" >Unnamed: 0</th>
      <th id="T_0fc8e_level0_col1" class="col_heading level0 col1" >Unnamed: 1</th>
      <th id="T_0fc8e_level0_col2" class="col_heading level0 col2" >Unnamed: 2</th>
      <th id="T_0fc8e_level0_col3" class="col_heading level0 col3" >Unnamed: 3</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_0fc8e_level0_row0" class="row_heading level0 row0" >0</th>
      <td id="T_0fc8e_row0_col0" class="data row0 col0" >nan</td>
      <td id="T_0fc8e_row0_col1" class="data row0 col1" >Título HOJA 1</td>
      <td id="T_0fc8e_row0_col2" class="data row0 col2" >nan</td>
      <td id="T_0fc8e_row0_col3" class="data row0 col3" >nan</td>
    </tr>
    <tr>
      <th id="T_0fc8e_level0_row1" class="row_heading level0 row1" >1</th>
      <td id="T_0fc8e_row1_col0" class="data row1 col0" >nan</td>
      <td id="T_0fc8e_row1_col1" class="data row1 col1" >nan</td>
      <td id="T_0fc8e_row1_col2" class="data row1 col2" >X</td>
      <td id="T_0fc8e_row1_col3" class="data row1 col3" >Y</td>
    </tr>
    <tr>
      <th id="T_0fc8e_level0_row2" class="row_heading level0 row2" >2</th>
      <td id="T_0fc8e_row2_col0" class="data row2 col0" >A</td>
      <td id="T_0fc8e_row2_col1" class="data row2 col1" >nan</td>
      <td id="T_0fc8e_row2_col2" class="data row2 col2" >2</td>
      <td id="T_0fc8e_row2_col3" class="data row2 col3" >10</td>
    </tr>
    <tr>
      <th id="T_0fc8e_level0_row3" class="row_heading level0 row3" >3</th>
      <td id="T_0fc8e_row3_col0" class="data row3 col0" >B</td>
      <td id="T_0fc8e_row3_col1" class="data row3 col1" >nan</td>
      <td id="T_0fc8e_row3_col2" class="data row3 col2" >50</td>
      <td id="T_0fc8e_row3_col3" class="data row3 col3" >nan</td>
    </tr>
    <tr>
      <th id="T_0fc8e_level0_row4" class="row_heading level0 row4" >4</th>
      <td id="T_0fc8e_row4_col0" class="data row4 col0" >C</td>
      <td id="T_0fc8e_row4_col1" class="data row4 col1" >nan</td>
      <td id="T_0fc8e_row4_col2" class="data row4 col2" >nan</td>
      <td id="T_0fc8e_row4_col3" class="data row4 col3" >25</td>
    </tr>
    <tr>
      <th id="T_0fc8e_level0_row5" class="row_heading level0 row5" >5</th>
      <td id="T_0fc8e_row5_col0" class="data row5 col0" >nan</td>
      <td id="T_0fc8e_row5_col1" class="data row5 col1" >nan</td>
      <td id="T_0fc8e_row5_col2" class="data row5 col2" >nan</td>
      <td id="T_0fc8e_row5_col3" class="data row5 col3" >nan</td>
    </tr>
    <tr>
      <th id="T_0fc8e_level0_row6" class="row_heading level0 row6" >6</th>
      <td id="T_0fc8e_row6_col0" class="data row6 col0" >D</td>
      <td id="T_0fc8e_row6_col1" class="data row6 col1" >nan</td>
      <td id="T_0fc8e_row6_col2" class="data row6 col2" >20</td>
      <td id="T_0fc8e_row6_col3" class="data row6 col3" >34</td>
    </tr>
    <tr>
      <th id="T_0fc8e_level0_row7" class="row_heading level0 row7" >7</th>
      <td id="T_0fc8e_row7_col0" class="data row7 col0" >E</td>
      <td id="T_0fc8e_row7_col1" class="data row7 col1" >nan</td>
      <td id="T_0fc8e_row7_col2" class="data row7 col2" >200</td>
      <td id="T_0fc8e_row7_col3" class="data row7 col3" >23</td>
    </tr>
  </tbody>
</table>

```

Sometimes is useful to read the Excel file and define as the index a column of the Excel file. In this case, we want the column "Unnamed: 0".

```{}
data=pd.read_excel("data/df.xlsx",sheet_name=sheets[0],index_col="Unnamed: 0")
data
```


```{=html}
<style type="text/css">
</style>
<table id="T_4fa17">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_4fa17_level0_col0" class="col_heading level0 col0" >Unnamed: 1</th>
      <th id="T_4fa17_level0_col1" class="col_heading level0 col1" >Unnamed: 2</th>
      <th id="T_4fa17_level0_col2" class="col_heading level0 col2" >Unnamed: 3</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_4fa17_level0_row0" class="row_heading level0 row0" >nan</th>
      <td id="T_4fa17_row0_col0" class="data row0 col0" >Título HOJA 1</td>
      <td id="T_4fa17_row0_col1" class="data row0 col1" >nan</td>
      <td id="T_4fa17_row0_col2" class="data row0 col2" >nan</td>
    </tr>
    <tr>
      <th id="T_4fa17_level0_row1" class="row_heading level0 row1" >nan</th>
      <td id="T_4fa17_row1_col0" class="data row1 col0" >nan</td>
      <td id="T_4fa17_row1_col1" class="data row1 col1" >X</td>
      <td id="T_4fa17_row1_col2" class="data row1 col2" >Y</td>
    </tr>
    <tr>
      <th id="T_4fa17_level0_row2" class="row_heading level0 row2" >A</th>
      <td id="T_4fa17_row2_col0" class="data row2 col0" >nan</td>
      <td id="T_4fa17_row2_col1" class="data row2 col1" >2</td>
      <td id="T_4fa17_row2_col2" class="data row2 col2" >10</td>
    </tr>
    <tr>
      <th id="T_4fa17_level0_row3" class="row_heading level0 row3" >B</th>
      <td id="T_4fa17_row3_col0" class="data row3 col0" >nan</td>
      <td id="T_4fa17_row3_col1" class="data row3 col1" >50</td>
      <td id="T_4fa17_row3_col2" class="data row3 col2" >nan</td>
    </tr>
    <tr>
      <th id="T_4fa17_level0_row4" class="row_heading level0 row4" >C</th>
      <td id="T_4fa17_row4_col0" class="data row4 col0" >nan</td>
      <td id="T_4fa17_row4_col1" class="data row4 col1" >nan</td>
      <td id="T_4fa17_row4_col2" class="data row4 col2" >25</td>
    </tr>
    <tr>
      <th id="T_4fa17_level0_row5" class="row_heading level0 row5" >nan</th>
      <td id="T_4fa17_row5_col0" class="data row5 col0" >nan</td>
      <td id="T_4fa17_row5_col1" class="data row5 col1" >nan</td>
      <td id="T_4fa17_row5_col2" class="data row5 col2" >nan</td>
    </tr>
    <tr>
      <th id="T_4fa17_level0_row6" class="row_heading level0 row6" >D</th>
      <td id="T_4fa17_row6_col0" class="data row6 col0" >nan</td>
      <td id="T_4fa17_row6_col1" class="data row6 col1" >20</td>
      <td id="T_4fa17_row6_col2" class="data row6 col2" >34</td>
    </tr>
    <tr>
      <th id="T_4fa17_level0_row7" class="row_heading level0 row7" >E</th>
      <td id="T_4fa17_row7_col0" class="data row7 col0" >nan</td>
      <td id="T_4fa17_row7_col1" class="data row7 col1" >200</td>
      <td id="T_4fa17_row7_col2" class="data row7 col2" >23</td>
    </tr>
  </tbody>
</table>

```


## Numpy modules

NumPy is the fundamental package for scientific computing in Python. It is a Python library that provides a multidimensional array object, various derived objects (such as masked arrays and matrices), and an assortment of routines for fast operations on arrays, including mathematical, logical, shape manipulation, sorting, selecting, I/O, discrete Fourier transforms, basic linear algebra, basic statistical operations, random simulation and much more.

In machine learning models is useful to work with Numpy arrays.
NumPy’s main object is the homogeneous multidimensional array.
For example, we could define variables x and y as an array. 


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

Sometimes is useful to simulate a missing value:

```python
np.nan
#> nan
```


## Missing value management

Suppose we read the Excel we used in the section "Reading Excel and CSV files." We want to work with the "Unnamed: 2" column, as a header we use the second row.
```{}
data=pd.read_excel("data/df.xlsx",sheet_name="Sheet1",index_col="Unnamed: 0",header=2)
data
```



```{=html}
<style type="text/css">
</style>
<table id="T_98b28">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_98b28_level0_col0" class="col_heading level0 col0" >Unnamed: 1</th>
      <th id="T_98b28_level0_col1" class="col_heading level0 col1" >X</th>
      <th id="T_98b28_level0_col2" class="col_heading level0 col2" >Y</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_98b28_level0_row0" class="row_heading level0 row0" >A</th>
      <td id="T_98b28_row0_col0" class="data row0 col0" >nan</td>
      <td id="T_98b28_row0_col1" class="data row0 col1" >2.000000</td>
      <td id="T_98b28_row0_col2" class="data row0 col2" >10.000000</td>
    </tr>
    <tr>
      <th id="T_98b28_level0_row1" class="row_heading level0 row1" >B</th>
      <td id="T_98b28_row1_col0" class="data row1 col0" >nan</td>
      <td id="T_98b28_row1_col1" class="data row1 col1" >50.000000</td>
      <td id="T_98b28_row1_col2" class="data row1 col2" >nan</td>
    </tr>
    <tr>
      <th id="T_98b28_level0_row2" class="row_heading level0 row2" >C</th>
      <td id="T_98b28_row2_col0" class="data row2 col0" >nan</td>
      <td id="T_98b28_row2_col1" class="data row2 col1" >nan</td>
      <td id="T_98b28_row2_col2" class="data row2 col2" >25.000000</td>
    </tr>
    <tr>
      <th id="T_98b28_level0_row3" class="row_heading level0 row3" >nan</th>
      <td id="T_98b28_row3_col0" class="data row3 col0" >nan</td>
      <td id="T_98b28_row3_col1" class="data row3 col1" >nan</td>
      <td id="T_98b28_row3_col2" class="data row3 col2" >nan</td>
    </tr>
    <tr>
      <th id="T_98b28_level0_row4" class="row_heading level0 row4" >D</th>
      <td id="T_98b28_row4_col0" class="data row4 col0" >nan</td>
      <td id="T_98b28_row4_col1" class="data row4 col1" >20.000000</td>
      <td id="T_98b28_row4_col2" class="data row4 col2" >34.000000</td>
    </tr>
    <tr>
      <th id="T_98b28_level0_row5" class="row_heading level0 row5" >E</th>
      <td id="T_98b28_row5_col0" class="data row5 col0" >nan</td>
      <td id="T_98b28_row5_col1" class="data row5 col1" >200.000000</td>
      <td id="T_98b28_row5_col2" class="data row5 col2" >23.000000</td>
    </tr>
  </tbody>
</table>

```

To get a better look  at our data frame, we select the second and third columns, with the following method:

```{}
data=data.iloc[:,1:]
data
```


```{=html}
<style type="text/css">
</style>
<table id="T_c1bd2">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_c1bd2_level0_col0" class="col_heading level0 col0" >X</th>
      <th id="T_c1bd2_level0_col1" class="col_heading level0 col1" >Y</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_c1bd2_level0_row0" class="row_heading level0 row0" >A</th>
      <td id="T_c1bd2_row0_col0" class="data row0 col0" >2.000000</td>
      <td id="T_c1bd2_row0_col1" class="data row0 col1" >10.000000</td>
    </tr>
    <tr>
      <th id="T_c1bd2_level0_row1" class="row_heading level0 row1" >B</th>
      <td id="T_c1bd2_row1_col0" class="data row1 col0" >50.000000</td>
      <td id="T_c1bd2_row1_col1" class="data row1 col1" >nan</td>
    </tr>
    <tr>
      <th id="T_c1bd2_level0_row2" class="row_heading level0 row2" >C</th>
      <td id="T_c1bd2_row2_col0" class="data row2 col0" >nan</td>
      <td id="T_c1bd2_row2_col1" class="data row2 col1" >25.000000</td>
    </tr>
    <tr>
      <th id="T_c1bd2_level0_row3" class="row_heading level0 row3" >nan</th>
      <td id="T_c1bd2_row3_col0" class="data row3 col0" >nan</td>
      <td id="T_c1bd2_row3_col1" class="data row3 col1" >nan</td>
    </tr>
    <tr>
      <th id="T_c1bd2_level0_row4" class="row_heading level0 row4" >D</th>
      <td id="T_c1bd2_row4_col0" class="data row4 col0" >20.000000</td>
      <td id="T_c1bd2_row4_col1" class="data row4 col1" >34.000000</td>
    </tr>
    <tr>
      <th id="T_c1bd2_level0_row5" class="row_heading level0 row5" >E</th>
      <td id="T_c1bd2_row5_col0" class="data row5 col0" >200.000000</td>
      <td id="T_c1bd2_row5_col1" class="data row5 col1" >23.000000</td>
    </tr>
  </tbody>
</table>

```

A first look to detect missing values is using the following function:

```python
data.isna().sum()
#> X    2
#> Y    2
#> dtype: int64
```

It tells us that both columns, "X" and "Y", have two missing values.

We have many alternatives to manage them. First eliminating the row that has at least one missing value.
```{}
data.dropna()
```


```{=html}
<style type="text/css">
</style>
<table id="T_12ba1">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_12ba1_level0_col0" class="col_heading level0 col0" >X</th>
      <th id="T_12ba1_level0_col1" class="col_heading level0 col1" >Y</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_12ba1_level0_row0" class="row_heading level0 row0" >A</th>
      <td id="T_12ba1_row0_col0" class="data row0 col0" >2.000000</td>
      <td id="T_12ba1_row0_col1" class="data row0 col1" >10.000000</td>
    </tr>
    <tr>
      <th id="T_12ba1_level0_row1" class="row_heading level0 row1" >D</th>
      <td id="T_12ba1_row1_col0" class="data row1 col0" >20.000000</td>
      <td id="T_12ba1_row1_col1" class="data row1 col1" >34.000000</td>
    </tr>
    <tr>
      <th id="T_12ba1_level0_row2" class="row_heading level0 row2" >E</th>
      <td id="T_12ba1_row2_col0" class="data row2 col0" >200.000000</td>
      <td id="T_12ba1_row2_col1" class="data row2 col1" >23.000000</td>
    </tr>
  </tbody>
</table>

```

Another alternative is filling them with a value, for example the last value available in the data frame. 
```{}
data.fillna(method='ffill')
```


```{=html}
<style type="text/css">
</style>
<table id="T_7d9a1">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_7d9a1_level0_col0" class="col_heading level0 col0" >X</th>
      <th id="T_7d9a1_level0_col1" class="col_heading level0 col1" >Y</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_7d9a1_level0_row0" class="row_heading level0 row0" >A</th>
      <td id="T_7d9a1_row0_col0" class="data row0 col0" >2.000000</td>
      <td id="T_7d9a1_row0_col1" class="data row0 col1" >10.000000</td>
    </tr>
    <tr>
      <th id="T_7d9a1_level0_row1" class="row_heading level0 row1" >B</th>
      <td id="T_7d9a1_row1_col0" class="data row1 col0" >50.000000</td>
      <td id="T_7d9a1_row1_col1" class="data row1 col1" >10.000000</td>
    </tr>
    <tr>
      <th id="T_7d9a1_level0_row2" class="row_heading level0 row2" >C</th>
      <td id="T_7d9a1_row2_col0" class="data row2 col0" >50.000000</td>
      <td id="T_7d9a1_row2_col1" class="data row2 col1" >25.000000</td>
    </tr>
    <tr>
      <th id="T_7d9a1_level0_row3" class="row_heading level0 row3" >nan</th>
      <td id="T_7d9a1_row3_col0" class="data row3 col0" >50.000000</td>
      <td id="T_7d9a1_row3_col1" class="data row3 col1" >25.000000</td>
    </tr>
    <tr>
      <th id="T_7d9a1_level0_row4" class="row_heading level0 row4" >D</th>
      <td id="T_7d9a1_row4_col0" class="data row4 col0" >20.000000</td>
      <td id="T_7d9a1_row4_col1" class="data row4 col1" >34.000000</td>
    </tr>
    <tr>
      <th id="T_7d9a1_level0_row5" class="row_heading level0 row5" >E</th>
      <td id="T_7d9a1_row5_col0" class="data row5 col0" >200.000000</td>
      <td id="T_7d9a1_row5_col1" class="data row5 col1" >23.000000</td>
    </tr>
  </tbody>
</table>

```

Or with a value such  as zero:
```{}
data_clenan=data.replace(np.nan,0)

# which is equivalent to 
#data.fillna(0)

data_clenan
```


```python
data_clean=data.replace(np.nan,0)

# which is equivalent to 
#data.fillna(0)

data_clean
#>          X     Y
#> A      2.0  10.0
#> B     50.0   0.0
#> C      0.0  25.0
#> NaN    0.0   0.0
#> D     20.0  34.0
#> E    200.0  23.0
```

We still have a missing value in the index, after letter "C". 
The next function
```{}
# This function skips the index elements of a data frame that are missing values, space or: ".",",",";",";","'",'""'.
# It returns a data frame without the ignored elements.
# Parameters:
# df: data frame  

#---- Do not change anything from here ----
def clean_na_index2(df):
    skips=[".",",",";",";","'",'""'," ",np.nan]
    con=[name_ind for name_ind in df.index if name_ind not in skips]
    return  df.loc[con, ]
#----- To here ------------

#Run the code so that Python can execute the function

df_index_clean=clean_na_index2(data_clean)
df_index_clean
```



```{=html}
<style type="text/css">
</style>
<table id="T_cc157">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_cc157_level0_col0" class="col_heading level0 col0" >X</th>
      <th id="T_cc157_level0_col1" class="col_heading level0 col1" >Y</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_cc157_level0_row0" class="row_heading level0 row0" >A</th>
      <td id="T_cc157_row0_col0" class="data row0 col0" >2.000000</td>
      <td id="T_cc157_row0_col1" class="data row0 col1" >10.000000</td>
    </tr>
    <tr>
      <th id="T_cc157_level0_row1" class="row_heading level0 row1" >B</th>
      <td id="T_cc157_row1_col0" class="data row1 col0" >50.000000</td>
      <td id="T_cc157_row1_col1" class="data row1 col1" >0.000000</td>
    </tr>
    <tr>
      <th id="T_cc157_level0_row2" class="row_heading level0 row2" >C</th>
      <td id="T_cc157_row2_col0" class="data row2 col0" >0.000000</td>
      <td id="T_cc157_row2_col1" class="data row2 col1" >25.000000</td>
    </tr>
    <tr>
      <th id="T_cc157_level0_row3" class="row_heading level0 row3" >D</th>
      <td id="T_cc157_row3_col0" class="data row3 col0" >20.000000</td>
      <td id="T_cc157_row3_col1" class="data row3 col1" >34.000000</td>
    </tr>
    <tr>
      <th id="T_cc157_level0_row4" class="row_heading level0 row4" >E</th>
      <td id="T_cc157_row4_col0" class="data row4 col0" >200.000000</td>
      <td id="T_cc157_row4_col1" class="data row4 col1" >23.000000</td>
    </tr>
  </tbody>
</table>

```

## Merge, joint or concatenate data frames

Suppose we want to merge the object df_index_clean of the previous section, with the data frame in the "Sheet2" of the Excel file "df.xlsx":
```{}
#,header=2
data=pd.read_excel("data/df.xlsx",sheet_name="Sheet2",index_col="Unnamed: 0")
data
```


```{=html}
<style type="text/css">
</style>
<table id="T_8e00b">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_8e00b_level0_col0" class="col_heading level0 col0" >W</th>
      <th id="T_8e00b_level0_col1" class="col_heading level0 col1" >Z</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_8e00b_level0_row0" class="row_heading level0 row0" >A</th>
      <td id="T_8e00b_row0_col0" class="data row0 col0" >2</td>
      <td id="T_8e00b_row0_col1" class="data row0 col1" >10</td>
    </tr>
    <tr>
      <th id="T_8e00b_level0_row1" class="row_heading level0 row1" >B</th>
      <td id="T_8e00b_row1_col0" class="data row1 col0" >50</td>
      <td id="T_8e00b_row1_col1" class="data row1 col1" >30</td>
    </tr>
    <tr>
      <th id="T_8e00b_level0_row2" class="row_heading level0 row2" >C</th>
      <td id="T_8e00b_row2_col0" class="data row2 col0" >30</td>
      <td id="T_8e00b_row2_col1" class="data row2 col1" >25</td>
    </tr>
    <tr>
      <th id="T_8e00b_level0_row3" class="row_heading level0 row3" >D</th>
      <td id="T_8e00b_row3_col0" class="data row3 col0" >20</td>
      <td id="T_8e00b_row3_col1" class="data row3 col1" >34</td>
    </tr>
    <tr>
      <th id="T_8e00b_level0_row4" class="row_heading level0 row4" >E</th>
      <td id="T_8e00b_row4_col0" class="data row4 col0" >200</td>
      <td id="T_8e00b_row4_col1" class="data row4 col1" >23</td>
    </tr>
  </tbody>
</table>

```
In this case, both data frames have the same index:


```python
print(data.index)
#> Index(['A', 'B', 'C', 'D', 'E'], dtype='object')
print(df_index_clean.index)
#> Index(['A', 'B', 'C', 'D', 'E'], dtype='object')
```
Then we can use the function concat. The argument axis=1 is to concatenate the columns of both data frames in the columns.
```{}
pd.concat([df_index_clean,data],axis=1)
```


```{=html}
<style type="text/css">
</style>
<table id="T_cb81c">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_cb81c_level0_col0" class="col_heading level0 col0" >X</th>
      <th id="T_cb81c_level0_col1" class="col_heading level0 col1" >Y</th>
      <th id="T_cb81c_level0_col2" class="col_heading level0 col2" >W</th>
      <th id="T_cb81c_level0_col3" class="col_heading level0 col3" >Z</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_cb81c_level0_row0" class="row_heading level0 row0" >A</th>
      <td id="T_cb81c_row0_col0" class="data row0 col0" >2.000000</td>
      <td id="T_cb81c_row0_col1" class="data row0 col1" >10.000000</td>
      <td id="T_cb81c_row0_col2" class="data row0 col2" >2</td>
      <td id="T_cb81c_row0_col3" class="data row0 col3" >10</td>
    </tr>
    <tr>
      <th id="T_cb81c_level0_row1" class="row_heading level0 row1" >B</th>
      <td id="T_cb81c_row1_col0" class="data row1 col0" >50.000000</td>
      <td id="T_cb81c_row1_col1" class="data row1 col1" >0.000000</td>
      <td id="T_cb81c_row1_col2" class="data row1 col2" >50</td>
      <td id="T_cb81c_row1_col3" class="data row1 col3" >30</td>
    </tr>
    <tr>
      <th id="T_cb81c_level0_row2" class="row_heading level0 row2" >C</th>
      <td id="T_cb81c_row2_col0" class="data row2 col0" >0.000000</td>
      <td id="T_cb81c_row2_col1" class="data row2 col1" >25.000000</td>
      <td id="T_cb81c_row2_col2" class="data row2 col2" >30</td>
      <td id="T_cb81c_row2_col3" class="data row2 col3" >25</td>
    </tr>
    <tr>
      <th id="T_cb81c_level0_row3" class="row_heading level0 row3" >D</th>
      <td id="T_cb81c_row3_col0" class="data row3 col0" >20.000000</td>
      <td id="T_cb81c_row3_col1" class="data row3 col1" >34.000000</td>
      <td id="T_cb81c_row3_col2" class="data row3 col2" >20</td>
      <td id="T_cb81c_row3_col3" class="data row3 col3" >34</td>
    </tr>
    <tr>
      <th id="T_cb81c_level0_row4" class="row_heading level0 row4" >E</th>
      <td id="T_cb81c_row4_col0" class="data row4 col0" >200.000000</td>
      <td id="T_cb81c_row4_col1" class="data row4 col1" >23.000000</td>
      <td id="T_cb81c_row4_col2" class="data row4 col2" >200</td>
      <td id="T_cb81c_row4_col3" class="data row4 col3" >23</td>
    </tr>
  </tbody>
</table>

```

Otherwise it would concatenate in the index (rows)

```{}
pd.concat([df_index_clean,data])

```



```{=html}
<style type="text/css">
</style>
<table id="T_22d2d">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_22d2d_level0_col0" class="col_heading level0 col0" >X</th>
      <th id="T_22d2d_level0_col1" class="col_heading level0 col1" >Y</th>
      <th id="T_22d2d_level0_col2" class="col_heading level0 col2" >W</th>
      <th id="T_22d2d_level0_col3" class="col_heading level0 col3" >Z</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_22d2d_level0_row0" class="row_heading level0 row0" >A</th>
      <td id="T_22d2d_row0_col0" class="data row0 col0" >2.000000</td>
      <td id="T_22d2d_row0_col1" class="data row0 col1" >10.000000</td>
      <td id="T_22d2d_row0_col2" class="data row0 col2" >nan</td>
      <td id="T_22d2d_row0_col3" class="data row0 col3" >nan</td>
    </tr>
    <tr>
      <th id="T_22d2d_level0_row1" class="row_heading level0 row1" >B</th>
      <td id="T_22d2d_row1_col0" class="data row1 col0" >50.000000</td>
      <td id="T_22d2d_row1_col1" class="data row1 col1" >0.000000</td>
      <td id="T_22d2d_row1_col2" class="data row1 col2" >nan</td>
      <td id="T_22d2d_row1_col3" class="data row1 col3" >nan</td>
    </tr>
    <tr>
      <th id="T_22d2d_level0_row2" class="row_heading level0 row2" >C</th>
      <td id="T_22d2d_row2_col0" class="data row2 col0" >0.000000</td>
      <td id="T_22d2d_row2_col1" class="data row2 col1" >25.000000</td>
      <td id="T_22d2d_row2_col2" class="data row2 col2" >nan</td>
      <td id="T_22d2d_row2_col3" class="data row2 col3" >nan</td>
    </tr>
    <tr>
      <th id="T_22d2d_level0_row3" class="row_heading level0 row3" >D</th>
      <td id="T_22d2d_row3_col0" class="data row3 col0" >20.000000</td>
      <td id="T_22d2d_row3_col1" class="data row3 col1" >34.000000</td>
      <td id="T_22d2d_row3_col2" class="data row3 col2" >nan</td>
      <td id="T_22d2d_row3_col3" class="data row3 col3" >nan</td>
    </tr>
    <tr>
      <th id="T_22d2d_level0_row4" class="row_heading level0 row4" >E</th>
      <td id="T_22d2d_row4_col0" class="data row4 col0" >200.000000</td>
      <td id="T_22d2d_row4_col1" class="data row4 col1" >23.000000</td>
      <td id="T_22d2d_row4_col2" class="data row4 col2" >nan</td>
      <td id="T_22d2d_row4_col3" class="data row4 col3" >nan</td>
    </tr>
    <tr>
      <th id="T_22d2d_level0_row5" class="row_heading level0 row5" >A</th>
      <td id="T_22d2d_row5_col0" class="data row5 col0" >nan</td>
      <td id="T_22d2d_row5_col1" class="data row5 col1" >nan</td>
      <td id="T_22d2d_row5_col2" class="data row5 col2" >2.000000</td>
      <td id="T_22d2d_row5_col3" class="data row5 col3" >10.000000</td>
    </tr>
    <tr>
      <th id="T_22d2d_level0_row6" class="row_heading level0 row6" >B</th>
      <td id="T_22d2d_row6_col0" class="data row6 col0" >nan</td>
      <td id="T_22d2d_row6_col1" class="data row6 col1" >nan</td>
      <td id="T_22d2d_row6_col2" class="data row6 col2" >50.000000</td>
      <td id="T_22d2d_row6_col3" class="data row6 col3" >30.000000</td>
    </tr>
    <tr>
      <th id="T_22d2d_level0_row7" class="row_heading level0 row7" >C</th>
      <td id="T_22d2d_row7_col0" class="data row7 col0" >nan</td>
      <td id="T_22d2d_row7_col1" class="data row7 col1" >nan</td>
      <td id="T_22d2d_row7_col2" class="data row7 col2" >30.000000</td>
      <td id="T_22d2d_row7_col3" class="data row7 col3" >25.000000</td>
    </tr>
    <tr>
      <th id="T_22d2d_level0_row8" class="row_heading level0 row8" >D</th>
      <td id="T_22d2d_row8_col0" class="data row8 col0" >nan</td>
      <td id="T_22d2d_row8_col1" class="data row8 col1" >nan</td>
      <td id="T_22d2d_row8_col2" class="data row8 col2" >20.000000</td>
      <td id="T_22d2d_row8_col3" class="data row8 col3" >34.000000</td>
    </tr>
    <tr>
      <th id="T_22d2d_level0_row9" class="row_heading level0 row9" >E</th>
      <td id="T_22d2d_row9_col0" class="data row9 col0" >nan</td>
      <td id="T_22d2d_row9_col1" class="data row9 col1" >nan</td>
      <td id="T_22d2d_row9_col2" class="data row9 col2" >200.000000</td>
      <td id="T_22d2d_row9_col3" class="data row9 col3" >23.000000</td>
    </tr>
  </tbody>
</table>

```

## APIS




## Plots or graphs





## Dates management

In this section we mange the data frame dates. 
```{}
data=pd.read_excel("data/df.xlsx",sheet_name="Sheet3")
#,index_col="Unnamed: 0"
data
```


```{=html}
<style type="text/css">
</style>
<table id="T_f2bdd">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_f2bdd_level0_col0" class="col_heading level0 col0" >date</th>
      <th id="T_f2bdd_level0_col1" class="col_heading level0 col1" >Sales</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_f2bdd_level0_row0" class="row_heading level0 row0" >0</th>
      <td id="T_f2bdd_row0_col0" class="data row0 col0" >Ene 2021</td>
      <td id="T_f2bdd_row0_col1" class="data row0 col1" >5</td>
    </tr>
    <tr>
      <th id="T_f2bdd_level0_row1" class="row_heading level0 row1" >1</th>
      <td id="T_f2bdd_row1_col0" class="data row1 col0" >Feb 2021</td>
      <td id="T_f2bdd_row1_col1" class="data row1 col1" >7</td>
    </tr>
    <tr>
      <th id="T_f2bdd_level0_row2" class="row_heading level0 row2" >2</th>
      <td id="T_f2bdd_row2_col0" class="data row2 col0" >Mar 2021</td>
      <td id="T_f2bdd_row2_col1" class="data row2 col1" >60</td>
    </tr>
    <tr>
      <th id="T_f2bdd_level0_row3" class="row_heading level0 row3" >3</th>
      <td id="T_f2bdd_row3_col0" class="data row3 col0" >Abr 2021</td>
      <td id="T_f2bdd_row3_col1" class="data row3 col1" >20</td>
    </tr>
    <tr>
      <th id="T_f2bdd_level0_row4" class="row_heading level0 row4" >4</th>
      <td id="T_f2bdd_row4_col0" class="data row4 col0" >May 2021</td>
      <td id="T_f2bdd_row4_col1" class="data row4 col1" >21</td>
    </tr>
  </tbody>
</table>

```

For some analysis in machine learning, we require to have the index data as date format. The previous data frame index type is a string:

```python
type(data.index[0])
#> <class 'int'>
```

Then, we use the following function:
```{}
# This function transforms the data frame index into a date format  
# (Timestamp). 
# It returns a data frame with the new date index
# Parameters:
# data: data frame with two columns, a date column and another one
# i_date: is the start date of the new index
# freq_i; frequency if the new index, "y" for year, "m" month, "d" day, "h" hour.
# col_name: name of the column in the data frame data that is not the date
# date_name= name of the column in the data that is the date
#---- Do not change anything from here ----
def index_date(data,i_date,freq_i,col_name,date_name):
  dat=data.set_index(date_name)
  ventas_s= pd.Series(
  list(dat[col_name]), index=pd.date_range(i_date, periods=len(dat),
  freq=freq_i), name=col_name)
  return pd.DataFrame(ventas_s)
#----- To here ------------

#Run the code so that Python can execute the function

i_date="1-1-2020"
freq_i="m"
col_name="Sales"
date_name="date"
data_ind_date=index_date(data,i_date,freq_i,col_name,date_name)
data_ind_date
```
Now the index is in a "Timestamp" format. For the moment, let's say that it is a date format. 


```{=html}
<style type="text/css">
</style>
<table id="T_6cb80">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_6cb80_level0_col0" class="col_heading level0 col0" >Sales</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_6cb80_level0_row0" class="row_heading level0 row0" >2020-01-31 00:00:00</th>
      <td id="T_6cb80_row0_col0" class="data row0 col0" >5</td>
    </tr>
    <tr>
      <th id="T_6cb80_level0_row1" class="row_heading level0 row1" >2020-02-29 00:00:00</th>
      <td id="T_6cb80_row1_col0" class="data row1 col0" >7</td>
    </tr>
    <tr>
      <th id="T_6cb80_level0_row2" class="row_heading level0 row2" >2020-03-31 00:00:00</th>
      <td id="T_6cb80_row2_col0" class="data row2 col0" >60</td>
    </tr>
    <tr>
      <th id="T_6cb80_level0_row3" class="row_heading level0 row3" >2020-04-30 00:00:00</th>
      <td id="T_6cb80_row3_col0" class="data row3 col0" >20</td>
    </tr>
    <tr>
      <th id="T_6cb80_level0_row4" class="row_heading level0 row4" >2020-05-31 00:00:00</th>
      <td id="T_6cb80_row4_col0" class="data row4 col0" >21</td>
    </tr>
  </tbody>
</table>

```

