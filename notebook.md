## 1. Inspecting the data
<p><img src="https://assets.datacamp.com/production/project_1416/img/schoolbus.jpg" alt="New York City schoolbus" height="300px" width="300px"></p>
<p>Photo by <a href="https://unsplash.com/@jannis_lucas">Jannis Lucas</a> on <a href="https://unsplash.com">Unsplash</a>.
<br></p>
<p>Every year, American high school students take SATs, which are standardized tests intended to measure literacy, numeracy, and writing skills. There are three sections - reading, math, and writing, each with a maximum score of 800 points. These tests are extremely important for students and colleges, as they play a pivotal role in the admissions process.</p>
<p>Analyzing the performance of schools is important for a variety of stakeholders, including policy and education professionals, researchers, government, and even parents considering which school their children should attend. </p>
<p>In this notebook, we will take a look at data on SATs across public schools in New York City. Our database contains a single table:</p>
<h1 id="schools"><code>schools</code></h1>
<table>
<thead>
<tr>
<th>column</th>
<th>type</th>
<th>description</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>school_name</code></td>
<td><code>varchar</code></td>
<td>Name of school</td>
</tr>
<tr>
<td><code>borough</code></td>
<td><code>varchar</code></td>
<td>Borough that the school is located in</td>
</tr>
<tr>
<td><code>building_code</code></td>
<td><code>varchar</code></td>
<td>Code for the building</td>
</tr>
<tr>
<td><code>average_math</code></td>
<td><code>int</code></td>
<td>Average math score for SATs</td>
</tr>
<tr>
<td><code>average_reading</code></td>
<td><code>int</code></td>
<td>Average reading score for SATs</td>
</tr>
<tr>
<td><code>average_writing</code></td>
<td><code>int</code></td>
<td>Average writing score for SATs</td>
</tr>
<tr>
<td><code>percent_tested</code></td>
<td><code>numeric</code></td>
<td>Percentage of students completing SATs</td>
</tr>
</tbody>
</table>
<p>Let's familiarize ourselves with the data by taking a looking at the first few schools!</p>


```sql
%%sql
postgresql:///schools
    
-- Select all columns from the database
-- Display only the first ten rows

SELECT *
FROM schools
LIMIT 10
```

    10 rows affected.





<table>
    <thead>
        <tr>
            <th>school_name</th>
            <th>borough</th>
            <th>building_code</th>
            <th>average_math</th>
            <th>average_reading</th>
            <th>average_writing</th>
            <th>percent_tested</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>New Explorations into Science, Technology and Math High School</td>
            <td>Manhattan</td>
            <td>M022</td>
            <td>657</td>
            <td>601</td>
            <td>601</td>
            <td>None</td>
        </tr>
        <tr>
            <td>Essex Street Academy</td>
            <td>Manhattan</td>
            <td>M445</td>
            <td>395</td>
            <td>411</td>
            <td>387</td>
            <td>78.9</td>
        </tr>
        <tr>
            <td>Lower Manhattan Arts Academy</td>
            <td>Manhattan</td>
            <td>M445</td>
            <td>418</td>
            <td>428</td>
            <td>415</td>
            <td>65.1</td>
        </tr>
        <tr>
            <td>High School for Dual Language and Asian Studies</td>
            <td>Manhattan</td>
            <td>M445</td>
            <td>613</td>
            <td>453</td>
            <td>463</td>
            <td>95.9</td>
        </tr>
        <tr>
            <td>Henry Street School for International Studies</td>
            <td>Manhattan</td>
            <td>M056</td>
            <td>410</td>
            <td>406</td>
            <td>381</td>
            <td>59.7</td>
        </tr>
        <tr>
            <td>Bard High School Early College</td>
            <td>Manhattan</td>
            <td>M097</td>
            <td>634</td>
            <td>641</td>
            <td>639</td>
            <td>70.8</td>
        </tr>
        <tr>
            <td>Urban Assembly Academy of Government and Law</td>
            <td>Manhattan</td>
            <td>M445</td>
            <td>389</td>
            <td>395</td>
            <td>381</td>
            <td>80.8</td>
        </tr>
        <tr>
            <td>Marta Valle High School</td>
            <td>Manhattan</td>
            <td>M025</td>
            <td>438</td>
            <td>413</td>
            <td>394</td>
            <td>35.6</td>
        </tr>
        <tr>
            <td>University Neighborhood High School</td>
            <td>Manhattan</td>
            <td>M446</td>
            <td>437</td>
            <td>355</td>
            <td>352</td>
            <td>69.9</td>
        </tr>
        <tr>
            <td>New Design High School</td>
            <td>Manhattan</td>
            <td>M445</td>
            <td>381</td>
            <td>396</td>
            <td>372</td>
            <td>73.7</td>
        </tr>
    </tbody>
