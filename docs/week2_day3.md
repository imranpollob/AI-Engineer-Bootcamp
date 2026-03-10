# Day 3: Introduction to Pandas for Data Manipulation

Welcome to Day 3! NumPy is fantastic for pure math, but real-world data isn't just a giant block of numbers. You have customer names (Strings), timestamps (Dates), and purchases (Floats) all bundled together in spreadsheets and CSVs.

In the AI ecosystem, we organize tabular data using **Pandas**. 

## Pandas Data Structures
Pandas has two primary data structures:
1.  **Series:** A 1D column of data. You can think of it as a single column in an Excel sheet.
2.  **DataFrame:** A 2D table of data. This is simply a collection of Series side-by-side.

### Creating DataFrames
Let's manually build a tiny dataset and load it into a Pandas DataFrame using a standard Python dictionary:

```python
# day3_samples.py
import pandas as pd

# Define our data using a Dictionary
data = {"Name": ["Alice", "Bob"], "Age": [25, 30]}

# Convert it into a Pandas DataFrame
df = pd.DataFrame(data)

# Print the resulting table!
print(df)
```

```result
    Name  Age
0  Alice   25
1    Bob   30
```

Notice the `0` and `1` on the left? That is the **Index**. Pandas automatically assigns a row number to every entry, which makes looking up data lightning fast.

## Loading and Exploring Data
You rarely create DataFrames by hand. Usually, you import massive datasets from the web or your hard drive.

Pandas does this with a single line of code. We can pull data directly from a URL or a local `.csv` file.

```python
df = pd.read_csv("data.csv")
df.to_csv("data.csv", index=False)
df.to_excel("data.xlsx", index=False)
```

Once the data is loaded, how do we look at it? If you print a DataFrame with 10,000 rows, your terminal will freeze. Instead, we use exploratory methods:

*   `.head(n)`: Shows the first *n* rows.
*   `.tail(n)`: Shows the last *n* rows.
*   `.info()`: Gives you a technical summary of the dataset (columns, non-null counts, data types).
*   `.describe()`: Calculates instant statistical summaries (mean, min, max, quartiles) for all numerical columns!

## Hands-On Let's Code!

Let's look at `day3_ex1.py` to see Pandas in action. We are going to load the famous "Iris" dataset directly from GitHub.

### Exercise 1: Exploring and Filtering the Iris Dataset
The Iris dataset contains descriptions of different types of flowers. Let's load it and slice it up!

```python
# day3_ex1.py
import pandas as pd

# Load Dataset from the internet!
df = pd.read_csv("https://raw.githubusercontent.com/mwaskom/seaborn-data/master/iris.csv")

# Instantly grab mathematical summaries of the flower sizes
print(df.describe())

# We can select specific columns by passing a list of column names
selected_columns = df[["species", "sepal_length"]]
print("Selected Columns: \n", selected_columns.head())
```

```result
count    150.000000   150.000000    150.000000   150.000000
mean       5.843333     3.057333      3.758000     1.199333
std        0.828066     0.435866      1.765298     0.762238
min        4.300000     2.000000      1.000000     0.100000
25%        5.100000     2.800000      1.600000     0.300000
50%        5.800000     3.000000      4.350000     1.300000
75%        6.400000     3.300000      5.100000     1.800000
max        7.900000     4.400000      6.900000     2.500000
Selected Columns:
   species  sepal_length
0  setosa           5.1
1  setosa           4.9
2  setosa           4.7
3  setosa           4.6
4  setosa           5.0
```

### Filtering Data with Conditionals
Just like we did with NumPy boolean masking yesterday, Pandas allows us to instantly filter thousands of rows. 

```python
import pandas as pd
df = pd.read_csv("https://raw.githubusercontent.com/mwaskom/seaborn-data/master/iris.csv")

# Let's find every 'setosa' flower with a sepal_length greater than 5.0
filtered_rows = df[(df["sepal_length"] > 5.0) & (df["species"] == "setosa")]

print("Filtered Rows: \n", filtered_rows.head())
```

```result
Filtered Rows: 
     sepal_length  sepal_width  petal_length  petal_width species
0            5.1          3.5           1.4          0.2  setosa
5            5.4          3.9           1.7          0.4  setosa
10           5.4          3.7           1.5          0.2  setosa
14           5.8          4.0           1.2          0.2  setosa
15           5.7          4.4           1.5          0.4  setosa
```



Finally, if you need to access specific rows by their numerical index, you use `.iloc` (Index Location):
```python
import pandas as pd
df = pd.read_csv("https://raw.githubusercontent.com/mwaskom/seaborn-data/master/iris.csv")

# Prints the very first row
print(df.iloc[0]) 

# Prints all rows (:), but only the first column
print(df.iloc[:, 0]) 
```

```result
sepal_length       5.1
sepal_width        3.5
petal_length       1.4
petal_width        0.2
species         setosa
Name: 0, dtype: object

0      5.1
1      4.9
2      4.7
3      4.6
4      5.0
      ...
145    6.7
146    6.3
147    6.5
148    6.2
149    5.9
Name: sepal_length, Length: 150, dtype: float64
```

## Wrapping Up Day 3
You now know how to pull data from a `.csv`, look at its summary statistics, select columns, and filter rows based on multiple conditions!

But what happens when that data is wrong? What if the spreadsheet is missing "Ages" or has formatting errors?

Tomorrow, on **Day 4: Data Cleaning**, we roll up our sleeves and learn how to handle missing values, transform columns, and merge multiple broken datasets back together!
