# HW6

# Problem Set

**1.Bullshit**

**2. You have a table R containing 50M rows and wish to partition it into 2 <= k <= 32 shards.  You decide to partition on attribute A, whose range is documented as [-2^31, 2^31].**

Recall that in lecture, we defined skew as the existence of a partition Ri such that |Ri| >> |R|/k.  For the purposes of this question, we will add specificity to this definition by requiring |Ri| > 1.5 * |R|/k.

##### 2a
The correct choice is: **Choice 3: Hash Partition**

**Explanation:**
Hash partitioning involves applying a hash function to the attribute value (in this case, the value of A), which distributes the data evenly across the partitions regardless of the specific range of values A assumes. This approach minimizes skew because it does not rely on the values being evenly distributed within their range, as would be the case with range or block partitioning.

##### 2b
The correct choice is: **Choice 3: Hash Partition**
**Explanation:**
Hash partitioning is designed to distribute data uniformly across partitions by applying a hash function to the data values. Since powers of 2 are sparse and may cluster in specific ranges, range and block partitioning can lead to skewed partitions. Hash partitioning ensures an even distribution regardless of the specific values, effectively preventing skew.

##### 2c
The correct choice is: **Choice 4: None of the above**
**Explanation:**
When A only contains multiples of a given k, the distribution may be irregular depending on k's value. This pattern can lead to skew in both block and range partitions, as the data might cluster unevenly. Hash partitioning could also cause skew if k influences the distribution pattern in the hash function. Therefore, none of the partitioning schemes would guarantee an even distribution without skew.


##### 2d
The correct choice is: **Choice 4: None of the above**

**Explanation:**
When A only contains values that are power-of-2 multiples of k, these values will not be evenly distributed and can cluster in ways that lead to skew. Block and range partitioning could be significantly skewed due to the clustering patterns. Hash partitioning might also result in skew if the hash function doesn't adequately balance this specific pattern. Therefore, none of the partitioning schemes can guarantee an even distribution without skew in this scenario.

**3. Consider our relations Payroll(UserID, Name, Job, Salary) and Regist(UserID, Car, Year) and the following query:**
```sql
WITH OldestCar AS
     (  SELECT P1.Job AS Job, MIN(R1.Year) AS Year
          FROM Payroll AS P1, Regist AS R1
         WHERE P1.UserID = R1.UserID
      GROUP BY P1.Job)
SELECT P.Name, OC.Year
FROM Payroll AS P, 
     Regist AS R,
     OldestCar AS OC
WHERE P.UserID = R.UserID
  AND P.Job = OC.Job
  AND R.Year = OC.Year
  AND R.Year <= 2020
```
Use FJWGHOS to construct the unoptimized relational algebra tree; you do not need to submit this tree.  Next, optimize your tree by pushing the selection (σR.Year <= 2020) as far down as possible and possibly reordering the joins.  Submit your optimized tree.

##### Answer:
Here's a simple text representation of the optimized relational algebra tree without the additional formatting:

1. Initial Selection:
   - Apply the selection σ(Year ≤ 2020) (Regist) to filter rows based on the condition that Year is less than or equal to 2020.

2. Join Payroll and Filtered Regist:
   - Join Payroll with the filtered Regist on UserID.
   - Payroll ⋈UserID σ(Year ≤ 2020) (Regist)

3. Group By and Aggregation:
   - Perform a grouping on Job and compute the minimum Year for each job.
   - γ(Job, min(Year)) (Payroll ⋈UserID σ(Year ≤ 2020) (Regist))

4. Final Join:
   - Join Payroll, filtered Regist, and the grouped results from OldestCar on the matching Job and Year.
   - Payroll ⋈ Regist ⋈ (γ(Job, min(Year)) (Payroll ⋈UserID σ(Year ≤ 2020) (Regist)))

##### Final TREE attached. 



**4.Implement the previous question's query as a series of MapReduce tasks; place your selection (σR.Year <= 2020) in the location suggested by your optimized tree.**