</table>




```python
%%nose
last_output = _

def test_task1_output_type():
    assert str(type(last_output)) == "<class 'sql.run.ResultSet'>", \
    "Please ensure an SQL ResultSet is the output of the code cell." 

results = last_output.DataFrame()

def test_task1_results():
    assert results.shape == (10, 7), \
    "The results should have fourteen columns and ten rows."
    assert set(results.columns) == set(['school_name', 'borough', 'building_code', 'average_math', 'average_reading', 'average_writing', 'percent_tested']), \
    'The results should include all columns from the database, without using an alias.'
    assert last_output.DataFrame().loc[0, 'building_code'] == "M022", \
    "The building code for the first school should be M022."
```






    2/2 tests passed




## 2. Finding missing values
<p>It looks like the first school in our database had no data in the <code>percent_tested</code> column! </p>
<p>Let's identify how many schools have missing data for this column, indicating schools that did not report the percentage of students tested. </p>
<p>To understand whether this missing data problem is widespread in New York, we will also calculate the total number of schools in the database.</p>


```sql
%%sql

-- Count rows with percent_tested missing and total number of schools

SELECT COUNT(*)-COUNT(percent_tested) num_tested_missing, count(*) num_schools
FROM schools

```

     * postgresql:///schools
    1 rows affected.





<table>
    <thead>
        <tr>
            <th>num_tested_missing</th>
            <th>num_schools</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>20</td>
            <td>375</td>
        </tr>
    </tbody>
</table>




