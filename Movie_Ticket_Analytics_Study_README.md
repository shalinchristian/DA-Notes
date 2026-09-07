# Movie Ticket Booking Analytics — Study README

This README is a **memorisation sheet** for the Movie Ticket Booking Analytics assignment.

The main idea is:

> **Explore → Clean → Validate → Visualize → Export**

---

# 1. Libraries

Remember these three:

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

pd.set_option('display.max_columns', None)
```

### Easy memory trick

- `pd` → Pandas → tables/dataframes
- `np` → NumPy → numerical work / `NaN`
- `plt` → Matplotlib → graphs

---

# 2. Load the 3 Datasets

There are three DataFrames:

```python
df1 = pd.read_csv('Movie_Ticket_Assignment3_CSV1_Movies_Messy_500.csv')
df2 = pd.read_csv('Movie_Ticket_Assignment3_CSV2_Shows_Messy_500.csv')
df3 = pd.read_csv('Movie_Ticket_Assignment3_CSV3_Bookings_Messy_500.csv')
```

### Remember

```text
df1 → Movies
df2 → Shows
df3 → Bookings
```

### Relationship

```text
Movies ──Movie_ID──> Shows ──Show_ID──> Bookings
```

---

# 3. Q1 — Data Exploration

## A. Basic Exploration

For each dataset, check:

1. Shape
2. Info / data types
3. Missing values
4. Duplicate rows

```python
for name, df in [('Movies', df1), ('Shows', df2), ('Bookings', df3)]:

    print(f"\n===== {name} =====")

    print("Shape:", df.shape)

    df.info()

    print("\nMissing values:")
    print(df.isnull().sum())

    print("Duplicate rows:", df.duplicated().sum())
```

### Memorise the basic commands

```python
df.shape
df.info()
df.isnull().sum()
df.duplicated().sum()
```

---

## B. Categorical Values

Use:

```python
df['Column'].unique()
```

In this assignment:

```python
df1['Genre'].unique()
df1['Language'].unique()
df1['Certificate'].unique()

df2['Show_Type'].unique()