Hints: Your third MapReduction will take the output of the previous two MRs as input.

###### Answer
To implement the query from the previous exercise as a series of MapReduce tasks, you will structure your operations based on the steps in the optimized relational algebra tree. The goal here is to distribute data processing using MapReduce patterns such as mapping, reducing, and possibly combining (intermediate reducing step).

### Q4.1 First Map 

```python
def map(key, value):
    # key: default input key from data source (not used)
    # value: a single record from either the Payroll or Regist relation

    # Split the input line into parts based on comma (assuming CSV format)
    fields = value.split(',')

    # Check if the record comes from the Regist table
    if "Regist" in fields:
        user_id = fields[0]  # Assuming UserID is the first field
        car = fields[1]      # Assuming Car is the second field
        year = int(fields[2])  # Assuming Year is the third field

        # Apply the selection operation
        if year <= 2020:
            # Emit UserID with a tuple including a tag 'Regist', Car, and Year
            print(f"{user_id}\t('Regist', '{car}', {year})")

    # Check if the record comes from the Payroll table
    elif "Payroll" in fields:
        user_id = fields[0]  # Assuming UserID is the first field
        name = fields[1]     # Assuming Name is the second field
        job = fields[2]      # Assuming Job is the third field
        salary = fields[3]   # Assuming Salary is the fourth field

        # Emit UserID with a tuple including a tag 'Payroll', Name, Job, and Salary
        print(f"{user_id}\t('Payroll', '{name}', '{job}', {salary})")
```

##### Explanation:
- **Splitting the Record**: The line is split into fields based on a comma, assuming the input data is in CSV format.
- **Deterministic Output**: Based on the origin (indicated here hypothetically by the presence of "Regist" or "Payroll" within the data), different fields are extracted.
- **Selection for Regist**: The selection condition is directly applied to filter out records from the `Regist` table where `Year` exceeds 2020.
- **Output Format**: The output is formatted to include the `UserID` as the key (for aggregation in the reduce step) and a tuple containing identifiers and relevant attributes as the value. The use of a tuple including tags ('Regist', 'Payroll') helps in distinguishing the origin of the records during the reduce phase.

This approach is directly compatible with frameworks like Hadoop Streaming, where inputs are typically read line-by-line and outputs are printed as tab-separated key-value pairs.


### Q4.2 First Reduction 

For the first reduce function in our MapReduce task, which deals with joining the filtered records from `Regist` and the records from `Payroll`, we can proceed with the following Python pseudocode:

```python
def reduce(user_id, values):
    # user_id: UserID, which serves as the key for the reduce function
    # values: List of tuples containing records from both Payroll and Regist tables

    # Separate lists to store Payroll and Regist records
    payroll_records = []
    regist_records = []

    # Iterate over values to separate them based on the origin tag
    for value in values:
        record_type = value[0]  # 'Payroll' or 'Regist'
        if record_type == 'Regist':
            regist_records.append(value)
        elif record_type == 'Payroll':
            payroll_records.append(value)

    # Join operation: Iterate over both list of records and join them
    for payroll in payroll_records:
        for regist in regist_records:
            # Output the combined record
            # Output format: User ID, Payroll Name, Payroll Job, Payroll Salary, Regist Car, Regist Year
            print(f"{user_id}\t{payroll[1]}, {payroll[2]}, {payroll[3]}, {regist[1]}, {regist[2]}")
```

##### Explanation:
- **Input and Output**: The `reduce` function takes a `user_id` and a list of `values` as input, where each value is a tuple representing either a `Payroll` or `Regist` record. The output is formatted to show combined data fields suitable for further processing or storage.
- **Data Separation**: The function first categorizes the records into two separate lists for `Payroll` and `Regist` based on the tags included in the tuples. This separation makes the subsequent join operation straightforward.
- **Join Logic**: The function performs a nested iteration over the two lists of records to simulate a join operation based on the common `UserID`. Each matched pair is then printed or stored, depending on the intended use of the output.

