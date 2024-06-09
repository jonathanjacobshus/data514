# DATA 514 Section 8 Worksheet - Jonathan Jacobs

DATA 514 Section 8 Worksheet
This doc outlines the running Payroll/Regist example, as well as how it should be implemented in the various NoSQL options we’ll be discussing. We will add a new table to the running example, ParkingTickets.
As a reminder, these are the schema and example values. Note that Frances doesn’t have a car; Magda has two, and nobody employed at UW owns the Aston Martin.
Payroll

### Tables
Payroll
UserID 	Name	Job 	ParkingPermit
123 	Leslie	TA 		C15
345		Frances	TA 		NULL
567 	Magda  	Prof 	E18
789 	Quinn 	Prof 	NULL


Regist
UserID	Car				LicensePlate
123 	Charger			123 AAA
567 	Civic			MMM 1234
567 	Ferrari			MMM 5678
789		Picklemobile	PIK 1024
007 	Aston Martin	XYZ 0007


ParkingTickets
LicensePlate	ParkingLot	Date		Amount
MMM 1234		C15			2022-11-20	$10
MMM 1234 		E01			2022-11-21	$15
PIK 1024		E18			2022-11-22	$10
XYZ 0007		C19			2022-11-01	$10
MMM 1234		E01			2022-11-22	$20

**Application / Use-Cases** 
This is a parking enforcement app which supports the following methods:
	1. [infrequent] Listing the permitted parking lot and per-car tickets incurred by each user
		o method sig is uid -> [ {car1, [{tix1}, {tix2}]}, {car2, []} ]
	2. [frequent] Counting how many tickets a license plate has ever had
		o method sig is plate -> int or it can be plate -> [ {tix1}, {tix2}]
		o We use the output of this method to determine the citation amount.
	3. [multiple times / sec] Determining whether a plate is allowed to be a specific lot
		o method sig is (plate, lot) -> true/false or it can be plate -> lot
Because NoSQL design is so intimately tied to its use-cases, there are three decisions which will need to be made for each system:
	● Method 3: should we store the license plate and permitted lot as the key, with the value being true and the expectation that values which don’t exist should default to false?
	● Method 2: should we store a count of the tickets, or the actual tickets themselves? Note that the latter introduces the possibility of data anomalies (which may be deemed acceptable)
	● How should we handle cars without owners (the Aston Martin) or employees without cars (Frances)? Is data loss acceptable or not?

### Summary of Application and Use Cases

This is a parking enforcement application designed to manage parking permits and parking tickets for users. The app supports the following methods:

1. **Listing Permitted Parking Lot and Per-Car Tickets Incurred by Each User**:
   - Method Signature: `uid -> [ {car1, [{tix1}, {tix2}]}, {car2, []} ]`
   - This method is used infrequently to list each user's permitted parking lot and the tickets incurred by each of their cars.

2. **Counting How Many Tickets a License Plate Has Ever Had**:
   - Method Signature: `plate -> int` or `plate -> [ {tix1}, {tix2} ]`
   - This method is used frequently to count the total number of tickets a specific license plate has received. This count is used to determine the citation amount.

3. **Determining Whether a Plate is Allowed to Be in a Specific Lot**:
   - Method Signature: `(plate, lot) -> true/false` or `plate -> lot`
   - This method is executed multiple times per second to check if a specific license plate is permitted to park in a specific lot.

### Key Decisions for NoSQL System Design

1. **Method 3: Storage of License Plate and Permitted Lot**:
   - **Option**: Store the license plate and permitted lot as the key, with the value being `true`, and assume non-existent values default to `false`.
   - **Considerations**: This approach allows for quick lookup to determine if a plate is allowed in a lot, which is critical given the high frequency of this method.

2. **Method 2: Storage of Ticket Counts vs. Actual Tickets**:
   - **Option**: Store either a count of the tickets or the actual ticket records.
   - **Considerations**: Storing a count is efficient and avoids data anomalies but provides less detailed information. Storing actual tickets allows for detailed queries at the expense of potential data anomalies and increased storage requirements.

3. **Handling Cars Without Owners and Employees Without Cars**:
   - **Option**: Determine whether data loss is acceptable for cars without owners (e.g., the Aston Martin) and employees without cars (e.g., Frances).
   - **Considerations**: If data loss is not acceptable, the design must account for these cases, potentially complicating the schema. If data loss is acceptable, the schema can be simplified.



