# Takeaway Sheet 5 - Jonathan Jacobs

### Question 1
A full table scan is slow because it requires the database system to read and evaluate every row in a table, which is inefficient for large datasets. This process involves extensive disk I/O operations, heavy CPU usage, and does not utilize indexes, leading to increased resource consumption and slower response times. Additionally, full table scans can disrupt cache efficiency and reduce database concurrency, further impacting performance.


### Question 2

1. **Effect**: **Faster**
   **Reason**: If an index exists on `fid`, the database can quickly locate the flight without scanning the entire table. `fid` is likely a primary key, and primary keys are typically indexed by default, which drastically improves the search speed.

2. **Effect**: **Faster**
   **Reason**: With an index on `date`, the database system can efficiently find all records up to a certain date using the index. This is significantly faster than scanning all rows. Without an index, this operation would be much slower.

3. **Effect**: **Faster**
   **Reason**: An index on the `dest` column would allow the database to quickly find all flights going to 'NYC' without scanning the entire table. This would be particularly useful if the dataset is large.

4. **Effect**: **Slower**
   **Reason**: While indexes speed up data retrieval, they slow down insert operations. Each time a new record is added, the database must update all indexes where the inserted data is relevant, which adds overhead to the insert operation.


### Question 3

**Effect**: **Same**
**Reason**: An index on `(date, origin)` is not optimal for this particular query because the query filters solely by `origin`. For a composite index to be effective, the query should utilize the first component of the index, or a combination starting from the first indexed column. In this case, since `date` is not part of the query condition, the database might not efficiently utilize the `(date, origin)` index. Thus, this query's performance is unlikely to improve significantly with this specific index and might perform the same as if no index were used.


### Question 4

An index might make a SELECT-FROM query run slower due to several factors:
1. **Large Indexed Columns**: Large or complex data types in an index can lead to slower lookups.
2. **Low Selectivity**: Indexes on columns with many duplicate values might not reduce the search effort effectively, reducing their efficiency.
3. **Maintenance Overhead**: The overhead of updating indexes on data changes can indirectly slow down query performance.
4. **Inappropriate Index Usage**: If a query cannot effectively use an index, either due to its structure or the query planner's choice, it might introduce unnecessary overhead.
5. **Non-Covering Index**: If the index does not include all the columns needed by the query, additional data lookups can slow down execution.
6. **Resource Constraints**: High disk I/O and memory usage from index operations can degrade performance in resource-limited environments.


### Question 5

For the provided queries involving the `Users` table, the following indexes could enhance performance:
1. **Index for Join Operation**:
   - **Suggested Index**: `Users(id)`
   - **Reason**: An index on `Users.id` would accelerate the join operation with the `Assets` table where `Users.id` matches `Assets.uid`. This index would help quickly locate matching records in the `Users` table.

2. **Index for Filtering by Score**:
   - **Suggested Index**: `Users(score)`
   - **Reason**: An index on `Users.score` would facilitate rapid filtering of users with scores greater than 95. This would be particularly beneficial if score-based queries are common, as it avoids full table scans.

3. **Index for Filtering by Age**:
   - **Suggested Index**: `Users(age)`
   - **Reason**: An index on `Users.age` would make queries filtering users by age more efficient by quickly isolating records that meet the age condition.


### Question 6

#### Given Queries and Their Frequencies:
1. **Query joining `Users` and `Assets`**:
   - **Frequency**: 1000 executions per day.
   - **Query**: `SELECT * FROM Users, Assets WHERE Users.id = Assets.uid`
   - **Proposed Index**: An unclustered index on `Users.id` would generally suffice since `id` is a primary key and likely already indexed. However, considering the high frequency, ensuring this index is efficiently structured for joins could be beneficial.

2. **Query filtering by user score**:
   - **Frequency**: 1000 executions per day.
   - **Query**: `SELECT * FROM Users WHERE Users.score > 95`
   - **Proposed Index**: A clustered index on `score` could be advantageous for performance due to the high frequency and direct filtering on this attribute. However, as only one clustered index can exist and it's typically on the primary key, an unclustered index might be used instead unless there's a compelling reason to restructure the table around `score`.

3. **Query filtering by user age**:
   - **Frequency**: 10 executions per day.
   - **Query**: `SELECT * FROM Users WHERE Users.age > 21`
   - **Proposed Index**: Given the relatively low frequency, a basic unclustered index on `age` would improve performance without needing the heavier investment of a clustered index.

