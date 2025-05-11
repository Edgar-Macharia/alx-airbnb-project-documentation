# Technical and Functional Requirement Specifications for Airbnb Clone Backend

## 1. User Authentication

### Functional Requirements

* **User Registration** : Allow users to sign up as guests or hosts, providing details such as email, password, first name, last name, phone number, and role. Support OAuth registration (e.g., Google, Facebook) for convenience.
* **User Login** : Enable users to log in using email and password or OAuth, providing secure access to platform features.
* **Profile Management** : Allow users to update their profile information, including profile photos, contact details, and preferences, to maintain accurate user data.

### Technical Requirements

* **API Endpoints** :
* **POST /api/auth/register** :

  * **Description** : Register a new user.
  * **Input** : JSON body:

  ```json
  {
    "email": "string",
    "password": "string",
    "first_name": "string",
    "last_name": "string",
    "phone_number": "string",
    "role": "string (guest, host, admin)"
  }
  ```

  ***Output** : JSON response (HTTP 201 Created):
  ``json       {         "user_id": "integer",         "email": "string",         "first_name": "string",         "last_name": "string",         "phone_number": "string",         "role": "string",         "token": "string (JWT)"       }       ``

  * **Validation Rules** :
    * Email must be unique and follow a valid format (e.g., [user@example.com](mailto:user@example.com)).
    * Password must be at least 8 characters, including uppercase, lowercase, number, and special character.
    * Role must be one of: guest, host, admin.
    * First name, last name, and phone number are required.
  * **Error Handling** :
    * HTTP 400 Bad Request: Invalid input (e.g., duplicate email, weak password).
    * HTTP 422 Unprocessable Entity: Missing required fields.
* **POST /api/auth/login** :

  * **Description** : Authenticate a user and return a JWT token.
  * **Input** : JSON body:

  ```json
  {
    "email": "string",
    "password": "string"
  }
  ```

  ***Output** : JSON response (HTTP 200 OK):
  ``json       {         "user_id": "integer",         "email": "string",         "role": "string",         "token": "string (JWT)"       }       ``

  * **Validation Rules** :
    * Email and password must match an existing user record.
    * Handle OAuth login by validating tokens from providers like Google.
  * **Error Handling** :
    * HTTP 401 Unauthorized: Invalid credentials.
    * HTTP 400 Bad Request: Invalid input format.
* **PUT /api/users/{user_id}** :

  * **Description** : Update user profile.
  * **Input** : JSON body (partial updates allowed):

  ```json
  {
    "first_name": "string",
    "last_name": "string",
    "phone_number": "string",
    "profile_photo": "string (URL)"
  }
  ```

  ***Output** : JSON response (HTTP 200 OK):
  ``json       {         "user_id": "integer",         "email": "string",         "first_name": "string",         "last_name": "string",         "phone_number": "string",         "profile_photo": "string"       }       ``

  * **Validation Rules** :
    * User must be authenticated (via JWT).
    * Only the user can update their own profile (RBAC check).
    * Profile photo URL must be valid if provided.
  * **Error Handling** :
    * HTTP 403 Forbidden: User not authorized to update this profile.
    * HTTP 400 Bad Request: Invalid input data.
* **Authentication Mechanism** :
* Use JSON Web Tokens (JWT) for secure session management, included in the Authorization header (Bearer token).
* Implement role-based access control (RBAC) to restrict access based on user roles (guest, host, admin).
* **Database** :
* **Users Table** :| Field         | Type                           | Constraints                         |
  | --------------- | -------------------------------- | ------------------------------------- |
  | user_id       | BIGINT UNSIGNED                | Primary Key, Auto-increment         |
  | first_name    | VARCHAR(255)                   | Not Null                            |
  | last_name     | VARCHAR(255)                   | Not Null                            |
  | email         | VARCHAR(255)                   | Not Null, Unique                    |
  | password_hash | VARCHAR(255)                   | Not Null                            |
  | phone_number  | VARCHAR(255)                   | Not Null                            |
  | role          | ENUM('guest', 'host', 'admin') | Not Null                            |
  | profile_image | VARCHAR(255)                   | Nullable                            |
  | created_at    | TIMESTAMP                      | Not Null, Default CURRENT_TIMESTAMP |
* **Performance Criteria** :
* Registration and login endpoints should respond in under 1 second under normal load.
* Use caching (e.g., Redis) for user profile data to reduce database queries.
* Handle at least 1,000 concurrent authentication requests without significant latency.
* **Security** :
* Hash passwords using a secure algorithm (e.g., bcrypt).
* Implement rate limiting to prevent brute-force attacks.
* Use HTTPS for all API communications.

## 2. Property Management

### Functional Requirements

* **Add Listings** : Hosts can create property listings with details such as title, description, location, price per night, amenities, and availability.
* **Edit Listings** : Hosts can update existing property listings to reflect changes in details or availability.
* **Delete Listings** : Hosts can remove their property listings from the platform.
* **Search and Filter** : Guests can search properties by location, price range, number of guests, and amenities, with paginated results for usability.

