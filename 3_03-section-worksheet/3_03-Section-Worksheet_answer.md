# DATA 514 Section 3 Worksheet - Jonathan Jacobs

#### Question 1

```sql
SELECT DISTINCT I.fname, I.lname
FROM Instructor I
JOIN Teaches T ON I.username = T.username;
```

#### Question 2

```sql
SELECT I.username, COUNT(T.dept) AS num_classes
FROM Instructor I
LEFT JOIN Teaches T ON I.username = T.username
GROUP BY I.username;
```

#### Question 3

```sql
SELECT I.username, COUNT(T.username) AS num_classes
FROM Instructor I
LEFT JOIN Teaches T ON I.username = T.username
GROUP BY I.username;
```

#### Question 4

```sql
SELECT C.dept, MAX(C.number) AS max_number
FROM Class C
GROUP BY C.dept;
```

#### Question 5

```sql
SELECT COUNT(DISTINCT T.username) AS num_instructors
FROM Teaches T;
```

#### Question 6

```sql
SELECT I.fname, I.lname
FROM Instructor I
WHERE I.started_on = (SELECT MAX(started_on) FROM Instructor);
```

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







