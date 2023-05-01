# Understanding Lego Sets Popularity

## Background

We recently applied to work as a data analyst at the Lego Group. As part of the job interview process, we received the following take-home assignment:

We are asked to use the provided dataset to understand the popularity of different Lego sets and themes. The idea is to become familiarized with the data to be ready for an interview with a business stakeholder.

### Research questions

Create a report to summarize our findings for the following questions. The following questions need to be answered:

1. What is the average number of Lego sets released per year?
2. What is the average number of Lego parts per year? Create a visualization
3. What are the 5 most popular colors used in Lego parts?
4. What proportion of Lego parts are transparent?
5. What are the 5 rarest lego bricks?
6. Summarize findings.

## Database Schema

Visualization of how the tables are related to each other. ([source](https://rebrickable.com/downloads)):

![erd](data/lego_erd.png)

## Database Description

#### inventory_parts
- "inventory_id" - id of the inventory the part is in (as in the inventories table)
- "part_num" - unique id for the part (as in the parts table)
- "color_id" - id of the color
- "quantity" - the number of copies of the part included in the set
- "is_spare" - whether or not it is a spare part

#### parts
- "part_num" - unique id for the part (as in the inventory_parts table)
- "name" - name of the part
- "part_cat_id" - part category id (as in part_catagories table)

#### part_categories
- "id" - part category id (as in parts table)
- "name" - name of the category the part belongs to

#### colors
- "id" - id of the color (as in inventory_parts table)
- "name" - color name
- "rgb" - rgb code of the color
- "is_trans" - whether or not the part is transparent/translucent

#### inventories
- "id" - id of the inventory the part is in (as in the inventory_sets and inventory_parts tables)
- "version" - version number
- "set_num" - set number (as in sets table)

#### inventory_sets
- "inventory_id" - id of the inventory the part is in (as in the inventories table)
- "set_num" - set number (as in sets table)
- "quantity" - the quantity of sets included

#### sets
- "set_num" - unique set id (as in inventory_sets and inventories tables)
- "name" - the name of the set
- "year" - the year the set was published
- "theme_id" - the id of the theme the set belongs to (as in themes table)
- num-parts - the number of parts in the set

#### themes
- "id" - the id of the theme (as in the sets table)
- "name" - the name of the theme
- "parent_id" - the id of the larger theme, if there is one


***Acknowledgments**: Rebrickable.com*

## Methods 

The database has been querying with PostgreSQL using Common Table Expression, aggregate functions and filtering. 

The visualizations have been created using Python (plotly). 

# Analysis

## 1. What is the average number of Lego sets released per year?


**Answer:** The average number of Lego sets per year is 176 sets. 

**Solution:**
To calculate the average number of Lego sets released per year, we'll follow these steps :
- determine the total amount of sets (aliased here as total_sets),
- determine the total amount of years (aliased here as total_years),
- calcute the average by dividing the total amount of sets by the total amount of years.


```python
SELECT 
    s.total_sets/y.total_years AS avg_sets_per_year,
    s.total_sets,
    y.total_years
FROM
    -- calculate total sets
    (SELECT COUNT(set_num) AS total_sets
     FROM sets) AS s, 
    -- calculate total years
    (SELECT COUNT(DISTINCT year) AS total_years
     FROM sets) AS y
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>avg_sets_per_year</th>
      <th>total_sets</th>
      <th>total_years</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>176</td>
      <td>11673</td>
      <td>66</td>
    </tr>
  </tbody>
</table>
</div>



## 2. What is the average number of Lego parts per year?

### 2.1. Total average ### 
In first solution as average we take the total of the parts released over all of the years. With this logic, we have the result of 28698 parts/year as average. 


```python
-- find average number of lego parts for all the year

-- CTE for all parts
WITH data_per_year AS (
    SELECT year, 
    SUM(num_parts) AS parts_per_year, 
    COUNT(num_parts) AS sets_per_year,
    SUM(num_parts) / COUNT(num_parts) AS avg_parts_per_set_per_year
FROM sets
GROUP BY year
ORDER BY year)

-- calculate the average parts per all of the years
SELECT ROUND(AVG(parts_per_year), 2) AS avg_parts
FROM data_per_year
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>avg_parts</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>28698.32</td>
    </tr>
  </tbody>
</table>
</div>



### 2.2. Average per each year ### 
The average number of pieces across all sets has been determined to be 162. Additionally, by analyzing the data further, we are able to calculate the average number of pieces produced by the company each year, as demonstrated in the table below. 


```python
-- find average of lego parts by each year

SELECT ROUND(AVG(num_parts), 0) AS avg_parts_general
FROM sets
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>avg_parts_general</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>162</td>
    </tr>
  </tbody>
</table>
</div>




```python
-- find average of lego parts by each year

SELECT year, 
	SUM(num_parts) AS sum_parts, 
	COUNT(num_parts) AS count_parts,
	ROUND(AVG(num_parts), 0) AS avg_parts_per_year
FROM sets
GROUP BY year
ORDER BY year 
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>year</th>
      <th>sum_parts</th>
      <th>count_parts</th>
      <th>avg_parts_per_year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1950</td>
      <td>71</td>
      <td>7</td>
      <td>10</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1953</td>
      <td>66</td>
      <td>4</td>
      <td>17</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1954</td>
      <td>173</td>
      <td>14</td>
      <td>12</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1955</td>
      <td>1032</td>
      <td>28</td>
      <td>37</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1956</td>
      <td>222</td>
      <td>12</td>
      <td>19</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>61</th>
      <td>2013</td>
      <td>107537</td>
      <td>593</td>
      <td>181</td>
    </tr>
    <tr>
      <th>62</th>
      <td>2014</td>
      <td>121007</td>
      <td>713</td>
      <td>170</td>
    </tr>
    <tr>
      <th>63</th>
      <td>2015</td>
      <td>134110</td>
      <td>665</td>
      <td>202</td>
    </tr>
    <tr>
      <th>64</th>
      <td>2016</td>
      <td>150834</td>
      <td>596</td>
      <td>253</td>
    </tr>
    <tr>
      <th>65</th>
      <td>2017</td>
      <td>77203</td>
      <td>296</td>
      <td>261</td>
    </tr>
  </tbody>
</table>
<p>66 rows × 4 columns</p>
</div>



### 2.3. Create a visualization

The bar chart below built based on the data from item 2.2 (the table is copied below). ach bar represents average part released per each year. Green line on the chart represents average number across all sets. 


```python
-- find average lego parts per each year

SELECT year, 
	ROUND(AVG(num_parts), 0) AS avg_parts
FROM sets
GROUP BY year
ORDER BY year 
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>year</th>
      <th>avg_parts</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1950</td>
      <td>10</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1953</td>
      <td>17</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1954</td>
      <td>12</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1955</td>
      <td>37</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1956</td>
      <td>19</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>61</th>
      <td>2013</td>
      <td>181</td>
    </tr>
    <tr>
      <th>62</th>
      <td>2014</td>
      <td>170</td>
    </tr>
    <tr>
      <th>63</th>
      <td>2015</td>
      <td>202</td>
    </tr>
    <tr>
      <th>64</th>
      <td>2016</td>
      <td>253</td>
    </tr>
    <tr>
      <th>65</th>
      <td>2017</td>
      <td>261</td>
    </tr>
  </tbody>
</table>
<p>66 rows × 2 columns</p>
</div>




```python
# import plotly module
import plotly.express as px

# create bar chart trace
fig = px.bar(df2,
             x="year",
             y="avg_parts",
             title="<b>Average number of Lego parts per year</b>",
             labels={"year": "Year of Production", "avg_parts": "Average Number of Parts"})

# add horizontal line at y=162
fig.add_shape(type='line',
              x0=df2['year'].min(), y0=162,
              x1=df2['year'].max(), y1=162,
              line=dict(color='yellow', width=2),
              xref='x', yref='y')

# add title to the line at y=162
fig.add_annotation(x=df2['year'].min() + 2, y=172,
                   text="Average",
                   showarrow=False,
                   font=dict(color='gray', size=10),
                   xref='x',
                   yref='y')

# show the plot
fig.show()
```



## 4. What are the 5 most popular colors used in Lego parts?

**Answer:** 5 the most popular colors used in Lego parts:
1) Black (115085 parts)
2) White (66536 parts)
3) Light Bluish Gray (55302 parts)
4) Red (50213 parts)
5) Dark Bluish Gray (43907 parts)

**Solution:** To calculate the 5 most popular colors in Lego parts, we joined 'colors' and 'inventory_parts' tables, and counted the amount of each color. 


```python
-- find 5 most popular colors in lego parts

SELECT c.name AS color_name,
	COUNT(ip.part_num) AS count
FROM colors AS c
JOIN inventory_parts AS ip ON c.id=ip.color_id
GROUP BY color_name
ORDER BY count DESC
LIMIT 5
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>color_name</th>
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Black</td>
      <td>115085</td>
    </tr>
    <tr>
      <th>1</th>
      <td>White</td>
      <td>66536</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Light Bluish Gray</td>
      <td>55302</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Red</td>
      <td>50213</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Dark Bluish Gray</td>
      <td>43907</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Create bar chart
fig = px.bar(df3,
             x="color_name",
             y="count",
             title="<b>5 most popular colors used in Lego parts</b>",
             labels={"color_name": "Color", "count": "Number of parts"})

# Show the plot
fig.show()
```



## 5. What proportion of Lego parts are transparent?

**Answer:** The proportion of transparent lego parts to all parts is around 6.3%

**Solution:** We need to find number of all parts and number of transparent parts, which are 580069 and 36318 retrospectively, that leads to result around 6.3% transparent lego parts. 


```python
-- find proportion of transparent parts 

-- CTE for all parts
WITH all_parts AS (
	SELECT COUNT(DISTINCT part_num)::float AS all_parts
	FROM inventory_parts),

-- CTE for transparent parts
transparent_parts AS (
	SELECT COUNT(DISTINCT ip.part_num) AS transparent_parts
	FROM inventory_parts AS ip
	JOIN colors AS c ON ip.color_id = c.id
	WHERE is_trans = True)

-- calculate the proportion
SELECT (transparent_parts/all_parts) AS proportion_transparent_parts
FROM transparent_parts, all_parts
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>proportion_transparent_parts</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.062949</td>
    </tr>
  </tbody>
</table>
</div>



## 6. What are the 5 rarest lego bricks?

**Answer:** 5 the most rarest Lego bricks:
1) Bricks Printed (4580 released parts)
2) Bricks Wedged (9209 released parts)
3) Technic Bricks (36173 released parts)
4) Bricks Curved (38560 released parts)
5) Bricks Round and Cones (48525 released parts)

