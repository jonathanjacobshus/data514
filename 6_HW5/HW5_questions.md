# HW5

# Problem Set
#### The technical questions (i.e., all questions except the last three) refer to the following tables:

```sql
CREATE TABLE Customers (
    id           INTEGER PRIMARY KEY,
    firstname    VARCHAR(64) NOT NULL,
    lastname     VARCHAR(64) NOT NULL,
    city         VARCHAR(64) NOT NULL
);
CREATE TABLE Orders (
    id           INTEGER PRIMARY KEY,
    cid          INTEGER REFERENCES Customers,
    orderDate    DATE      -- assume this type is equivalent to
                           -- INTEGER; ie >, =, MAX(), etc are
                           -- well-defined
);
CREATE TABLE OrderItems (
    oid          INTEGER REFERENCES Orders,
    itemName     VARCHAR(64) NOT NULL,
    quantity     INTEGER,  -- guaranteed >0
    isFulfilled  INTEGER   -- boolean
);

```

**1. The following queries are semantically identical to the queries from section, though some names have changed.  Note that they use bind variables, which we have not covered in class; however, from an index-design perspective you can assume they behave as if they were a constant value (eg, "2024-05-09").**

This query is executed several times per second:**
```sql
-- Identify the items we need to put into a shipment (any shipment)
SELECT oi.itemName, SUM(oi.quantity)
FROM OrderItems oi
WHERE oi.isFulfilled = 0
GROUP BY oi.itemName;
```

--This query is executed once per hour:
```sql
-- Identify the lingering items ordered by the Bezos family which have
-- not been shipped yet, and the city to which they'd like their items to
-- be shipped.
SELECT oi.itemName, c.city, SUM(oi.quantity)
FROM Orders o, OrderItems oi, Customers c
WHERE o.id = oi.oid AND o.cid = c.id
AND oi.isFulfilled = 0
AND c.lastName = 'Bezos'
AND o.orderDate < ?
GROUP BY oi.itemName, c.city
ORDER BY o.orderDate, SUM(oi.quantity);
```

--These queries are executed approximately once every second:

```sql
-- Record a new order 
INSERT INTO Orders VALUES (?, ?, 0, ?);
-- Record their ordered items.  An "average" order has 3-5 items, but
-- it is possible to have a single-item order.  Orders are capped to 256
-- unique items, enforced by the database application.
INSERT INTO OrderItems VALUES (?, ?, ?);  
INSERT INTO OrderItems VALUES (?, ?, ?);  
INSERT INTO OrderItems VALUES (?, ?, 0, ?);  
-- ... etc ...
```

This query is executed many many times per second, but only during business hours:
```sql
-- Ship an item
UPDATE OrderItems
SET isFulfilled = 1
WHERE oid = ?
AND itemName = ?;

```
Lastly, approximately 10% of queries are none of these; they're a mixture of tools that aren't run frequently or adhoc reports run by analysts looking for interesting patterns (maybe brand loyalty to Bose earbuds?).  We don't want to optimize our database for these queries.

What indices would you create to support this query load?  Unlike the section worksheet, we would like you to consider the query load holistically rather than considering each query individually.  For example, if you decide to have more than one index on a table you must also describe how you chose which index to cluster.

You can assume the DBMS has not created any indices, not even an index on the tables' primary keys. 

##### Answer:
Certainly! Here are the responses formatted in markdown:

###### Q1.1 First Index
- **Table for Index:** OrderItems
- **Type of Index:** Unclustered
- **Indexed Attributes:** `isFulfilled`, `itemName`
- **Rationale:** Indexing `isFulfilled` and `itemName` on `OrderItems` enhances filtering for shipping and optimizes frequent item aggregation queries.

###### My Second Supplementary Index
- **Table for Index:** Orders
- **Type of Index:** Unclustered
- **Indexed Attributes:** `orderDate`, `cid`
- **Rationale:** These attributes were selected because `orderDate` and `cid` are commonly used in conditions to filter orders, especially in the hourly query that targets specific customers' older orders. An unclustered index on these attributes enables efficient filtering and sorting of orders by date without rearranging the base table, thus preserving quick insert and update operations.

###### Next: Q1.3
- **Table for Index:** N/A. I would only create two supplementary indices
- **Type of Index:** N/A
- **Indexed Attributes:** N/A
- **Rationale:** Given the primary query loads focusing on `OrderItems` and `Orders`, the two previously created indices are sufficient to optimize the most frequent and impactful operations. A third index may not provide additional performance benefits substantial enough to outweigh the cost of maintaining another index, especially considering the balance needed between read and write operations in a highly transactional system.