This `reduce` function effectively simulates an SQL join between two tables filtered and mapped by the previous `map` function, aligning with the requirements of the optimized query execution plan.


### Q4.3 Second Map
For the second map function in the MapReduce sequence, which takes the output from the first reduction (joined records) and prepares data for the aggregation task, here's the Python pseudocode:

```python
def map(key, value):
    # key: default input key (not used in this context)
    # value: a single line from joined_payroll_regist_output.txt containing joined data

    # Split the input line into parts
    fields = value.split('\t')

    # Extract relevant fields
    user_id = fields[0]  # Assuming UserID is the first field in the output
    job = fields[2]      # Assuming Job is the third field in the output
    year = int(fields[5])  # Assuming Year is the sixth field in the output

    # Emit Job and Year for aggregation
    print(f"{job}\t{year}")
```

##### Explanation:
- **Input and Output**: This `map` function processes each line (a value) from the file `joined_payroll_regist_output.txt`, which contains the joined records from `Payroll` and `Regist`. The function emits `Job` as the key and `Year` as the value for the subsequent aggregation step.
- **Splitting the Record**: The line is split based on tab delimiters, extracting the `Job` and `Year` fields, which are crucial for the aggregation operation.
- **Emission of Data**: The function emits the `Job` along with the corresponding `Year` to group all years associated with the same job in the next reduce step, where the minimum year will be calculated.

This map function is designed to extract and emit the necessary data for grouping and minimum year calculation, setting up for an efficient aggregation in the following reduce step.


### Q4.4 Second Reduction

The second reduce function is designed to handle aggregation, specifically to compute the minimum year for each job category based on the output of the second map function. Here's the Python pseudocode for this task:

```python
def reduce(job, years):
    # job: Job, which serves as the key for the reduce function
    # years: Iterator of years associated with the job

    # Initialize the minimum year to a high value
    min_year = float('inf')

    # Iterate through years to find the minimum
    for year in years:
        if year < min_year:
            min_year = year

    # Emit the job and its minimum year
    print(f"{job}\t{min_year}")
```

##### Explanation:
- **Input and Output**: The `reduce` function takes `job` as the key and an iterator of `years` as values. These years are the ones emitted by the second map function for each job.
- **Finding Minimum Year**: The function initializes `min_year` to infinity, which is a common practice to ensure any real year will be lower. It iterates through each year in the input and updates `min_year` if a smaller value is found.
- **Emission of Result**: After determining the minimum year for each job, the function emits the job along with its minimum year. This output can be used directly or stored for further processing.

This reduce function completes the task of finding the oldest car year associated with each job, aggregating data effectively in a distributed MapReduce environment.


###### If your pseudocode outputs a file (ie, bag of key/value pairs) that you will need in your next map, please provide its name here:
Since the output from the second reduce function contains the minimum year for each job, which is crucial for the final join operation, let's name this output file:

**`min_year_per_job_output.txt`**

This file will store the results of the aggregation, formatted as a bag of key/value pairs where each line contains a job and its minimum registered year, ready for use in the next phase of processing in the MapReduce sequence.


### Q4.5 Third Map

The third map function in the MapReduce sequence will prepare for the final join operation, using the output from the previous aggregation step and the initially joined records. This map function must emit tuples suitable for joining by key. Here's the Python pseudocode for this task:

```python
def map(key, value):
    # key: default input key (not used in this context)
    # value: a single line from either joined_payroll_regist_output.txt or min_year_per_job_output.txt

    # Determine the source of the data based on the contents
    fields = value.split('\t')

    if len(fields) == 2:
        # This record is from min_year_per_job_output.txt
        # Fields: [Job, MinYear]
        job = fields[0]
        min_year = fields[1]
        # Emit Job with the value containing a tag for distinction and MinYear
        print(f"{job}\t('min_year', {min_year})")
    else:
        # This record is from joined_payroll_regist_output.txt
        # Fields: [UserID, Name, Job, Salary, Car, Year]
        user_id = fields[0]
        name = fields[1]
        job = fields[2]
        salary = fields[3]
        car = fields[4]
        year = fields[5]
        # Emit Job with the value containing a tag for distinction and all necessary data for final output
        print(f"{job}\t('details', {user_id}, '{name}', {salary}, '{car}', {year})")
```