### Technical Requirements

* **API Endpoints** :
* **POST /api/properties** :

  * **Description** : Create a new property listing.
  * **Input** : JSON body:

  ```json
  {
    "host_id": "integer",
    "name": "string",
    "description": "string",
    "location": "string",
    "price_per_night": "decimal",
    "amenities": ["string"]
  }
  ```

  ***Output** : JSON response (HTTP 201 Created):
  ``json       {         "property_id": "integer",         "host_id": "integer",         "name": "string",         "description": "string",         "location": "string",         "price_per_night": "decimal",         "amenities": ["string"],         "created_at": "timestamp"       }       ``

  * **Validation Rules** :
    * Host_id must match the authenticated user (RBAC check).
    * Name, description, location, and price_per_night are required.
    * Price_per_night must be positive.
    * Amenities must be a valid list of predefined options (e.g., Wi-Fi, pool).
  * **Error Handling** :
    * HTTP 403 Forbidden: User not authorized as a host.
    * HTTP 400 Bad Request: Invalid or missing fields.
* **PUT /api/properties/{property_id}** :

  * **Description** : Update an existing property listing.
  * **Input** : JSON body (partial updates allowed):

  ```json
  {
    "name": "string",
    "description": "string",
    "location": "string",
    "price_per_night": "decimal",
    "amenities": ["string"]
  }
  ```

  ***Output** : JSON response (HTTP 200 OK):
  ``json       {         "property_id": "integer",         "host_id": "integer",         "name": "string",         "description": "string",         "location": "string",         "price_per_night": "decimal",         "amenities": ["string"],         "updated_at": "timestamp"       }       ``

  * **Validation Rules** :
    * User must be authenticated and the host of the property.
    * Updated fields must meet the same validation as creation.
  * **Error Handling** :
    * HTTP 403 Forbidden: User not authorized to edit this property.
    * HTTP 404 Not Found: Property does not exist.
* **DELETE /api/properties/{property_id}** :

  * **Description** : Delete a property listing.
  * **Input** : None (property_id in URL).
  * **Output** : JSON response (HTTP 200 OK):

  ```json
  {
    "message": "Property deleted successfully"
  }
  ```

  ***Validation Rules** :

  * User must be authenticated and the host of the property.
  * **Error Handling** :

    * HTTP 403 Forbidden: User not authorized to delete this property.
    * HTTP 404 Not Found: Property does not exist.
* **GET /api/properties** :

  * **Description** : Search properties with filters.
  * **Input** : Query parameters:

  ```json
  {
    "location": "string",
    "min_price": "decimal",
    "max_price": "decimal",
    "guests": "integer",
    "amenities": ["string"],
    "page": "integer",
    "limit": "integer"
  }
  ```

  ***Output** : JSON response (HTTP 200 OK):
  ``json       {         "properties": [           {             "property_id": "integer",             "name": "string",             "location": "string",             "price_per_night": "decimal",             "amenities": ["string"]           }         ],         "total": "integer",         "page": "integer",         "limit": "integer"       }       ``

  * **Validation Rules** :
    * Location and amenities must be valid strings.
    * Min_price and max_price must be positive and min_price ≤ max_price.
    * Page and limit must be positive integers.
  * **Error Handling** :
    * HTTP 400 Bad Request: Invalid query parameters.
* **Database** :
* **Properties Table** :| Field           | Type            | Constraints                         |
  | ----------------- | ----------------- | ------------------------------------- |
  | property_id     | BIGINT UNSIGNED | Primary Key, Auto-increment         |
  | host_id         | BIGINT UNSIGNED | Foreign Key (Users.user_id)         |
  | name            | VARCHAR(255)    | Not Null                            |
  | description     | TEXT            | Not Null                            |
  | location        | VARCHAR(255)    | Not Null                            |
  | price_per_night | DECIMAL(8,2)    | Not Null                            |
  | created_at      | TIMESTAMP       | Not Null, Default CURRENT_TIMESTAMP |
  | updated_at      | TIMESTAMP       | Not Null, Default CURRENT_TIMESTAMP |
* **Property_Images Table** :| Field       | Type            | Constraints                          |
  | ------------- | ----------------- | -------------------------------------- |
  | image_id    | BIGINT UNSIGNED | Primary Key, Auto-increment          |
  | property_id | BIGINT UNSIGNED | Foreign Key (Properties.property_id) |
  | image_url   | VARCHAR(255)    | Not Null                             |