df3['Payment_Mode'].unique()
df3['Booking_Status'].unique()
df3['Booking_Channel'].unique()
```

### Memory

> **Categorical → `unique()`**

---

## C. Numerical Summary

Use:

```python
df.describe()
```

Example:

```python
df1.describe()
df2.describe()
df3.describe()
```

### Memory

> **Numerical summary → `describe()`**

---

## D. Referential Integrity

### Shows → Movies

```python
(~df2['Movie_ID'].isin(df1['Movie_ID'])).sum()
```

### Bookings → Shows

```python
(~df3['Show_ID'].isin(df2['Show_ID'])).sum()
```

### Memory

> `isin()` checks whether IDs exist in the parent table.

```text
Shows.Movie_ID   must exist in Movies.Movie_ID
Bookings.Show_ID must exist in Shows.Show_ID
```

---

# 4. Q2 — Data Preprocessing & Cleaning

The main cleaning pattern is:

```text
Duplicates → Strings → Invalid values → Missing values → Dates → Referential integrity
```

---

# 5. Movies Cleaning

## A. Remove duplicates

```python
df1 = df1.drop_duplicates()
```

---

## B. Clean string columns

### Genre

```python
df1['Genre'] = df1['Genre'].str.strip().str.title()
df1['Genre'] = df1['Genre'].fillna('Unknown')
```

### Language

```python
df1['Language'] = df1['Language'].str.strip().str.title()
df1['Language'] = df1['Language'].fillna('Unknown')
```

### Certificate

```python
df1['Certificate'] = df1['Certificate'].str.strip().str.upper()
```

### Movie Status

```python
df1['Movie_Status'] = df1['Movie_Status'].str.strip().str.title()
```

### Director

```python
df1['Director'] = df1['Director'].fillna('Unknown')
```

### Memorise the string pattern

```python
.str.strip()
.str.title()
.str.upper()
.fillna('Unknown')
```

---

## C. Invalid Duration

Valid duration is assumed to be:

```text
30 to 300 minutes
```

Code:

```python
df1.loc[
    (df1['Duration_Min'] < 30) | (df1['Duration_Min'] > 300),
    'Duration_Min'
] = np.nan
```

Then fill with median:

```python
df1['Duration_Min'] = df1['Duration_Min'].fillna(
    df1['Duration_Min'].median()
)
```

### Memory

> Invalid → `NaN` → median

---

## D. Invalid Rating

Valid rating:

```text
0 to 10
```

```python
df1.loc[
    (df1['Rating'] < 0) | (df1['Rating'] > 10),
    'Rating'
] = np.nan
```

Fill with mean:

```python
df1['Rating'] = df1['Rating'].fillna(
    df1['Rating'].mean()
)
```

### Memory

> Rating → invalid values become `NaN` → **mean**

---

## E. Invalid Production Budget

Negative budgets are invalid:

```python
df1.loc[
    df1['Production_Budget'] < 0,
    'Production_Budget'
] = np.nan
```

Fill with median:

```python
df1['Production_Budget'] = df1['Production_Budget'].fillna(
    df1['Production_Budget'].median()
)
```

### Memory

> Budget → negative = invalid → `NaN` → median

---

## F. Convert Release Date

```python
df1['Release_Date'] = pd.to_datetime(
    df1['Release_Date'],
    errors='coerce'
)
```

### Memory

> Date → `pd.to_datetime()`

---

# 6. Shows Cleaning

## A. Remove duplicates

```python
df2 = df2.drop_duplicates()
```

---

## B. Clean string columns

### Cinema Name

```python
df2['Cinema_Name'] = df2['Cinema_Name'].str.strip().str.title()
df2['Cinema_Name'] = df2['Cinema_Name'].fillna('Unknown')
```

### City

```python
df2['City'] = df2['City'].str.strip().str.title()
df2['City'] = df2['City'].fillna('Unknown')
```

### Show Type

```python
df2['Show_Type'] = df2['Show_Type'].str.strip().str.upper()
```

---

## C. Invalid Total Seats

Negative seats are invalid:

```python
df2.loc[
    df2['Total_Seats'] < 0,
    'Total_Seats'
] = np.nan
```

Fill with median:

```python
df2['Total_Seats'] = df2['Total_Seats'].fillna(
    df2['Total_Seats'].median()
)
```

---

## D. Invalid Ticket Price

Negative ticket price is invalid:

```python
df2.loc[
    df2['Ticket_Price'] < 0,
    'Ticket_Price'
] = np.nan
```

Fill with median:

```python
df2['Ticket_Price'] = df2['Ticket_Price'].fillna(
    df2['Ticket_Price'].median()
)
```

---

## E. Convert Show Date

```python
df2['Show_Date'] = pd.to_datetime(
    df2['Show_Date'],
    errors='coerce'
)
```

---

## F. Remove Orphan Movie IDs

```python
df2 = df2[df2['Movie_ID'].isin(df1['Movie_ID'])]
```

### Memory

> Keep only Shows whose `Movie_ID` exists in Movies.

---

# 7. Bookings Cleaning

## A. Remove duplicates

```python
df3 = df3.drop_duplicates()
```

---

## B. Clean strings

### Payment Mode

```python
df3['Payment_Mode'] = df3['Payment_Mode'].str.strip().str.title()
df3['Payment_Mode'] = df3['Payment_Mode'].fillna('Unknown')
```

### Booking Status

```python
df3['Booking_Status'] = df3['Booking_Status'].str.strip().str.title()
```

### Booking Channel

```python
df3['Booking_Channel'] = df3['Booking_Channel'].str.strip().str.title()
df3['Booking_Channel'] = df3['Booking_Channel'].fillna('Unknown')
```

### Customer Name

```python
df3['Customer_Name'] = df3['Customer_Name'].fillna('Unknown')
```

---

## C. Invalid Tickets

Tickets must be greater than 0.

```python
df3.loc[
    df3['Tickets'] <= 0,
    'Tickets'
] = np.nan
```

Fill with median:

```python
df3['Tickets'] = df3['Tickets'].fillna(
    df3['Tickets'].median()
)
```

---

## D. Invalid Customer Age

Assumed valid range:

```text
1 to 100
```

```python
df3.loc[
    (df3['Customer_Age'] < 1) | (df3['Customer_Age'] > 100),
    'Customer_Age'
] = np.nan
```

Fill with median:

```python
df3['Customer_Age'] = df3['Customer_Age'].fillna(
    df3['Customer_Age'].median()
)
```

---

## E. Missing Discount

Missing discount is treated as zero:

```python
df3['Discount'] = df3['Discount'].fillna(0)
```

### Memory

> Discount missing → **0**

---

## F. Convert Booking Date

```python
df3['Booking_Date'] = pd.to_datetime(
    df3['Booking_Date'],
    errors='coerce'
)
```

---

## G. Remove Orphan Show IDs

```python
df3 = df3[df3['Show_ID'].isin(df2['Show_ID'])]
```

### Memory

> Keep only Bookings whose `Show_ID` exists in Shows.

---

# 8. Q3 — Validation

After cleaning, check whether the data is actually clean.

## A. Missing values

```python
df1.isnull().sum()
df2.isnull().sum()
df3.isnull().sum()
```

---

## B. Duplicates

```python
df1.duplicated().sum()
df2.duplicated().sum()
df3.duplicated().sum()
```

---

## C. Categorical consistency

```python
df1['Genre'].unique()
df1['Language'].unique()

