# DATA 514 - Final Project - Q1 + Q3


### Q1 Project Overview
###### Original 
This project involves designing a database schema and writing queries for the BDD100K dataset, a large-scale diverse driving video database. Our focus will be on a subset of the data to create a manageable project scope.

###### New

### Project Overview
This project focuses on designing a comprehensive database schema for the BDD100K dataset, an extensive and varied driving video database created to support computer vision research for autonomous driving. The dataset includes 100,000 video clips, each 40 seconds long, recorded at 720p and 30 fps, capturing diverse driving scenarios across different locations, weather conditions, and times of day in the United States.

Our aim is to create a manageable subset of this dataset to facilitate efficient data access and use. This will involve constructing entities for videos, annotations, objects, and trajectories, ensuring well-defined relationships and optimized performance for future analysis.

### Objectives:
1. **Database Schema Design**: Develop a robust schema that accurately represents the diverse data within the BDD100K dataset, ensuring well-defined and optimized relationships between entities such as videos, annotations, objects, and trajectories.
   
2. **Data Governance**: Identify and address key data governance issues, including managing sensitive information like GPS data and video footage, and propose measures to protect this information and ensure compliance with privacy laws.

3. **Query Development**: Write and execute SQL queries to demonstrate practical applications of the database schema, covering use cases such as retrieving metadata, counting object categories, and listing specific annotations.

4. **Query Optimization and Indexing**: Analyze and optimize the logical plans of our queries to improve performance, and identify appropriate indexes to support use cases and enhance query efficiency.

5. **DBMS Selection**: Recommend a suitable DBMS for managing the BDD100K dataset, considering performance, scalability, and feature support.

By achieving these objectives, we aim to develop a schema that enhances future users' ability to access and analyze the BDD100K data efficiently.



### Q3 Data Governance Issues

###### Original 
- **Sensitive Information**: GPS data and video footage.
- **Protection Measures**: Anonymization of personal data, secure storage, and restricted access.
- **Future Concerns**: Compliance with data privacy laws and ethical use of autonomous driving data.


###### New

### Data Governance Issues

#### Sensitive Information
The BDD100K dataset contains various types of sensitive information, including GPS data, video footage, and personal identifiers. These data points could potentially be used to identify individuals or specific locations, posing significant privacy risks.

#### Protection Measures
To mitigate these risks, several protection measures should be implemented:
1. **Anonymization**: Personal identifiers and location details should be anonymized to prevent identification.
2. **Secure Storage**: Data should be stored in secure, encrypted databases with restricted access to authorized personnel only.
3. **Access Controls**: Implement role-based access controls to ensure that only individuals with the necessary permissions can access sensitive information.

#### Compliance with Data Privacy Laws
Compliance with data privacy laws, such as the General Data Protection Regulation (GDPR) and the California Consumer Privacy Act (CCPA), is essential. These laws mandate specific measures for data protection, user consent, and the right to data deletion. Ensuring that the dataset complies with these regulations is crucial to avoid legal repercussions.

#### Ethical Use of Data
Ethical considerations must be addressed to ensure the data is used responsibly:
1. **Informed Consent**: Data should be collected with informed consent from individuals captured in the videos, where feasible.
2. **Usage Limitations**: Clearly define and limit the scope of data usage to research and development of autonomous driving technologies.
3. **Transparency**: Maintain transparency about how the data is used, shared, and stored, providing clear information to stakeholders and the public.

#### Future Concerns
Future reuse of the dataset may raise additional concerns:
1. **Data Retention**: Establish clear policies on data retention, specifying how long data will be kept and under what conditions it will be deleted.
2. **Data Sharing**: Regulate data sharing practices to ensure that third parties adhere to the same data protection standards.
3. **Continuous Monitoring**: Implement continuous monitoring and auditing processes to ensure ongoing compliance with data protection standards and regulations.

#### Data Quality and Integrity
Maintaining high data quality and integrity is critical:
1. **Validation**: Regularly validate and update the data to ensure accuracy and completeness.
2. **Error Handling**: Establish protocols for identifying and correcting data errors or inconsistencies.

By addressing these data governance issues, we can ensure the responsible and ethical use of the BDD100K dataset, protecting individual privacy while supporting valuable research in autonomous driving.