* **File Storage** :
* Store property images in cloud storage (e.g., [AWS S3](https://aws.amazon.com/s3/)).
* Store image URLs in the `Property_Images` table.
* **Performance Criteria** :
* Search queries should respond in under 500ms with proper indexing on `location` and `price_per_night`.
* Use caching (e.g., Redis) for frequently accessed search results.
* Support pagination with a default limit of 20 properties per page.
* Handle at least 500 concurrent search requests without performance degradation.
* **Security** :
* Restrict listing creation/editing/deletion to authenticated hosts via RBAC.
* Validate image URLs to prevent malicious content.

## 3. Booking System

### Functional Requirements

* **Booking Creation** : Guests can book a property for specified dates, with validation to prevent double bookings.
* **Booking Cancellation** : Guests or hosts can cancel bookings based on the cancellation policy.
* **Booking Status** : Track booking statuses (pending, confirmed, canceled, completed).

### Technical Requirements

* **API Endpoints** :
* **POST /api/bookings** :

  * **Description** : Create a new booking.
  * **Input** : JSON body:

  ```json
  {
    "property_id": "integer",
    "user_id": "integer",
    "start_date": "date (YYYY-MM-DD)",
    "end_date": "date (YYYY-MM-DD)"
  }
  ```

  ***Output** : JSON response (HTTP 201 Created):
  ``json       {         "booking_id": "integer",         "property_id": "integer",         "user_id": "integer",         "start_date": "date",         "end_date": "date",         "total_price": "decimal",         "status": "string (pending)"       }       ``

  * **Validation Rules** :
    * User must be authenticated and a guest.
    * Property_id must exist in the `Properties` table.
    * Start_date must be before end_date and both in the future.
    * Check availability: No overlapping bookings (confirmed or pending) in the `Bookings` table.
    * Calculate `total_price` based on `price_per_night` and number of nights.
  * **Error Handling** :
    * HTTP 403 Forbidden: User not authorized to book.
    * HTTP 409 Conflict: Dates not available.
    * HTTP 400 Bad Request: Invalid dates or missing fields.
* **PUT /api/bookings/{booking_id}/cancel** :

  * **Description** : Cancel a booking.
  * **Input** : None (booking_id in URL).
  * **Output** : JSON response (HTTP 200 OK):

  ```json
  {
    "booking_id": "integer",
    "status": "canceled"
  }
  ```

  ***Validation Rules** :

  * User must be authenticated and either the guest or host of the booking.
  * Booking must not already be canceled or completed.
  * Check cancellation policy (e.g., within allowed timeframe).
  * **Error Handling** :

    * HTTP 403 Forbidden: User not authorized to cancel.
    * HTTP 404 Not Found: Booking does not exist.
    * HTTP 400 Bad Request: Cancellation not allowed per policy.
* **GET /api/bookings/{booking_id}** :

  * **Description** : Retrieve booking details.
  * **Input** : booking_id in URL.
  * **Output** : JSON response (HTTP 200 OK):

  ```json
  {
    "booking_id": "integer",
    "property_id": "integer",
    "user_id": "integer",
    "start_date": "date",
    "end_date": "date",
    "total_price": "decimal",
    "status": "string"
  }
  ```

  ***Validation Rules** :

  * User must be authenticated and either the guest or host of the booking.
  * **Error Handling** :

    * HTTP 403 Forbidden: User not authorized to view.
    * HTTP 404 Not Found: Booking does not exist.
* **Database** :
* **Bookings Table** :| Field       | Type                                                  | Constraints                          |
  | ------------- | ------------------------------------------------------- | -------------------------------------- |
  | booking_id  | BIGINT UNSIGNED                                       | Primary Key, Auto-increment          |
  | property_id | BIGINT UNSIGNED                                       | Foreign Key (Properties.property_id) |
  | user_id     | BIGINT UNSIGNED                                       | Foreign Key (Users.user_id)          |
  | start_date  | DATE                                                  | Not Null                             |
  | end_date    | DATE                                                  | Not Null                             |
  | total_price | DECIMAL(8,2)                                          | Not Null                             |
  | status      | ENUM('pending', 'confirmed', 'canceled', 'completed') | Not Null                             |
  | created_at  | TIMESTAMP                                             | Not Null, Default CURRENT_TIMESTAMP  |
* **Availability Check** :
* Query `Bookings` table for overlapping bookings:

  ```sql
  SELECT * FROM Bookings
  WHERE property_id = :property_id
  AND status IN ('confirmed', 'pending')
  AND start_date < :end_date
  AND end_date > :start_date;
  ```
* If no results, the dates are available.
* **Payment Integration** :
* After creating a "pending" booking, redirect to a payment gateway (e.g., [Stripe](https://stripe.com/)).
* On successful payment, update booking status to "confirmed" and create a record in the `Payment` table.
* On payment failure, set booking status to "canceled."
* **Notifications** :
* Send email notifications to guests and hosts upon booking confirmation or cancellation using services like [SendGrid](https://sendgrid.com/).
* **Performance Criteria** :
* Availability checks should complete in under 200ms with proper indexing on `start_date`, `end_date`, and `property_id`.
* Booking creation should handle concurrent requests using database transactions to prevent race conditions.
* Support at least 200 concurrent booking requests without significant latency.
* **Security** :
* Restrict booking creation to authenticated guests and cancellation to authorized users (guest or host).
* Use transactions to ensure atomicity during booking creation and payment processing.