## Question 1: Key-Value Data Store
Describe how you would implement the Payroll/Regist application using a key-value data store. Your description should contain enough detail that we understand the key (or keys if you have multiple “types” of keys), any structure that you introduce into the key, and the structure of a key’s value.


### Answer:

### Implementation Using a Key-Value Data Store
A key-value data store is a type of NoSQL database where each item is stored as a key-value pair. For the Payroll/Regist application, we need to design keys and values to support the required methods efficiently. Below is a detailed implementation:
#### Key Structures
1. **User Data (Payroll and Regist)**
   - **Key**: `user:{UserID}`
   - **Value**: A JSON object containing user information and their registered cars.

   ```json
   {
       "name": "Leslie",
       "job": "TA",
       "parkingPermit": "C15",
       "cars": [
           {"car": "Charger", "licensePlate": "123 AAA"}
       ]
   }
   ```
2. **Car Data (ParkingTickets)**
   - **Key**: `car:{LicensePlate}`
   - **Value**: A JSON array of parking tickets for the car.
   ```json
   [
       {"parkingLot": "C15", "date": "2022-11-20", "amount": "$10"},
       {"parkingLot": "E01", "date": "2022-11-21", "amount": "$15"}
   ]
   ```
3. **Parking Lot Permissions**
   - **Key**: `permit:{ParkingPermit}:{LicensePlate}`
   - **Value**: `true`
   - **Explanation**: This structure allows quick checking of whether a specific license plate is permitted to park in a specific lot. Non-existent keys will imply `false`.
#### Detailed Breakdown of Keys and Values
1. **User Data Key-Value Pairs**
   - **Key**: `user:{UserID}`
   - **Value**: A JSON object with fields `name`, `job`, `parkingPermit`, and `cars`. The `cars` field is an array of car objects, each containing `car` and `licensePlate`.
   Example:
   ```json
   "user:123": {
       "name": "Leslie",
       "job": "TA",
       "parkingPermit": "C15",
       "cars": [
           {"car": "Charger", "licensePlate": "123 AAA"}
       ]
   }
   ```
2. **Car Data Key-Value Pairs**
   - **Key**: `car:{LicensePlate}`
   - **Value**: A JSON array where each element is a parking ticket object with fields `parkingLot`, `date`, and `amount`.

   Example:
   ```json
   "car:MMM 1234": [
       {"parkingLot": "C15", "date": "2022-11-20", "amount": "$10"},
       {"parkingLot": "E01", "date": "2022-11-21", "amount": "$15"},
       {"parkingLot": "E01", "date": "2022-11-22", "amount": "$20"}
   ]
   ```

3. **Parking Lot Permissions Key-Value Pairs**
   - **Key**: `permit:{ParkingPermit}:{LicensePlate}`
   - **Value**: `true`

   Example:
   ```json
   "permit:C15:123 AAA": true
   ```
#### Method Implementations
1. **Listing the Permitted Parking Lot and Per-Car Tickets Incurred by Each User**
   - **Method Signature**: `uid -> [ {car1, [{tix1}, {tix2}]}, {car2, []} ]`
   - **Implementation**:
     - Fetch user data using `user:{UserID}`.
     - For each car in the user's `cars` array, fetch ticket data using `car:{LicensePlate}`.

2. **Counting How Many Tickets a License Plate Has Ever Had**
   - **Method Signature**: `plate -> int` or `plate -> [ {tix1}, {tix2} ]`
   - **Implementation**:
     - Fetch car data using `car:{LicensePlate}`.
     - Return the length of the array for the count or return the array itself for detailed tickets.

3. **Determining Whether a Plate is Allowed to Be in a Specific Lot**
   - **Method Signature**: `(plate, lot) -> true/false`
   - **Implementation**:
     - Check the existence of the key `permit:{ParkingPermit}:{LicensePlate}`.
     - If the key exists, return `true`; otherwise, return `false`.
### Summary
This key-value data store implementation efficiently supports the application’s methods by structuring keys and values to allow quick lookups and minimal data duplication. Each key represents a specific piece of data, and values are structured to provide all necessary details, enabling the application to perform the required operations with minimal overhead.


