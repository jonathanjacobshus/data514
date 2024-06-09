# Takeaway Sheet 9 - Jonathan Jacobs

### Question 1
Can you have a ______ as your value?"
- **JSON document**: Yes
- **Hash table**: Yes
- **Hash table of hash tables**: Yes

### Question 2
To look up the origin city in a wide column database, I need to use the following identifiers in order:

1. **Row Key**: This uniquely identifies the specific row that contains the data for the flight.
2. **Column Family**: This logical grouping within the row organizes related columns together.
3. **Column**: This specific field within the column family holds the value of the origin city.

The physical entities that each of the identifiers correspond to are:
1. **Row Key**: This corresponds to a specific row in the database, which is stored as an entry in the underlying storage system.
2. **Column Family**: This corresponds to a physical file or segment within the database file that groups related columns together. Each column family can have its own physical file for optimized storage and retrieval.
3. **Column**: Within the column family file, this corresponds to a specific field or key-value pair that holds the actual data, such as the origin city value.


### Question 3
For the application with two types of lookups, I would use one row with the following row key:

##### Row Key
**Row Key**: `license_plate`

##### Example Structure
```plaintext
Row Key: license_plate_ABC123
    Column Family: tickets
        Column: ticket_count -> "5"
    Column Family: permits
        Column: permit_lot -> "C15"
```

This structure ensures efficient access to both types of data related to the same license plate, using separate column families for tickets and permits.