df2['Show_Type'].unique()

df3['Payment_Mode'].unique()
df3['Booking_Status'].unique()
df3['Booking_Channel'].unique()
```

---

## D. Numerical ranges

Use:

```python
df['column'].min()
df['column'].max()
```

Example:

```python
df1['Duration_Min'].min()
df1['Duration_Min'].max()

df1['Rating'].min()
df1['Rating'].max()
```

Other columns checked:

```text
Production_Budget
Total_Seats
Ticket_Price
Customer_Age
Tickets
```

---

## E. Date data types

```python
df1['Release_Date'].dtype
df2['Show_Date'].dtype
df3['Booking_Date'].dtype
```

---

## F. Referential integrity again

```python
(~df2['Movie_ID'].isin(df1['Movie_ID'])).sum()

(~df3['Show_ID'].isin(df2['Show_ID'])).sum()
```

### Memory

> **Validation = Missing + Duplicate + Category + Range + Date + IDs**

---

# 9. Q4 — Matplotlib

There are four validation graphs in Part A and two additional graphs.

---

## Graph 1 — Histogram

### Purpose

Shows the distribution of movie ratings.

```python
plt.hist(df1['Rating'], bins=20, edgecolor='black')
plt.title('Rating Distribution')
plt.xlabel('Rating')
plt.ylabel('Frequency')
plt.show()
```

### Memory

> **Histogram → distribution**

---

# 10. Graph 2 — Boxplot

### Purpose

Checks the spread/outliers of movie duration.

```python
plt.boxplot(df1['Duration_Min'])
plt.title('Duration_Min Spread')
plt.ylabel('Duration (Minutes)')
plt.show()
```

### Memory

> **Boxplot → spread + outliers**

---

# 11. Graph 3 — Bar Graph

### Purpose

Shows the number of movies per genre.

```python
genre_count = df1['Genre'].value_counts()

plt.bar(
    genre_count.index,
    genre_count.values
)

plt.title('Movies per Genre')
plt.xlabel('Genre')
plt.ylabel('Number of Movies')
plt.xticks(rotation=45)
plt.show()
```

### Memory

> `value_counts()` → count categories → bar graph

---

# 12. Graph 4 — Scatter Plot

### Purpose

Shows the relationship between tickets and final amount.

```python
plt.scatter(
    df3['Tickets'],
    df3['Final_Amount'],
    alpha=0.5
)

plt.title('Tickets vs Final Amount')
plt.xlabel('Tickets')
plt.ylabel('Final Amount')
plt.show()
```

### Memory

> **Scatter → relationship between two numerical variables**

---

# 13. Graph 5 — Line Graph

### Purpose

Shows total booking amount over time.

First group by date:

```python
daily = df3.groupby(
    df3['Booking_Date'].dt.date
)['Final_Amount'].sum()
```

Then plot:

```python
daily.plot(
    kind='line',
    color='purple'
)

plt.title('Total Booking Amount Over Time')
plt.xlabel('Date')
plt.ylabel('Total Amount')
plt.show()
```

### Memory

```text
Date → groupby → sum → line
```

---

# 14. Graph 6 — Pie Chart

### Purpose

Shows payment mode share.

```python
df3['Payment_Mode'].value_counts().plot(
    kind='pie',
    autopct='%1.0f%%'
)