## Question 2: Document Store
Describe how you would implement the Payroll/Regist application using a document store database. Your description should contain enough detail that we understand all the types in your document.

### Answer:

### Implementation Using a Document Store Database
A document store database is a type of NoSQL database designed to store, retrieve, and manage document-oriented information. Documents are typically represented in JSON or BSON format. For the Payroll/Regist application, we will design the documents to efficiently support the required methods.
#### Document Structures
1. **User Document**
   - Contains user information, including registered cars and parking permit details.
   - Each user document includes an array of car documents.

   ```json
   {
       "userID": 123,
       "name": "Leslie",
       "job": "TA",
       "parkingPermit": "C15",
       "cars": [
           {
               "car": "Charger",
               "licensePlate": "123 AAA",
               "tickets": [
                   {"parkingLot": "C15", "date": "2022-11-20", "amount": "$10"}
               ]
           }
       ]
   }
   ```
2. **Car Document (Separate Collection)**
   - Contains details about a car, including its license plate and any associated parking tickets.
   - This document is primarily used for querying ticket counts and validating parking lot permissions.
   
   ```json
   {
       "licensePlate": "MMM 1234",
       "ownerUserID": 567,
       "tickets": [
           {"parkingLot": "C15", "date": "2022-11-20", "amount": "$10"},
           {"parkingLot": "E01", "date": "2022-11-21", "amount": "$15"},
           {"parkingLot": "E01", "date": "2022-11-22", "amount": "$20"}
       ]
   }
   ```
#### Collections
1. **Users Collection**
   - Each document in this collection represents a user and includes their details along with their cars and any tickets associated with those cars.
   Example Document:
   ```json
   {
       "_id": 123,
       "name": "Leslie",
       "job": "TA",
       "parkingPermit": "C15",
       "cars": [
           {
               "car": "Charger",
               "licensePlate": "123 AAA",
               "tickets": [
                   {"parkingLot": "C15", "date": "2022-11-20", "amount": "$10"}
               ]
           }
       ]
   }
   ```

2. **Cars Collection**
   - Each document in this collection represents a car and includes its license plate, owner (userID), and any tickets associated with the car.
   Example Document:
   ```json
   {
       "_id": "MMM 1234",
       "ownerUserID": 567,
       "tickets": [
           {"parkingLot": "C15", "date": "2022-11-20", "amount": "$10"},
           {"parkingLot": "E01", "date": "2022-11-21", "amount": "$15"},
           {"parkingLot": "E01", "date": "2022-11-22", "amount": "$20"}
       ]
   }
   ```

#### Method Implementations
1. **Listing the Permitted Parking Lot and Per-Car Tickets Incurred by Each User**
   - **Method Signature**: `uid -> [ {car1, [{tix1}, {tix2}]}, {car2, []} ]`
   - **Implementation**:
     - Query the `Users` collection for the document with `userID`.
     - Extract the `cars` array from the user document.
     - Each car in the `cars` array includes a `tickets` array with the incurred tickets.
2. **Counting How Many Tickets a License Plate Has Ever Had**
   - **Method Signature**: `plate -> int` or `plate -> [ {tix1}, {tix2} ]`
   - **Implementation**:
     - Query the `Cars` collection for the document with `_id` as `LicensePlate`.
     - Return the length of the `tickets` array for the count or return the `tickets` array itself for detailed tickets.

3. **Determining Whether a Plate is Allowed to Be in a Specific Lot**
   - **Method Signature**: `(plate, lot) -> true/false`
   - **Implementation**:
     - Query the `Cars` collection for the document with `_id` as `LicensePlate`.
     - Retrieve the `ownerUserID` from the car document.
     - Query the `Users` collection for the document with `userID` as `ownerUserID`.
     - Check if the `parkingPermit` matches the given lot.

### Summary
Using a document store database like MongoDB, the implementation of the Payroll/Regist application involves creating two main collections: `Users` and `Cars`. Each user document contains personal details, parking permit information, and an array of car documents, which include ticket details. Each car document in the `Cars` collection contains the license plate, owner information, and tickets. This structure ensures efficient querying for all use cases, leveraging the strengths of document-oriented databases.










	