##### Explanation:
- **Input and Output**: This `map` function processes lines from two different files, `joined_payroll_regist_output.txt` for detailed employee and car data, and `min_year_per_job_output.txt` for aggregated data (minimum year per job). It needs to distinguish between these sources to handle them appropriately.
- **Data Handling**: Depending on the length of the split fields array, the function determines the source of the data. For entries from `min_year_per_job_output.txt` (which should have two fields: Job and MinYear), it emits the job and minimum year with a tag 'min_year'. For entries from `joined_payroll_regist_output.txt` (which have more details), it emits all necessary data tagged as 'details'.
- **Key for Join**: The key for emission is the `Job`, which is used to align all relevant data in the subsequent reduce phase for final joining.

This approach ensures that all necessary data for the final output is aligned by job in the reduce step, facilitating the last phase of the query execution.



### Q4.6 Third Reduction

The third reduce function is designed to perform the final join operation, using the outputs from the third map function. This step will combine the detailed payroll and registration data with the minimum year information for each job. Here's the Python pseudocode for this final reduce operation:

```python
def reduce(job, values):
    # job: Job, which serves as the key for the reduce function
    # values: List of tuples containing either minimum year data or detailed records

    min_year = None
    details = []

    # Separate the data based on the tags included in the tuples
    for value in values:
        if value[0] == 'min_year':
            min_year = int(value[1])
        elif value[0] == 'details':
            details.append(value)

    # Join and emit the final results
    if min_year is not None:
        for detail in details:
            # Unpack the details
            user_id, name, salary, car, year = detail[1], detail[2], detail[3], detail[4], detail[5]
            # Check if the year matches the minimum year for the job
            if int(year) == min_year:
                print(f"{job}\t{name}, {min_year}")
```

##### Explanation:
- **Input and Output**: The `reduce` function takes `job` as the key and a list of `values` as input. Each value is either a tuple with the minimum year or a tuple with detailed record data.
- **Data Separation**: First, it separates the values into two categories: minimum year and detailed data, based on the tag within each tuple.
- **Final Join and Emission**: For each detail that matches the minimum year (which means it's the oldest car for that job), the function emits the job along with the name and the minimum year. This ensures that only relevant records matching the oldest year per job are outputted.

This `reduce` function effectively completes the MapReduce pipeline by generating the final results specified in the original SQL query, ensuring each job is associated with the oldest car used by an employee in that job category.



### Q5 Comparing Optimizations
8 Points
Grading comment:
If you had implemented your unoptimized tree, in which part of your MapReduce pipeline would you have implemented the selection 
σ_R.Year<=2020	?

Choice 1 of 6: First map()
Choice 2 of 6: First reduce()
Choice 3 of 6: Second map()
Choice 4 of 6: Second reduce()
Choice 5 of 6: Third map()
Choice 6 of 6: Third reduce()

#### Answer:
If you were implementing the unoptimized tree for the MapReduce pipeline, the selection operation σ_R.Year <= 2020 would still ideally be applied as early as possible to reduce the amount of data processed in subsequent steps. In the context of the options provided:

**The correct answer is: Choice 1 of 6: First map()**

**Explanation:**
- Applying the selection σ_R.Year <= 2020 in the first map() function aligns with the principle of minimizing data early in the processing pipeline. This placement filters out records from the Regist table that do not meet the year criteria before any joins or other operations are performed, thus reducing the data volume that needs to be processed in later stages. This is essential whether optimizing or not, as it directly impacts the efficiency and performance of the entire MapReduce job.