plt.title('Payment Mode Share')
plt.ylabel('')
plt.show()
```

### Memory

> `value_counts()` → percentages → pie chart

---

# 15. Q5 — Export

Save the cleaned datasets:

```python
df1.to_csv('Movies_Cleaned.csv', index=False)
df2.to_csv('Shows_Cleaned.csv', index=False)
df3.to_csv('Bookings_Cleaned.csv', index=False)
```

### Output

```text
Movies_Cleaned.csv
Shows_Cleaned.csv
Bookings_Cleaned.csv
```

### Memory

> **DataFrame → `to_csv()`**

---

# 16. Most Important Pandas Commands

Memorise this table.

| Task | Command |
|---|---|
| Read CSV | `pd.read_csv()` |
| Rows/columns | `df.shape` |
| Data information | `df.info()` |
| Missing values | `df.isnull().sum()` |
| Duplicates | `df.duplicated().sum()` |
| Remove duplicates | `df.drop_duplicates()` |
| Unique categories | `df['col'].unique()` |
| Numerical summary | `df.describe()` |
| Count categories | `df['col'].value_counts()` |
| Remove whitespace | `.str.strip()` |
| Title case | `.str.title()` |
| Uppercase | `.str.upper()` |
| Fill missing values | `.fillna()` |
| Mean | `.mean()` |
| Median | `.median()` |
| Minimum | `.min()` |
| Maximum | `.max()` |
| Convert date | `pd.to_datetime()` |
| Membership check | `.isin()` |
| Save CSV | `.to_csv()` |

---

# 17. Most Important Matplotlib Commands

| Graph | Command |
|---|---|
| Histogram | `plt.hist()` |
| Boxplot | `plt.boxplot()` |
| Bar | `plt.bar()` |
| Scatter | `plt.scatter()` |
| Line | `plot(kind='line')` |
| Pie | `plot(kind='pie')` |
| Title | `plt.title()` |
| X label | `plt.xlabel()` |
| Y label | `plt.ylabel()` |
| Display | `plt.show()` |

---

# 18. The Cleaning Logic You Should Memorise

## Movies

```text
Duplicates
↓
Clean strings
↓
Fill missing text
↓
Fix invalid Duration
↓
Fix invalid Rating
↓
Fix invalid Budget
↓
Fill numerical missing values
↓
Convert Release_Date
```

### Exact memory

```text
Duration → 30–300 → median
Rating → 0–10 → mean
Budget → no negative → median
```

---

## Shows

```text
Duplicates
↓
Clean strings
↓
Fix invalid Seats
↓
Fix invalid Ticket Price
↓
Fill missing numbers
↓
Convert Show_Date
↓
Remove invalid Movie_ID
```

### Exact memory

```text
Seats → no negative → median
Ticket Price → no negative → median
Movie_ID → must exist in Movies
```

---

## Bookings

```text
Duplicates
↓
Clean strings
↓
Fix invalid Tickets
↓
Fix invalid Age
↓
Fill Discount
↓
Convert Booking_Date
↓
Remove invalid Show_ID
```

### Exact memory

```text
Tickets → > 0 → median
Age → 1–100 → median
Discount → missing = 0
Show_ID → must exist in Shows
```

---

# 19. One-Minute Revision

Before the exam, remember this:

```text
3 DATASETS
df1 = Movies
df2 = Shows
df3 = Bookings

RELATIONSHIP
Movies → Shows → Bookings

EXPLORATION
shape
info
isnull
duplicated
unique
describe
isin

CLEANING
drop_duplicates
strip
title
upper
fillna
loc
to_datetime
isin

MOVIES
Duration 30–300 → median
Rating 0–10 → mean
Budget >= 0 → median

SHOWS
Seats >= 0 → median
Price >= 0 → median
Movie_ID must exist

BOOKINGS
Tickets > 0 → median
Age 1–100 → median
Discount missing → 0
Show_ID must exist

VISUALIZATION
hist → distribution
boxplot → outliers/spread
bar → categories/count
scatter → relationship
line → trend over time
pie → proportions

EXPORT
to_csv()
```

---

# 20. Master Pattern

The entire assignment can be remembered as:

```text
IMPORT
   ↓
LOAD 3 CSVs
   ↓
EXPLORE
   ↓
CLEAN
   ↓
VALIDATE
   ↓
VISUALIZE
   ↓
EXPORT
```

> **Explore → Clean → Validate → Visualize → Export**
