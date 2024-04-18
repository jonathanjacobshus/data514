# NOTES NOTES NOTES NOTES NOTES NOTES
# NOTES NOTES NOTES NOTES NOTES NOTES

# DATA 514 Section 3 Worksheet - Jonathan Jacobs

These tables will be used throughout this worksheet.

```sql
CREATE TABLE Class (
    dept   VARCHAR(50),
    number INT,
    title  VARCHAR(50),
    PRIMARY KEY (dept, number));


CREATE TABLE Instructor (
	username  VARCHAR(50) PRIMARY KEY,
	fname VARCHAR(50),
	lname  VARCHAR(50),
	started_on CHAR(10));

CREATE TABLE Teaches(
    username VARCHAR(50),  -- alternately, can use the
REFERENCES syntax here
    dept     VARCHAR(50),
    number   INT,
    PRIMARY KEY (username, dept, number),
    FOREIGN KEY (username) REFERENCES Instructor,
    FOREIGN KEY (dept, number) REFERENCES Class);
```

#### Question 1
Write an SQL query that will return the first and last name of all instructors who are teaching at least one class."
```sql
SELECT DISTINCT I.fname, I.lname
FROM Instructor I
JOIN Teaches T ON I.username = T.username;
```
### Explanation:
- **`SELECT DISTINCT I.fname, I.lname`**: This part of the query selects the first name (`fname`) and last name (`lname`) of the instructors from the `Instructor` table. The `DISTINCT` keyword ensures that if an instructor teaches multiple classes, their name appears only once in the result.
- **`FROM Instructor I`**: This clause specifies that the data is coming from the `Instructor` table, and it gives the table an alias `I` for easier reference in other parts of the query.
- **`JOIN Teaches T ON I.username = T.username`**: This `JOIN` operation links the `Instructor` table with the `Teaches` table based on the `username` field. This ensures that only those instructors who have at least one teaching assignment in the `Teaches` table are included in the results.

#### Question 2
For each instructor, write an SQL query that will return their username and the number of classes they teach.

```sql
SELECT I.username, COUNT(T.dept) AS num_classes
FROM Instructor I
LEFT JOIN Teaches T ON I.username = T.username
GROUP BY I.username;
```

### Explanation:
- **`SELECT I.username, COUNT(T.dept) AS num_classes`**: This part of the query selects the username from the `Instructor` table and counts the number of departments (`dept`) entries in the `Teaches` table for each instructor. `num_classes` is an alias for the count of classes taught.
- **`FROM Instructor I`**: Specifies the data is coming from the `Instructor` table.
- **`LEFT JOIN Teaches T ON I.username = T.username`**: This joins the `Instructor` table to the `Teaches` table using a `LEFT JOIN`. This type of join ensures that all instructors are included in the result set, even if they do not match any rows in the `Teaches` table (i.e., they do not teach any classes).
- **`GROUP BY I.username`**: This clause groups the results by the instructor’s username. Since the `COUNT()` function is an aggregate function, the `GROUP BY` is necessary to ensure that each instructor’s count of classes is calculated separately.


#### Question 3

```sql
SELECT I.username, COUNT(T.username) AS num_classes
FROM Instructor I
LEFT JOIN Teaches T ON I.username = T.username
GROUP BY I.username;
```

### Explanation:
- **`SELECT I.username, COUNT(T.username) AS num_classes`**: This selects the `username` from the `Instructor` table and counts the occurrences of `username` in the `Teaches` table. This count gives the number of classes each instructor teaches.
- **`FROM Instructor I`**: Specifies that the data comes from the `Instructor` table.
- **`LEFT JOIN Teaches T ON I.username = T.username`**: This clause joins the `Instructor` table with the `Teaches` table based on the `username`. The use of `LEFT JOIN` ensures that all instructors are included in the output, even those who do not teach any classes (they will show a count of zero).
- **`GROUP BY I.username`**: The results are grouped by the instructor’s username to ensure that the count of classes taught is calculated for each instructor individually.

This query achieves the same result as the original query that used a subquery but in a more efficient manner by directly joining the tables and using aggregation. This method is typically faster and easier for SQL engines to optimize, especially with large datasets. If you have more queries or need further modifications, feel free to ask!


#### Question 4

Here’s how to write this query without using subqueries:

```sql
SELECT C.dept, MAX(C.number) AS max_number
FROM Class C
GROUP BY C.dept;
```

### Explanation:
- **`SELECT C.dept, MAX(C.number) AS max_number`**: This part of the query selects the department (`dept`) and the maximum course number (`number`) from the `Class` table. This effectively finds the highest course number in each department.
- **`FROM Class C`**: Specifies the data source, which is the `Class` table, with `C` as an alias for easier reference.
- **`GROUP BY C.dept`**: This clause groups the results by department. The `MAX()` function then operates within each group, returning the highest course number for each department.

This rewritten query is efficient and concise. It directly accomplishes what the original query does, which is to find the class with the highest number in each department, without using a subquery that checks against all other class numbers in the same department. If you need further adjustments or have more questions, just let me know!