**Solution:** To calculate the five rarest categories of bricks, we need to find the groups of bricks that have been released in the lowest amount.


```python
-- find 5 rarest lego bricks

SELECT pc.name, 
	SUM(IP.quantity) AS total_usage
FROM part_categories AS pc
JOIN parts AS p ON pc.id = p.part_cat_id
JOIN inventory_parts AS ip ON p.part_num = ip.part_num
WHERE PC.name LIKE '%rick%'
GROUP BY PC.name
ORDER BY total_usage
LIMIT 5
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>total_usage</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Bricks Printed</td>
      <td>4580</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Bricks Wedged</td>
      <td>9209</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Technic Bricks</td>
      <td>36173</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Bricks Curved</td>
      <td>38560</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Bricks Round and Cones</td>
      <td>48525</td>
    </tr>
  </tbody>
</table>
</div>



## Summary

### 1. What is the average number of Lego sets released per year?
In the total span of 66 years, 11673 sets have been released. This means an average of 176 sets per year have been released.

### 2. What is the average number of Lego parts over total time span and per year?
In the total span of 66 years, a grand total of 1894089 parts have been released, an average of 28698 parts per year have been released. We can see that in 1951 and 1952 no parts were released.

The average amount per year can also be calculated and visualized to detect a trend. Referring to the graph in item 3, we can can conclude there has been an increase in released parts throughout the years.

### 3. What are the 5 most popular colors used in Lego parts? 
The most popular colors used in Lego parts are:
1. Black (115085 parts)
2. White (66536 parts)
3. Light Bluish Gray (55302 parts)
4. Red (50213 parts)
5. Dark Bluish Gray (43907 parts)

### 4. What proportion of Lego parts are transparent?
About 6.3% of the Lego bricks are transparent.

### 5. What are the 5 rarest categories of Lego bricks?
The rarest categories of Lego bricks are:
1) Bricks Printed (4580 released parts)
2) Bricks Wedged (9209 released parts)
3) Technic Bricks (36173 released parts)
4) Bricks Curved (38560 released parts)
5) Bricks Round and Cones (48525 released parts)