**2. Convert the following SQL query to a relational algebra tree.  If relevant, please:
	- Draw the tables from left to right in the same order in which they appear in the FROM clause
	- Order any joins in the same order in which they appear in the query**


```sql
SELECT o.cid
     FROM Orders o, OrderItems i
    WHERE o.id = i.oid
      AND (i.itemName = 'Bose QuietComfort Earbuds'
           OR i.itemName = 'Beats Fit Pro')
 GROUP BY o.cid
   HAVING COUNT(DISTINCT i.itemName) = 2;

```

##### Answer:
To convert the SQL query to a relational algebra tree, we first analyze the SQL that identifies customers who ordered both 'Bose QuietComfort Earbuds' and 'Beats Fit Pro' using `Orders` and `OrderItems`. The relational algebra involves:

1. **Selection** to filter `OrderItems` for the two specific products.
2. **Join** to link `Orders` with `OrderItems` where the order IDs match.
3. **Projection** to display the customer ID and item name.
4. **Grouping** to aggregate results by customer ID, counting distinct item names.
5. **Selection** again to ensure only customers who ordered both items are shown.

The relational algebra tree represents these operations sequentially from filtering and joining to aggregation and final selection, ensuring that only relevant data meeting all conditions is included.

**Diagram Attached In Another**  


**3. This and the next two problems require the following statistics:**

This and the next two problems require the following statistics:

T(Customers) = 50,000
T(Orders) = 500,000
T(OrderItems) = 1,000,000

V(Customers, lastname) = 5000
min(OrderItems, quantity) = 1
max(OrderItems, quantity) = 200


and refer to this RA tree:

Π<sub>(c.firstname,i.itemName,o.orderDate)</sub>)
                        |
                        |
        σ<sub>(c.lastname = 'Frumble' </sub>)     
                        |
                        |
                ⨝<sub> o.cid = c.id</sub>
                     /                  \
                    /                    \
    σ<sub>i.quantity > 100</sub>        Customers c
                |                        T(Customers) = 50k
                |                        c.Id is PK
    ⨝<sub> o.id = i.oid</sub>
           /      \
          /        \
    Orders o      OrderItems i
(Orders) = 500k    T(OrderItems) = 1M
  o.id is PK       i.oid is FK into Orders
 K into Customers


###### TREE IS ON GOOGLE DRIVE HQ5

 For each of the 5 stages, determine its selectivity factor.  Then, compute the cardinality of those stages; as a hint, the final projection produces 100 tuples.


To solve this problem, we will determine the selectivity factor of each operation in the relational algebra (RA) tree and then compute the cardinality of each stage. We need to analyze how many tuples each operation is likely to produce. The final projection produces 100 tuples, which will help to guide our computations backwards through the tree.

### Initial Data:

- **T(Customers) = 50,000**
- **T(Orders) = 500,000**
- **T(OrderItems) = 1,000,000**

- **V(Customers, lastname) = 5,000** (number of distinct values for `lastname`)

### Steps and Calculations:

1. **Selection σ (i.quantity > 100) on OrderItems:**
   - Since `min(OrderItems, quantity) = 1` and `max(OrderItems, quantity) = 200`, assuming a uniform distribution, the selectivity factor \( S \) for `quantity > 100` would be the proportion of quantities over 100 out of the total range of quantities.
   - Selectivity \( S = \frac{max - threshold}{max - min} \)
   - \( S = \frac{200 - 100}{200 - 1} = \frac{100}{199} \approx 0.5025 \)
   - Cardinality \( = S \times T(OrderItems) = 0.5025 \times 1,000,000 = 502,512 \)

2. **Join ⨝ (o.id = i.oid) on OrderItems and Orders:**
   - Since `o.id` is a primary key for `Orders` and `i.oid` is a foreign key referencing `Orders`, each `OrderItem` matches exactly one `Order`.
   - Selectivity \( S = 1 \) (perfect join condition, each item matches exactly one order)
   - Cardinality \( = T(OrderItems) = 1,000,000 \) (prior to selection filter)