### Query Logical Plan Analysis 6

###### Original
- **Naive RA Tree**: [Insert RA Tree for one of the queries]
- **Optimized RA Tree**: [Insert optimized RA Tree]
- **Performance Differences**: The optimized RA tree reduces the number of joins and filters data earlier in the process, leading to improved performance.


###### New

### Query Logical Plan Analysis

#### Use Case 1: Counting the Number of Objects for Each Frame in a Given Video

**Query**:
```sql
SELECT f.frame_number, COUNT(o.object_id) AS object_count
FROM Frame f
LEFT JOIN Object o ON f.frame_id = o.frame_id
LEFT JOIN Video v ON f.video_id = v.video_id
WHERE v.title = 'NYC June 1 2019'
GROUP BY f.frame_number;
```

**Naive RA Tree**:
1. **Selection**: Filter `Video` by `title = 'NYC June 1 2019'`
2. **Join**: Merge `Frame` with filtered `Video` on `video_id`
3. **Join**: Merge `Frame` with `Object` on `frame_id`
4. **Projection**: Extract `frame_number` and `object_id`
5. **Aggregation**: Count `object_id` grouped by `frame_number`

**Optimized RA Tree**:
1. **Selection**: Filter `Video` by `title = 'NYC June 1 2019'`
2. **Join**: Merge `Frame` with filtered `Video` on `video_id`
3. **Projection**: Extract `frame_id` and `frame_number` from `Frame`
4. **Join**: Merge `Frame` with `Object` on `frame_id`
5. **Aggregation**: Count `object_id` grouped by `frame_number`

**Performance Differences**:
Performing the initial selection and join early reduces the number of rows processed in subsequent steps, enhancing efficiency.

---

#### Use Case 2: Find the Number of Frames with a Crosswalk per Video

**Query**:
```sql
SELECT v.video_id, COUNT(l.lane_id)
FROM Frame f
LEFT JOIN Video v ON f.video_id = v.video_id
LEFT JOIN LaneMarking l ON f.frame_id = l.frame_id
WHERE l.lane_category = 1
GROUP BY v.video_id;
```

**Naive RA Tree**:
1. **Selection**: Filter `LaneMarking` by `lane_category = 1`
2. **Join**: Merge `LaneMarking` with `Frame` on `frame_id`
3. **Join**: Merge `Frame` with `Video` on `video_id`
4. **Projection**: Extract `video_id` and `lane_id`
5. **Aggregation**: Count `lane_id` grouped by `video_id`

**Optimized RA Tree**:
1. **Selection**: Filter `LaneMarking` by `lane_category = 1`
2. **Join**: Merge `LaneMarking` with `Frame` on `frame_id`
3. **Projection**: Extract `frame_id` from `Frame`
4. **Join**: Merge `Frame` with `Video` on `video_id`
5. **Aggregation**: Count `lane_id` grouped by `video_id`

**Performance Differences**:
Early selection and join steps reduce intermediate result sizes, improving query performance.

---

#### Use Case 3: Finding the Number of Frames Available for Every Time of Day and Weather Combination

**Query**:
```sql
SELECT f.weather, f.timeofday, COUNT(*)
FROM Frame f
LEFT JOIN Video v ON f.video_id = v.video_id
GROUP BY f.weather, f.timeofday;
```

**Naive RA Tree**:
1. **Projection**: Extract `weather`, `timeofday`, and `frame_id` from `Frame`
2. **Aggregation**: Count `frame_id` grouped by `weather` and `timeofday`

**Optimized RA Tree**:
1. **Projection**: Extract `weather`, `timeofday`, and `frame_id` from `Frame`
2. **Aggregation**: Count `frame_id` grouped by `weather` and `timeofday`

**Performance Differences**:
Both plans are similar, but ensuring the use of indexes on `weather` and `timeofday` can enhance performance.

### Indexes
- **Weather and Time of Day Index**: Index on `weather` and `timeofday` columns in the `Frame` table to support the third query.
- **Frame ID Index**: Index on `frame_id` column in the `Object` and `LaneMarking` tables to support the first and second queries.

By incorporating these queries and their logical plans into the analysis, we can further ensure efficient data retrieval and processing within the database schema.

