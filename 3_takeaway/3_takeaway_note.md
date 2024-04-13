
# NOTES NOTES NOTES NOTES
# Takeaway Sheet 3 (Lectures 7-9) - Jonathan Jacobs
# NOTES NOTES NOTES NOTES


## Tables for Lecture 8 

Payroll table
userid  name	Job  	Salary
123		Leslie 	TA      50k
345		Frances TA   	60k
567		Magda	Prof	120k
789		Quinn	Prof	100k


Regist Table
userid	Car 			Year
123		Charger 		2009		
567		Civic			2016
567		Ferrari 		2000
789		Picklemobile	2018	 


### L8.1
To answer the question using a correlated subquery, you would generally structure your SQL query in a way that counts the number of cars associated with each user in the Regist table for every user listed in the Payroll table. This approach allows you to relate the two tables based on their common userid field. The correlated subquery will execute once for each row in the outer query (in this case, for each userid in the Payroll table), using the userid from the outer query to filter the results of the inner query.

Here's how you can structure the SQL query:

```sql
SELECT 
    p.userid,
    p.name,
    (SELECT COUNT(*) 
     FROM Regist r 
     WHERE r.userid = p.userid) AS NumberOfCars
FROM Payroll p;
```


Explanation of the query:
- The outer query selects from the `Payroll` table (`p` is the alias for `Payroll`).
- For each row in the `Payroll` table, the subquery counts the number of entries in the `Regist` table (`r` is the alias for `Regist`) that have the same `userid`.
- The result of the subquery, which is the count of cars for each `userid`, is shown in the `NumberOfCars` column in the final output.
- This query effectively correlates the count of cars from the `Regist` table for each person listed in the `Payroll` table by matching the `userid` in both tables.



### L8.2

#### Logical Proposition

Let \(P(x)\) be the proposition "Person \(x\) drives at least one car". We are interested in people for whom \(P(x)\) is false. Using the given tables, a person who does not drive any car will not have an entry in the `Regist` table. Therefore, the logical proposition for people who do not drive cars is the negation of \(P(x)\), represented as \(\neg P(x)\).

####  SQL Query

To find people who do not drive cars and return their name and salary, we can use a `LEFT JOIN` operation between the `Payroll` and `Regist` tables and check for those who have no matching entries in the `Regist` table. This approach is preferred over checking for an empty set directly, as hinted.

Here's how the SQL query can be structured:

```sql
SELECT 
    p.name,
    p.Salary
FROM 
    Payroll p
LEFT JOIN 
    Regist r ON p.userid = r.userid
WHERE 
    r.userid IS NULL;
```

Explanation of the query:
- The query performs a `LEFT JOIN` of the `Payroll` table (`p`) with the `Regist` table (`r`) based on the `userid`. This join ensures that all entries from the `Payroll` table are included in the result set, along with matching entries from the `Regist` table (if any).
- The `WHERE` clause filters the result to include only those rows where there is no matching `userid` in the `Regist` table, indicated by `r.userid IS NULL`. This condition effectively selects people who do not have any cars registered under their name.
- The `SELECT` statement specifies that only the `name` and `Salary` of such individuals are returned in the final result.

This query addresses the requirement by identifying individuals in the `Payroll` table who do not have corresponding entries in the `Regist` table, thereby indicating they do not drive (or at least do not have a car registered).


### L8.3

**Is the Query Monotone?**
A query is considered monotone if, whenever its input is extended, the output is extended as well. In other words, adding more data to the input of a monotone query cannot result in a smaller output set; it can only make the output set the same size or larger.

**Demonstrating Monotonicity**
* If we add another person with an existing job, say another "TA" with a different userid, the count for the "TA" group will increase.
* Alternatively, if we add a person with a new job title not currently in the Payroll table, such as "Admin", this will introduce a new job group to the results.

Let's say the original Payroll table looked like this:
| userid | name    | job  | Salary |
|--------|---------|------|--------|
| 123    | Leslie  | TA   | 50k    |
| 345    | Frances | TA   | 60k    |
| 567    | Magda   | Prof | 120k   |

And we add a record:
| userid | name    | job  | Salary |
|--------|---------|------|--------|
| 890  	 | Alex    | TA   | 70k   |

After adding Alex as another TA, the query's result set extends by increasing the count of TAs, demonstrating the query's monotonicity.


```sql
SELECT 
    p.job, COUNT(*),
FROM 
    Payroll as p
Group by p.job 

```


## Lecture 9
### L9.1
Find the number of each car each person drives (Including Frances Quinn!)
```sql
SELECT
    p.name,
    IFNULL(r.Car, 'No Car') as Car,
    COUNT(DISTINCT r.Car) as Count
FROM
    Payroll p
LEFT JOIN
    Regist r ON p.userid = r.userid
GROUP BY
    p.name
ORDER BY
    p.name;
```
### L9.2
Got it, let's approach this within a SQL context. To solve the problem of selecting each driver who drives all the vehicles listed in the Car table, we'll indeed use a relational division approach. There isn't a direct "division" operation in SQL, but we can simulate this with a combination of JOIN, GROUP BY, and a conditional count that matches the total number of distinct cars in the Car table.

Here’s a conceptual SQL query that does this:

```sql
SELECT d.driver
FROM Driver d
JOIN Car c ON d.Car = c.car
GROUP BY d.driver
HAVING COUNT(DISTINCT d.Car) = (SELECT COUNT(*) FROM Car);
```
Explanation:
JOIN Operation: This query starts by joining the Driver table with the Car table on the car type. This ensures we only consider the car types that are present in both tables.
GROUP BY: We group the results by driver to apply aggregate functions per driver.
HAVING Clause: This is where the division concept is applied. For a driver to be included in the result set, the number of distinct cars they drive (after the join, which filters out cars not in the Car table) must equal the total number of distinct cars in the Car table. This is checked using COUNT(DISTINCT d.Car) against the total count of cars from the Car table obtained via a subquery.
This query effectively finds drivers who have an entry for every car listed in the Car table, thus meeting the criteria of driving all vehicles listed.


### L9.3
```sql
SELECT * 
FROM Toys
WHERE price < 1000
AND (size = 2 OR color = 'red')

```


To find out how many records would be returned by the given SQL query:

```sql
SELECT * 
FROM Toys
WHERE price < 1000
AND (size = 2 OR color = 'red')
```

We'll apply the same logic to the table data provided, keeping in mind that the table you've mentioned as "Toys" seems to actually relate to the table we've just created. Assuming this, and correcting for the fact the table we're working with isn't named "Toys" in our current context, let's filter the data according to the query's conditions:

- The `price` must be less than 1000.
- The `size` must be equal to 2, or the `color` must be 'red'.

Let's evaluate this:

Based on the criteria specified in the query:

- The price must be less than $1000.
- The size must be 2 or the color must be 'red'.

No records from the provided table meet both of these conditions. Therefore, the query would return 0 records. 

It's worth noting that while we have a record with the color 'red' (the Bicycle), it doesn't have a price listed (it's `null`/`NaN`), and no record has a size of 2. This is why none match the query criteria.