3. **Selection σ (c.lastname = 'Frumble') on Customers:**
   - Assuming uniform distribution of lastnames:
   - Selectivity \( S = \frac{1}{V(Customers, lastname)} = \frac{1}{5,000} = 0.0002 \)
   - Cardinality \( = S \times T(Customers) = 0.0002 \times 50,000 = 10 \)

4. **Join ⨝ (o.cid = c.id) between filtered Customers and Orders (joined with OrderItems):**
   - This join depends on how many customers with `lastname = 'Frumble'` have placed orders.
   - Assume this join keeps the selectivity \( S = 1 \) (every customer found has orders)
   - Cardinality \( = T(filtered Customers) = 10 \) (from previous selection on Customers)

5. **Projection Π (c.firstname, i.itemName, o.orderDate):**
   - This projection reduces the tuples to 100, as stated in the hint.
   - Final cardinality \( = 100 \)

### Conclusion:
The operations and their cardinalities are estimated based on selectivity and initial table sizes. These calculations are guided by assumptions on data distribution and constraints provided (like primary and foreign keys). This approach enables estimation of output sizes at each stage of the RA tree.

#### Gradescope Questions

###### Q3.1 Join between Orders and Order Items (o.id=i.oid)
4 Points
**Selectivity Denominator:** 
For the join between Orders and OrderItems where o.id = i.oid, the selectivity denominator n is 1, because each item in OrderItems matches exactly one order in Orders due to the primary-foreign key relationship.

**Output Cardinality:**
 The output cardinality for this join is 1000000, reflecting the total number of tuples in OrderItems, since each tuple finds a match in Orders.

###### Q3.2 Selection on join result (i.quantity > 100)
4 Points
**Selectivity Denominator:**
For the selection i.quantity > 100 on the join result of Orders and OrderItems, the selectivity denominator is 199, assuming a uniform distribution of quantity values from 1 to 200.

**Output Cardinality:** 
The output cardinality for this selection is 502512. This is calculated by applying the selectivity factor 100/199 to the total 1000000 tuples from the join result between Orders and OrderItems.

 ###### "Q3.3 Selection on Customers (c.lastname='Frumble')
4 Points"

**Selectivity Denominator:** 
The selectivity denominator for the selection c.lastname = 'Frumble' on the Customers table is 5000, as there are 5000 distinct last names in the Customers table.

**Output Cardinality:** 
The output cardinality for this selection is 10. This is calculated by applying the selectivity factor 1/5000 to the total 50000 tuples in the Customers table.

###### Q3.4 Join between Orders and Customers (o.cid=c.id)
4 Points
**Selectivity Denominator:** 
The selectivity denominator for the join between Orders and Customers where o.cid = c.id is 10, based on the ratio of total orders (500000) to unique customers (50000).

**Cardinality:** 

##### Q3.5 Final Projection (c.firstname,i.itemName, o.orderDate)
2 Points
**Selectivity Denominator:** 
The final projection involving c.firstname, i.itemName, and o.orderDate does not inherently alter selectivity, so the denominator is considered 1, indicating that each tuple passes through unchanged.

**Output Cardinality:** 
The output cardinality for the final projection is 100, as specified in the problem setup, indicating the total number of tuples resulting from all prior selections and joins.

-- 



**4. Starting from the same tree as question 3, push the selection operators down as low as possible.  Submit your modified tree to Gradescope.**





**5. For each of the 5 stages in the tree you modified submitted for question 4, determine its selectivity factor and cardinality.  Then, compute the cardinality of those stages; the final projection will still produce 100 tuples.**


**6. Consider the trees (and their cardinality estimates) from question 2 and question 4.  Which tree executes more efficiently?  Why?**


**7. Please read the following two sections:
1. How we rank Feed and Stories
2. How you can influence what you see

of this article describing Facebook's algorithm for ranking Instagram posts.  Then, describe the scale, impact, and opacity of this algorithm.  You should be able to answer each section in a maximum of 3-5 sentences, and you may find this example helpful.** 


**8. Next, please reflect on your own personal experience.  Specifically:
1. What is one thing that you learned while doing this assignment?
2. What is one thing that surprised you while doing this assignment?


**9.Lastly, please answer the feedback questions:** 
1. How many hours did it take you to finish this assignment, including time to set up your computer (if necessary)?
2. How many of those hours did you feel were valuable and/or contributed to your learning?
3. Did you collaborate with other students on this assignment?  If so, approximately how many people did you collaborate with?  Do not include yourself or course staff in the count.
















