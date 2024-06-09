### ER Diagram Design for Relational Database (Flightapp)
#### Logical Entities and Relationships
1. **Flights**: Stores information about individual flights.
2. **Carriers**: Stores information about flight carriers.
3. **Months**: Stores information about months.
4. **Weekdays**: Stores information about weekdays.
5. **Users**: Stores user information including username, password, and account balance.
6. **Itineraries**: Represents one or more flights connecting an origin city to a destination city.
7. **Reservations**: Represents a user's reservation, consisting of one or more flights.
#### Operations Supported
- `create(username, balance, password)`
- `search(origin, dest, day_of_month, directonly)`
- `reserve(fid1, …, fidn)`
- `pay(reservation_id)`
- `list_reservations(user_id)`
#### Tables and Columns
- **Flights**
  - `flight_id` (PK)
  - `carrier_id` (FK)
  - `origin`
  - `destination`
  - `departure_time`
  - `arrival_time`
  - `capacity`
  - `reservable_capacity` (calculated dynamically based on reservations)
- **Carriers**
  - `carrier_id` (PK)
  - `carrier_name`
- **Months**
  - `month_id` (PK)
  - `month_name`
- **Weekdays**
  - `weekday_id` (PK)
  - `weekday_name`
- **Users**
  - `user_id` (PK)
  - `username` (unique)
  - `password`
  - `balance`
- **Itineraries**
  - `itinerary_id` (PK)
  - `origin`
  - `destination`
  - `is_direct` (boolean)
- **Itinerary_Flights** (link table for many-to-many relationship between Itineraries and Flights)
  - `itinerary_id` (FK)
  - `flight_id` (FK)
- **Reservations**
  - `reservation_id` (PK)
  - `user_id` (FK)
  - `is_paid` (boolean)
  - `reservation_date`
- **Reservation_Flights** (link table for many-to-many relationship between Reservations and Flights)
  - `reservation_id` (FK)
  - `flight_id` (FK)
#### Dynamic Calculation of Reservable Capacity
- Reservable capacity is calculated as `capacity - COUNT(reservation_id)` where `flight_id` is the same in the `Reservation_Flights` table.