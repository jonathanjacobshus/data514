Q2 Partitioning and Skew
24 Points
You have a table R
R containing 50 million rows and wish to partition it into 2≤k≤32
2≤k≤32 shards. You decide to partition on an attribute A
A, whose range is documented as [−2^31,2^31] 

In lecture, we defined skew as the existence of a partition R_i s.t. 
∣R_i∣ ≫ ∣R∣/k.  For the purposes of this question, we will add specificity to this definition by requiring 
∣R_i∣ > 1.5 ∗ ∣R∣ /k



First question: 

"Q2.1
6 Points
Grading comment:
Although A's range is declared as [−2^31,2^31]
 in actuality A only takes on the values from 
[0,2^31]
Select all the partitioning schemes which would not yield skew for all possible values of k:

Choice 1 of 4: Block Partition
Choice 2 of 4: Range Partition
Choice 3 of 4: Hash Partition
Choice 4 of 4: None of the above
"


Second question:

"
Q2.2
6 Points
Grading comment:
In actuality, A only takes on the values that are powers of 2 and also in the range [−2^2,2^31].  Thus, A only consists of  {-2^2,-2^1,-2^0,-2^0,2^1,2^1,...2^31 }

Select all the partitioning schemes which would not yield skew for all possible values of k:

Choice 1 of 4: Block Partition
Choice 2 of 4: Range Partition
Choice 3 of 4: Hash Partition
Choice 4 of 4: None of the above
"



Third Question 

"
Q2.3
6 Points
Grading comment:
In actuality, A only takes on the values that are multiples of k in its declared range.  Thus, A contains values such as {-2k, 173k, -216k, etc}. 

Select all the partitioning schemes which would not yield skew for all possible values of k:

Choice 1 of 4: Block Partition
Choice 2 of 4: Range Partition
Choice 3 of 4: Hash Partition
Choice 4 of 4: None of the above"

"



"Q2.4
6 Points
Grading comment:
In actuality A only takes on the values that are power-of-2 multiples of k.  Thus, A only consists of {
−2^(31−log k)k,..., −(2^1)k, (2^0)k, (2^1)k, ... 2^(31−log k)k}
Select all the partitioning schemes which would not yield skew for all possible values of k:

Choice 1 of 4: Block Partition
Choice 2 of 4: Range Partition
Choice 3 of 4: Hash Partition
Choice 4 of 4: None of the above"