```python
%%nose
last_output = _
last_output_df = last_output.DataFrame()

def test_task2_columns():
    assert last_output_df.shape == (1, 2), \
    "Did you correctly select the data? Expected the result to contain one row and two columns?"
    assert set(last_output_df.columns) == set(["num_tested_missing", "num_schools"]), \
    "Did you use the alias `num_tested_missing` and also select the `num_schools` column?"

def test_task2_output():
    assert last_output_df.iloc[0, 0] == 20, \
    """Did you correctly calculate `"num_tested_missing"?"""
    assert last_output_df.iloc[0, 1] == 375, \
    """Did you correctly calculate the total number of rows in the database?"""  
```






    2/2 tests passed




## 3. Schools by building code
<p>There are 20 schools with missing data for <code>percent_tested</code>, which only makes up 5% of all rows in the database.</p>
<p>Now let's turn our attention to how many schools there are. When we displayed the first ten rows of the database, several had the same value in the <code>building_code</code> column, suggesting there are multiple schools based in the same location. Let's find out how many unique school locations exist in our database. </p>


```sql
%%sql

-- Count the number of unique building_code values

SELECT COUNT(DISTINCT building_code) num_school_buildings
FROM schools
```

     * postgresql:///schools
    1 rows affected.





<table>
    <thead>
        <tr>
            <th>num_school_buildings</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>233</td>
        </tr>
    </tbody>
</table>




```python
%%nose
last_output = _
last_output_df = last_output.DataFrame()

def test_task3_column_name():
    assert last_output_df.columns.tolist() == ["num_school_buildings"], \
    "Did you use the correct alias for the number of unique school buildings?"

def test_task3_value():
    assert last_output_df.values.tolist() == [[233]], \
    "Did you use the correct method to calculate how many unique school buildings are in the database? Expected a different value."
```






    2/2 tests passed




## 4. Best schools for math
<p>Out of 375 schools, only 233 (62%) have a unique <code>building_code</code>! </p>
<p>Now let's start our analysis of school performance. As each school reports individually, we will treat them this way rather than grouping them by <code>building_code</code>. </p>
<p>First, let's find all schools with an average math score of at least 80% (out of 800). </p>


```sql
%%sql

-- Select school and average_math
-- Filter for average_math 640 or higher
-- Display from largest to smallest average_math

SELECT school_name, average_math
FROM schools
WHERE average_math >= 800*0.8
ORDER BY average_math DESC
```

     * postgresql:///schools
    10 rows affected.





<table>
    <thead>
        <tr>
            <th>school_name</th>
            <th>average_math</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Stuyvesant High School</td>
            <td>754</td>
        </tr>
        <tr>
            <td>Bronx High School of Science</td>
            <td>714</td>
        </tr>
        <tr>
            <td>Staten Island Technical High School</td>
            <td>711</td>
        </tr>
        <tr>
            <td>Queens High School for the Sciences at York College</td>
            <td>701</td>
        </tr>
        <tr>
            <td>High School for Mathematics, Science, and Engineering at City College</td>
            <td>683</td>
        </tr>
        <tr>
            <td>Brooklyn Technical High School</td>
            <td>682</td>
        </tr>
        <tr>
            <td>Townsend Harris High School</td>
            <td>680</td>
        </tr>
        <tr>
            <td>High School of American Studies at Lehman College</td>
            <td>669</td>
        </tr>
        <tr>
            <td>New Explorations into Science, Technology and Math High School</td>
            <td>657</td>
        </tr>
        <tr>
            <td>Eleanor Roosevelt High School</td>
            <td>641</td>
        </tr>
    </tbody>
</table>




```python
%%nose
last_output = _
last_output_df = last_output.DataFrame()

def test_task4_columns():
    assert set(last_output_df.columns) == set(["school_name", "average_math"]), \
    "Did you select the correct columns?"

def test_task4_filter():
    assert last_output_df["average_math"].min() >= 640, \
    """Did you correctly filter for "average_math" scores more than or equal to 640?"""
    assert last_output_df.shape == (10, 2), \
    """The output has the wrong number of results, did you correctly filter the "average_math" column?"""

def test_task4_values():
    assert last_output_df.iloc[0,0] == "Stuyvesant High School", \
    """Did you run the correct query? Expected the first school to be "Stuyvesant High School"."""
    assert last_output_df.iloc[0,1] == 754.0, \
    """Did you correctly sort the values by "average_math" in descending order? Expected a different range of results."""
```






    3/3 tests passed




## 5. Lowest reading score
<p>Wow, there are only ten public schools in New York City with an average math score of at least 640!</p>
<p>Now let's look at the other end of the spectrum and find the single lowest score for reading. We will only select the score, not the school, to avoid naming and shaming!</p>


```sql
%%sql

-- Find lowest average_reading

SELECT min(average_reading) lowest_reading
FROM schools
```

     * postgresql:///schools
    1 rows affected.





<table>
    <thead>
        <tr>
            <th>lowest_reading</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>302</td>
        </tr>
    </tbody>
</table>




```python
%%nose
last_output = _
last_output_df = last_output.DataFrame()

def test_task5_value():  
    assert last_output_df["lowest_reading"].values.tolist() == [302.0], \
    """Did you select the minimum value for the "average_reading" column?"""

def test_task5_alias():
    assert last_output_df.columns.tolist() == ["lowest_reading"], \
    """Did you use the correct alias? Expected "lowest_reading"."""
```






    2/2 tests passed




## 6. Best writing school
<p>The lowest average score for reading across schools in New York City is less than 40% of the total available points! </p>
<p>Now let's find the school with the highest average writing score.</p>


```sql
%%sql

-- Find the top score for average_writing
-- Group the results by school
-- Sort by max_writing in descending order
-- Reduce output to one school

SELECT school_name, average_writing max_writing
FROM schools
ORDER BY max_writing DESC
LIMIT 1
```

     * postgresql:///schools
    1 rows affected.





<table>
    <thead>
        <tr>
            <th>school_name</th>
            <th>max_writing</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Stuyvesant High School</td>
            <td>693</td>
        </tr>
    </tbody>
</table>




```python
%%nose
last_output = _
last_output_df = last_output.DataFrame()

def test_task6_columns():
    assert set(last_output_df.columns) == set(["school_name", "max_writing"]), \
    """Did you select "average_writing" and use an alias?"""
    
def test_task6_shape():
    assert last_output_df.shape[0] == 1, \
    "Did you select the correct number of values? Expected one row."

def test_task6_values():
    assert last_output_df.values.tolist() == [['Stuyvesant High School', 693.0]], \
    """Did you select the maximum value for "average_writing"? Expected a different value."""  
```






    3/3 tests passed




## 7. Top 10 schools
<p>An average writing score of 693 is pretty impressive! </p>
<p>This top writing score was at the same school that got the top math score, Stuyvesant High School. Stuyvesant is widely known as a perennial top school in New York. </p>
<p>What other schools are also excellent across the board? Let's look at scores across reading, writing, and math to find out.</p>


```sql
%%sql

-- Calculate average_sat
-- Group by school_name
-- Sort by average_sat in descending order
-- Display the top ten results

SELECT school_name, sum(average_math + average_reading + average_writing) average_sat
FROM schools
GROUP BY school_name
ORDER BY average_sat DESC
LIMIT 10
```

     * postgresql:///schools
    10 rows affected.





<table>
    <thead>
        <tr>
            <th>school_name</th>
            <th>average_sat</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Stuyvesant High School</td>
            <td>2144</td>
        </tr>
        <tr>
            <td>Staten Island Technical High School</td>
            <td>2041</td>
        </tr>
        <tr>
            <td>Bronx High School of Science</td>
            <td>2041</td>
        </tr>
        <tr>
            <td>High School of American Studies at Lehman College</td>
            <td>2013</td>
        </tr>
        <tr>
            <td>Townsend Harris High School</td>
            <td>1981</td>
        </tr>
        <tr>
            <td>Queens High School for the Sciences at York College</td>
            <td>1947</td>
        </tr>
        <tr>
            <td>Bard High School Early College</td>
            <td>1914</td>
        </tr>
        <tr>
            <td>Brooklyn Technical High School</td>
            <td>1896</td>
        </tr>
        <tr>
            <td>Eleanor Roosevelt High School</td>
            <td>1889</td>
        </tr>
        <tr>
            <td>High School for Mathematics, Science, and Engineering at City College</td>
            <td>1889</td>
        </tr>
    </tbody>
</table>




```python
%%nose
last_output = _
last_output_df = last_output.DataFrame()

def test_task7_columns():
    assert set(last_output_df.columns) == set(["school_name", "average_sat"]), \
    """Did you select the correct columns and use an alias for the sum of the three sat score columns?"""
    
def test_task7_shape():
    assert last_output_df.shape[0] == 10, \
    "Did you limit the number of results to ten?"
    assert last_output_df.shape[1] == 2, \
    """Expected your query to return two columns: "school_name" and "average_sat"."""

def test_task7_values():
    assert last_output_df.iloc[0].values.tolist() == ['Stuyvesant High School', 2144], \
    """Did you correctly define your query? Expected different values for the first school."""
    assert last_output_df["average_sat"].min() == 1889, \
    """Did you correctly filter the results? Expected a different lowest score for "average_sat"."""  
    assert last_output_df["average_sat"].max() == 2144, \
    """Did you correctly calculate the "average_sat" column? Expected a different top score."""
```






    3/3 tests passed




## 8. Ranking boroughs
<p>There are four schools with average SAT scores of over 2000! Now let's analyze performance by New York City borough. </p>
<p>We will build a query that calculates the number of schools and the average SAT score per borough!</p>


```sql
%%sql

-- Select borough and a count of all schools, aliased as num_schools
-- Calculate the sum of average_math, average_reading, and average_writing, divided by a count of all schools, aliased as average_borough_sat
-- Organize results by borough
-- Display by average_borough_sat in descending order

SELECT borough, count(*) num_schools, sum(average_math + average_reading + average_writing)/count(*) average_borough_sat
FROM schools
GROUP BY borough
ORDER BY average_borough_sat DESC

```

     * postgresql:///schools
    5 rows affected.





<table>
    <thead>
        <tr>
            <th>borough</th>
            <th>num_schools</th>
            <th>average_borough_sat</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Staten Island</td>
            <td>10</td>
            <td>1439</td>
        </tr>
        <tr>
            <td>Queens</td>
            <td>69</td>
            <td>1345</td>
        </tr>
        <tr>
            <td>Manhattan</td>
            <td>89</td>
            <td>1340</td>
        </tr>
        <tr>
            <td>Brooklyn</td>
            <td>109</td>
            <td>1230</td>
        </tr>
        <tr>
            <td>Bronx</td>
            <td>98</td>
            <td>1202</td>
        </tr>
    </tbody>
</table>




```python
%%nose
last_output = _
last_output_df = last_output.DataFrame()

def test_task8_columns():
    assert set(last_output_df.columns) == set(['borough', 'num_schools', 'average_borough_sat']), \
    """Did you select the correct columns and use aliases for the number of schools and average sat scores?"""

def test_task8_shape():
    assert last_output_df.shape[0] == 5, \
    "Did you group by the correct column? Expected five rows to be returned: one for each borough."
    assert last_output_df.shape[1] == 3, \
    """Expected your query to return three columns: "borough", "num_schools", and "average_borough_sat"."""

def test_task8_values():
    # Each assert statement checks values per row 
    assert last_output_df.iloc[0].values.tolist() == ['Staten Island', 10, 1439], \
    """Did you correctly define your query? Expected different values for Staten Island."""
    assert last_output_df.iloc[1].values.tolist() == ['Queens', 69, 1345], \
    """Did you correctly define your query? Expected different values for Queens."""
    assert last_output_df.iloc[2].values.tolist() == ['Manhattan', 89, 1340], \
    """Did you correctly define your query? Expected different values for Manhattan."""
    assert last_output_df.iloc[3].values.tolist() == ['Brooklyn', 109, 1230], \
    """Did you correctly define your query? Expected different values for Brooklyn."""
    assert last_output_df.iloc[4].values.tolist() == ['Bronx', 98, 1202], \
    """Did you correctly define your query? Expected different values for the Bronx."""
    # Check lowest average_reading score is in the last row
    assert last_output_df.iloc[-1, 0] == 'Bronx', \
    """Did you sort the results by "average_sat" in descending order?"""
```






    3/3 tests passed




## 9. Brooklyn numbers
<p>It appears that schools in Staten Island, on average, produce higher scores across all three categories. However, there are only 10 schools in Staten Island, compared to an average of 91 schools in the other four boroughs!</p>
<p>For our final query of the database, let's focus on Brooklyn, which has 109 schools. We wish to find the top five schools for math performance.</p>


```sql
%%sql

-- Select school and average_math
-- Filter for schools in Brooklyn
-- Aggregate on school_name
-- Display results from highest average_math and restrict output to five rows

SELECT school_name, average_math
FROM schools
WHERE borough = 'Brooklyn'
ORDER BY average_math DESC
LIMIT 5
```

     * postgresql:///schools
    5 rows affected.





<table>
    <thead>
        <tr>
            <th>school_name</th>
            <th>average_math</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Brooklyn Technical High School</td>
            <td>682</td>
        </tr>
        <tr>
            <td>Brooklyn Latin School</td>
            <td>625</td>
        </tr>
        <tr>
            <td>Leon M. Goldstein High School for the Sciences</td>
            <td>563</td>
        </tr>
        <tr>
            <td>Millennium Brooklyn High School</td>
            <td>553</td>
        </tr>
        <tr>
            <td>Midwood High School</td>
            <td>550</td>
        </tr>
    </tbody>
</table>




```python
%%nose
last_output = _
last_output_df = last_output.DataFrame()

def test_task9_columns():
    assert last_output_df.columns.tolist() == ['school_name', 'average_math'], \
    """Did you select the correct columns? Expected "school_name" and "average_math"."""
    
def test_task9_shape():
    assert last_output_df.shape[0] == 5, \
    "Did you limit the output to 5 rows?"
    assert last_output_df.shape[1] == 2, \
    "Did you select the correct number of columns? Expected two."
    
def test_task9_school_names():
    assert last_output_df["school_name"].tolist() == ['Brooklyn Technical High School', 'Brooklyn Latin School', 'Leon M. Goldstein High School for the Sciences', 'Millennium Brooklyn High School', 'Midwood High School'], \
    "Did you correctly filter by borough? Expected a different list of school names."
    
def test_task9_values():
    assert last_output_df["average_math"].max() == 682, \
    """Did you select the correct values? Expected a maximum value of 682.0 for "average_math"."""    
    assert last_output_df["average_math"].min() == 550, \
    """Did you select the correct values? Expected a minimum value of 550.0 for "average_math"."""
    assert last_output_df["average_math"].values.tolist() == [682, 625, 563, 553, 550], \
    """Did you sort by "average_math" in descending order? Expected different values."""
```






    4/4 tests passed



