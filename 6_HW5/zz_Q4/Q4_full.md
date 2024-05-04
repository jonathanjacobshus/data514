
### Question 4 Homework 5 

To optimize the relational algebra tree by pushing the selection operators as low as possible, you want to apply selections as close to their relevant data sources as possible. This reduces the size of intermediate results and can significantly enhance query performance. Here's how you can adjust the tree:

### Original RA Tree:
```
Π(c.firstname, i.itemName, o.orderDate)
    |
    σ(c.lastname = 'Frumble')
    |
    ⨝ (o.cid = c.id)
       /                \
      /                  \
σ(i.quantity > 100)       Customers c
    |                     (T(Customers) = 50000, c.Id is PK)
    |
    ⨝ (o.id = i.oid)
       /          \
      /            \
  Orders o       OrderItems i
 (T(Orders) = 500000, o.id is PK)   (T(OrderItems) = 1000000, i.oid is FK into Orders)
```

### Modified RA Tree with Pushed Down Selections:
```
Π(c.firstname, i.itemName, o.orderDate)
    |
    ⨝ (o.cid = c.id)
       /                \
      /                  \
    ⨝ (o.id = i.oid)     σ(c.lastname = 'Frumble')(Customers c)
       /          \      (T(Customers) = 50000, c.Id is PK)
      /            \
  Orders o       σ(i.quantity > 100)(OrderItems i)
                 (T(OrderItems) = 1000000, i.oid is FK into Orders)
```

### Steps Taken:
1. **Selection of `i.quantity > 100` on OrderItems**: This selection is applied directly to the `OrderItems` table before any joins occur, reducing the number of tuples carried forward into the join with `Orders`.

2. **Selection of `c.lastname = 'Frumble'` on Customers**: This selection is applied directly to the `Customers` table before joining with the result from the `Orders` and `OrderItems` join. This limits the size of the join dataset by ensuring only relevant customer records participate in the join.

3. **Join Operations**: After applying the selections, the remaining joins are executed. This approach ensures that only relevant data from both `Customers` and `OrderItems` is processed in the joins, optimizing the query execution.

This optimized tree should be more efficient in terms of both computational and memory resources by minimizing the data volume early in the query processing stages.