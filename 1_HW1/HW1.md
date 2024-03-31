# HW1: Intro to SQL and SQLite

## Q1 Assignment Setup

## Q2 Simple Table Creation

In this question, you will create a table and insert data into it.  Recall that we are looking for SQL commands that can be copy-and-pasted directly into a database and executed without error.

#### Q2.1

Write a SQL statement creating a table Friends(source, destination, arefriends) where both source and destination are strings and arefriends is an integer.

You can assume that names are no longer than 32 characters long.  Please use VARCHAR instead of TEXT for this homework.

```sql
CREATE TABLE Friends (
    source VARCHAR(32),
    destination VARCHAR(32),
    arefriends INTEGER
);

```

#### Q2.1
Write 3 different SQL statements to insert the tuples ('Amal', 'Ciaran', 1), ('Amal', 'Bo', 1), and ('Bo', 'Ciaran', 0) into the Friends table you created in 2.1.

Please do not submit a single INSERT statement; we want 3 separate SQL statements.

```sql
INSERT INTO Friends (source, destination, arefriends) VALUES ('Amal', 'Ciaran', 1);

INSERT INTO Friends (source, destination, arefriends) VALUES ('Amal', 'Bo', 1);

INSERT INTO Friends (source, destination, arefriends) VALUES ('Bo', 'Ciaran', 0);
```


#### Q2.3
Grading comment:
Write a SQL statement that returns all tuples in your Friends table.

```sql
SELECT * FROM Friends;
```

#### Q2.4
3 Points
Grading comment:
Write a SQL statement that returns all tuples' source field (aka "returns the source column for all rows").
```sql
SELECT source FROM Friends;
```


#### Q2.5
3 Points
Grading comment:
Write a SQL statement that returns all tuples where are friends > 0.
```sql
SELECT * FROM Friends WHERE arefriends > 0;

```

#### Q2.6
4 Points
Grading comment:
Insert the tuple ('Amal', 'Ciaran', 1) into your table a second time.  Do you get an error?  Why or why not?

```sql
INSERT INTO Friends (source, destination, arefriends) VALUES ('Amal', 'Ciaran', 1);

```
Since the `Friends` table was created without any unique constraints or primary keys, inserting the tuple ('Amal', 'Ciaran', 1) a second time does not violate any rules and thus does not result in an error. The table schema allows for duplicate rows because it lacks mechanisms to enforce uniqueness among the inserted data.


#### Q2.7
6 Points
Grading comment:
Now insert the tuple ('Amal', 'Amal', '1') into your table -- note how the last field is a string instead of an integer. Do you get an error? Why or why not?

```sql 
INSERT INTO Friends (source, destination, arefriends) VALUES ('Amal', 'Amal', '1');

```

#### Q2.8
4 Points
Grading comment:
Finally, insert the tuple ('Amal', 'Amal', 'hello') into your table.  Did you get the same result as when you inserted the string '1'?
######  DOES NOT WORK
```sql
INSERT INTO Friends (source, destination, arefriends) VALUES ('Amal', 'Amal', 'hello');
```
###### 

## Q3 SQLite Formatting
10 Points
Grading comment:
For this question, we will be experimenting with a few of SQLite's output commands and formats.  Please submit both the command you use to format the output along with your actual query itself.  Note, however, that we do not want the actual output.

When we run your code, we should see the entire contents of the Friends table printed 6 times.

#### Q3.1
8 Points
Grading comment:
Turn column headers on, then output the entire table in these three formats:
1. print the results in comma-separated format
```sql
.headers on
.mode csv
SELECT * FROM Friends;
```
**output**: 
source,destination,arefriends
Amal,Ciaran,1
Amal,Bo,1
Bo,Ciaran,0
Amal,Ciaran,1
Amal,Amal,1

2. print the results in list format, delimited by ":"
```sql
.headers on
.mode list
.separator ":"
SELECT * FROM Friends;

```
**output**:
source:destination:arefriends
Amal:Ciaran:1
Amal:Bo:1
Bo:Ciaran:0
Amal:Ciaran:1
Amal:Amal:1


3. print the results in column format and make every column has a width of >= 15 (not just the first one)
```sql
.headers on
.mode column
.width 15 15 15
SELECT * FROM Friends;

```
**output**:
source           destination      arefriends     
---------------  ---------------  ---------------
Amal             Ciaran           1              
Amal             Bo               1              
Bo               Ciaran           0              
Amal             Ciaran           1              
Amal             Amal             1              


#### Q3.2
Now turn the column headers off, and output the entire table again in the same three formats described above.

```sql
.headers off
``` 