#### Question 5

```sql
SELECT COUNT(DISTINCT T.username) AS num_instructors
FROM Teaches T;
```

### Explanation:
- **`SELECT COUNT(DISTINCT T.username) AS num_instructors`**: This part of the query counts the number of unique usernames in the `Teaches` table. The `DISTINCT` keyword ensures that each instructor is counted only once, regardless of how many classes they teach.
- **`FROM Teaches T`**: This clause specifies that the data is coming from the `Teaches` table. The alias `T` is used for simplicity.

This query directly counts the distinct instructors listed in the `Teaches` table, effectively giving you the number of instructors who are currently teaching at least one class. If you need further explanation or additional queries, feel free to ask!


#### Question 6

```sql
SELECT I.fname, I.lname
FROM Instructor I
WHERE I.started_on = (SELECT MAX(started_on) FROM Instructor);
```

### Explanation:
- **`SELECT I.fname, I.lname`**: This selects the first name (`fname`) and last name (`lname`) of the instructors.
- **`FROM Instructor I`**: This specifies that the data comes from the `Instructor` table, with `I` used as an alias for simplicity.
- **`WHERE I.started_on = (SELECT MAX(started_on) FROM Instructor)`**: The `WHERE` clause filters the instructors to include only those who started on the latest date. The subquery `(SELECT MAX(started_on) FROM Instructor)` finds the most recent start date in the `Instructor` table.

This query efficiently identifies and lists all instructors who have the most recent start date in the `Instructor` table. It's a straightforward way to ensure that all relevant instructors are included, especially useful when multiple instructors might have started on the same date. If you have any more questions or need further modifications to the query, feel free to ask!


#### Question 7

```sql
-- Creating a subquery to count the number of classes each instructor teaches.
WITH InstructorClassCounts AS (
    SELECT username, COUNT(*) AS num_classes
    FROM Teaches
    GROUP BY username
),

-- Determining the maximum number of classes taught by any instructor.
MaxClassCount AS (
    SELECT MAX(num_classes) AS max_classes
    FROM InstructorClassCounts
)

-- Selecting the username, first name, and last name of instructors who teach the maximum number of classes.
SELECT I.username, I.fname, I.lname
FROM Instructor I
JOIN InstructorClassCounts ICC ON I.username = ICC.username
JOIN MaxClassCount MCC ON ICC.num_classes = MCC.max_classes;

```

### Explanation:
- **`InstructorClassCounts` CTE (Common Table Expression)**: This part of the query calculates the number of classes each instructor teaches. It groups the `Teaches` table by `username` and counts the number of records for each, indicating how many classes they teach.
- **`MaxClassCount` CTE**: This subquery finds the maximum number of classes taught by any instructor using the counts calculated in the `InstructorClassCounts`.
- **Final `SELECT`**: This statement joins the `Instructor` table with the `InstructorClassCounts` to get the detailed information (username, first name, last name) and filters to only include those who match the maximum class count from the `MaxClassCount`.

This approach ensures you capture all instructors who tie for teaching the most classes, providing a comprehensive list. If you have any further questions or need additional details, feel free to ask!


#### Question 8

```sql
-- Select courses from the Class table that are not taught by either Dr. Lyle or Dr. Weatherwax.
SELECT C.dept, C.number, C.title
FROM Class C
WHERE C.dept = 'CSE' AND NOT EXISTS (
    -- Check for any matching teaching records for Dr. Lyle or Dr. Weatherwax.
    SELECT 1
    FROM Teaches T
    WHERE T.dept = C.dept AND T.number = C.number
    AND T.username IN ('lyle', 'jdw')
);
```

### Explanation:
- **`SELECT C.dept, C.number, C.title`**: This selects the department code, course number, and title from the `Class` table.
- **`FROM Class C`**: Specifies that the data comes from the `Class` table, aliasing it as `C` for easier reference.
- **`WHERE C.dept = 'CSE'`**: Limits the courses to those in the Computer Science and Engineering (CSE) department.
- **`AND NOT EXISTS (...)`**: The `NOT EXISTS` clause checks for the absence of any records that meet the nested query's conditions. This is used to filter out courses that are taught by either of the two instructors.
  - **`SELECT 1`**: This part of the nested query isn't concerned with what data is returned, just whether any rows exist.
  - **`FROM Teaches T`**: The nested query checks the `Teaches` table, aliased as `T`.
  - **`WHERE T.dept = C.dept AND T.number = C.number`**: This condition links the `Teaches` table with the `Class` table based on department and course number.
  - **`AND T.username IN ('lyle', 'jdw')`**: Limits the search to the courses taught by either Dr. Lyle or Dr. Weatherwax.

This query effectively lists all CSE courses that are not taught by Dr. Lyle or Dr. Weatherwax, ensuring that it only includes those courses that both instructors do not teach. If you have any more queries or need further explanations, feel free to ask!