```sql
.mode csv
SELECT * FROM Friends;
``` 
**output**:
Amal,Ciaran,1
Amal,Bo,1
Bo,Ciaran,0
Amal,Ciaran,1
Amal,Amal,1


```sql
.mode list
.separator ":"
SELECT * FROM Friends;
``` 
**output**:
Amal:Ciaran:1
Amal:Bo:1
Bo:Ciaran:0
Amal:Ciaran:1
Amal:Amal:1


```sql
.mode column
.width 15 15 15
SELECT * FROM Friends;

``` 
**output**:
Amal             Ciaran           1              
Amal             Bo               1              
Bo               Ciaran           0              
Amal             Ciaran           1       
Amal             Amal             1       


## Q4 Table Creation with Types
25 Points
Grading comment:
Next, you will create a table with attributes of type integer, varchar, date, and Boolean. However! SQLite does not have date and Boolean: you will need to learn how to use varchar and int in their stead.

Some advice:

0 (false) and 1 (true) are the values used to interpret Booleans.
Date strings in SQLite are in the form: 'YYYY-MM-DD'.
Valid date strings include: '1988-01-15', '0000-12-31', and '2011-03-28'.
Invalid date strings include: '11-11-01', '1900-1-20', '2011-03-5', and '2011-03-50'.
Additionally, SQLite supports simple date formatting and manipulation.  Feel free to try any of these examples to learn about the date function.  We have also included an example of a SQL conditional.

select date('2011-03-28');
select date('now');
select date('now', '-5 year');
select date('now', '-5 year', '+24 hour');
select case when date('now') < date('2021-12-10') then 'Taking classes' when date('now') < date('2021-12-15') then 'Exams' else 'Vacation' end;


#### Q4.1
15 Points
Grading comment:
Give a SQL statement that creates a table called MyHomeworks with the following attributes. You can pick your own attribute names, just ensure we can figure out which name is for which attribute:

Name of the homework assignment: a varchar field
Name of the class the assignment is for: a varchar field
Estimated number of hours the assignment will take: an int
Due date of the assignment: a varchar field (interpreted as date, specified above)
Whether you completed it or not: an int (interpreted as a Boolean, specified above)
You will need to determine an appropriate size limit for the homework and class names when creating the table.

```sql
CREATE TABLE MyHomeworks (
    HomeworkName VARCHAR(255),
    ClassName VARCHAR(255),
    EstimatedHours INT,
    DueDate VARCHAR(10), -- YYYY-MM-DD format
    Completed INT -- 0 for false, 1 for true
);
```

#### Q4.2
10 Points
Grading comment:
Give a single SQL INSERT statement that inserts at least five tuples. You should insert at least one assignment you completed, at least one assignment that is estimated to require more than 12h of effort, and at least one homework where you leave the completion field NULL.


```sql 
INSERT INTO MyHomeworks (HomeworkName, ClassName, EstimatedHours, DueDate, Completed) VALUES
('Math Homework 1', 'Math 101', 5, '2024-04-10', 1),
('Science Project', 'Science 202', 15, '2024-05-15', 0),
('English Essay', 'English 101', 8, '2024-04-20', NULL),
('Programming Lab 4', 'Computer Science 301', 10, '2024-04-15', 1),
('History Presentation', 'History 101', 12, '2024-05-01', 0);

```


## Q5 Writing Queries
36 Points
Grading comment:
This set of questions tests your ability to write simple queries over our MyHomeworks table.

#### Q5.1
12 Points
Grading comment:
Write a SQL query that returns everything we know about all homework assignments that are due between now and a week from now; include assignments that are due exactly one week from today.  You must use the date() function to calculate the date 1 week from now.

```sql
SELECT * FROM MyHomeworks
WHERE DueDate >= date('now') AND DueDate <= date('now', '+7 days');

```
(this query returns nothing because of the dates of the original insert)

#### Q5.2
12 Points
Grading comment:
Write a SQL query that returns every attribute of every assignment that will take more than 5 hours to complete.  Your query results should include all assignments, regardless of whether they are complete or were due in the past.

```sql
SELECT * FROM MyHomeworks
WHERE EstimatedHours > 5;

```


#### Q5.3
12 Points
Grading comment:
Write a SQL query that returns only the name, class, and whether the assignment is completed for all homework assignments due now or in the past; recall that a homework can be in the past but still not completed. 

Your query should list the assignments in (ascending) alphabetical order of names.

```sql
SELECT HomeworkName, ClassName, Completed
FROM MyHomeworks
WHERE DueDate <= date('now')
ORDER BY HomeworkName ASC;

```

## Q6 Syllabus Review
16 Points
Grading comment:
Everyone has a slightly different idea of what constitutes "cheating".  In this question, we will examine what you consider cheating and the repercussions of your personal definition.











