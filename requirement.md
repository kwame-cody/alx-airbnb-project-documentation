# 🏡 Airbnb Clone: Backend Feature Requirements

This document details the technical and functional requirements for key backend features of the Airbnb Clone, as a follow-up to the initial PRD.

---

### 1. User Authentication

**Objective:** To provide a secure and efficient way for users to register, log in, and manage their sessions.

#### Functional Requirements

- Allow users to sign up with a unique email and a password.
- Support login using email and password.
- Generate and return a **JWT (JSON Web Token)** upon successful authentication.
- Validate JWTs on all protected routes to ensure secure access.
- Provide a mechanism for users to update their password.

#### API Endpoints

- **Endpoint:** `POST /api/auth/register`

  - **Functionality:** Registers a new user.
  - **Input:**
    - `email` (string, required, valid email format)
    - `password` (string, required, min 8 characters)
    - `role` (string, required, "guest" or "host")
  - **Output:**
    - `201 Created`: `{ "message": "User registered successfully." }`
    - `400 Bad Request`: `{ "error": "Invalid input data." }`
    - `409 Conflict`: `{ "error": "Email already exists." }`

- **Endpoint:** `POST /api/auth/login`
  - **Functionality:** Authenticates a user and returns a JWT.
  - **Input:**
    - `email` (string, required)
    - `password` (string, required)
  - **Output:**
    - `200 OK`: `{ "token": "...", "user": { "id": "...", "role": "..." } }`
    - `401 Unauthorized`: `{ "error": "Invalid credentials." }`

#### Validation Rules

- Passwords must be at least 8 characters long and contain at least one uppercase letter, one number, and one special character.
- Email addresses must be in a valid format and unique within the database.

#### Performance Criteria

- Authentication responses should be returned in under **100ms**.
- The system should handle at least 500 concurrent authentication requests per second.

---

### 2. Property Listings Management

**Objective:** To enable hosts to create, read, update, and delete property listings and allow guests to view them.

#### Functional Requirements

- Hosts can create a new listing with required fields.
- Hosts can edit or delete their own listings.
- Guests can view all listings, including details and images.
- The system will handle image uploads and associate them with the correct listing.

#### API Endpoints

- **Endpoint:** `POST /api/listings`

  - **Functionality:** Creates a new property listing (protected for Hosts).
  - **Input:**
    - `title` (string, required)
    - `description` (string, required)
    - `location` (object, required: `city`, `country`)
    - `price` (number, required)
    - `amenities` (array of strings)
    - `images` (array of image files for upload)
  - **Output:**
    - `201 Created`: `{ "message": "Listing created successfully.", "listingId": "..." }`
    - `401 Unauthorized`: `{ "error": "Authentication required." }`
    - `403 Forbidden`: `{ "error": "Only hosts can create listings." }`

- **Endpoint:** `GET /api/listings`

  - **Functionality:** Retrieves a list of all listings with optional search/filter parameters.
  - **Input (Query Params):**
    - `location` (string)
    - `minPrice` (number)
    - `maxPrice` (number)
    - `amenities` (comma-separated string)
  - **Output:**
    - `200 OK`: `[ { "id": "...", "title": "...", "price": "...", ... }, ... ]`
    - `404 Not Found`: `{ "message": "No listings found." }`

- **Endpoint:** `GET /api/listings/{id}`

  - **Functionality:** Retrieves a single listing by its ID.
  - **Output:**
    - `200 OK`: `{ "id": "...", "title": "...", "description": "...", ... }`
    - `404 Not Found`: `{ "error": "Listing not found." }`

- **Endpoint:** `PUT/PATCH /api/listings/{id}`
  - **Functionality:** Updates an existing listing (protected for Hosts).
  - **Output:**
    - `200 OK`: `{ "message": "Listing updated successfully." }`
    - `401 Unauthorized`: `{ "error": "Authentication required." }`
    - `403 Forbidden`: `{ "error": "You can only update your own listings." }`

#### Validation Rules

- `title` must be between 10 and 100 characters.
- `price` must be a positive number.
- `location` fields are required.
- Image files must be of type JPEG, PNG, or GIF.

#### Performance Criteria

- Retrieving a single listing should be under **50ms**.
- The search and filtering API should return results in under **200ms**, even with large datasets.

---

### 3. Booking System

**Objective:** To enable guests to book properties and prevent conflicting reservations.

#### Functional Requirements

- Guests can create a new booking for specific check-in and check-out dates.
- The system must validate that the selected dates are available before a booking is confirmed.
- Guests can view their booking history.
- Bookings can be canceled by the guest or host, with appropriate status updates.

#### API Endpoints

- **Endpoint:** `POST /api/bookings`

  - **Functionality:** Creates a new booking (protected for Guests).
  - **Input:**
    - `listingId` (string, required)
    - `checkInDate` (date, required)
    - `checkOutDate` (date, required)
    - `guestId` (string, derived from authenticated user)
  - **Output:**
    - `201 Created`: `{ "message": "Booking request sent.", "bookingId": "..." }`
    - `400 Bad Request`: `{ "error": "Dates are not available." }`
    - `401 Unauthorized`: `{ "error": "Authentication required." }`

- **Endpoint:** `GET /api/bookings/{userId}`

  - **Functionality:** Retrieves all bookings for a specific user (protected).
  - **Output:**
    - `200 OK`: `[ { "id": "...", "listing": "...", "status": "pending", ... }, ... ]`
    - `401 Unauthorized`: `{ "error": "Authentication required." }`
    - `403 Forbidden`: `{ "error": "You can only view your own bookings." }`

- **Endpoint:** `PUT/PATCH /api/bookings/{id}`
  - **Functionality:** Updates a booking status (e.g., to "canceled").
  - **Input:**
    - `status` (string, required, "confirmed", "canceled", or "completed")
  - **Output:**
    - `200 OK`: `{ "message": "Booking updated successfully." }`
    - `403 Forbidden`: `{ "error": "You do not have permission to update this booking." }`

#### Validation Rules

- `checkInDate` must be before `checkOutDate`.
- The date range for a new booking must not overlap with any existing confirmed booking for the same property.
- Booking requests for dates in the past must be rejected.

#### Performance Criteria

- The booking validation process should be completed in under **150ms** to provide a quick response to the user.
- Retrieving a user's booking history should not exceed **100ms** with up to 1,000 bookings.

---

### 4. Reviews and Ratings

**Objective:** To enable guests to review properties and hosts to respond to reviews, building trust within the community.

#### Functional Requirements

- Guests can submit a star rating (1-5) and a written review for a completed booking.
- A guest can only submit one review per booking.
- Hosts can view and respond to reviews on their listings.
- Reviews and ratings are public and displayed on the property listing page.

#### API Endpoints

- **Endpoint:** `POST /api/reviews`

  - **Functionality:** Submits a new review for a completed booking (protected for Guests).
  - **Input:**
    - `bookingId` (string, required)
    - `rating` (integer, required, 1-5)
    - `comment` (string, required)
  - **Output:**
    - `201 Created`: `{ "message": "Review submitted successfully." }`
    - `400 Bad Request`: `{ "error": "Invalid booking ID or review already submitted." }`

- **Endpoint:** `GET /api/reviews/{listingId}`

  - **Functionality:** Retrieves all reviews for a specific property listing.
  - **Output:**
    - `200 OK`: `[ { "id": "...", "rating": 5, "comment": "...", "reviewer": "...", ... }, ... ]`
    - `404 Not Found`: `{ "error": "No reviews found for this listing." }`

- **Endpoint:** `POST /api/reviews/{id}/respond`
  - **Functionality:** Allows a host to respond to a review (protected for Hosts).
  - **Input:**
    - `responseComment` (string, required)
  - **Output:**
    - `200 OK`: `{ "message": "Response submitted successfully." }`
    - `403 Forbidden`: `{ "error": "You can only respond to reviews on your own listings." }`

#### Validation Rules

- The `rating` must be an integer between 1 and 5.
- A review can only be posted for a booking that has a "completed" status.
- A user can only submit one review per `bookingId`.

#### Performance Criteria

- Submitting a review should be under **200ms**.
- Retrieving all reviews for a listing should not exceed **150ms**.

---

### 5. Messaging System

**Objective:** To enable private communication between guests and hosts regarding specific bookings or listings.

#### Functional Requirements

- Users can send and receive private messages within the platform.
- Messages are organized into conversations or threads associated with a `bookingId` or a `listingId`.
- The system should notify users of new incoming messages in real-time.
- The system should store a history of all messages in a conversation.

#### API Endpoints

- **Endpoint:** `POST /api/messages`

  - **Functionality:** Sends a new message to a specific user (protected).
  - **Input:**
    - `recipientId` (string, required)
    - `message` (string, required)
    - `bookingId` (string, optional)
  - **Output:**
    - `201 Created`: `{ "message": "Message sent successfully.", "messageId": "..." }`
    - `400 Bad Request`: `{ "error": "Cannot send a message to yourself." }`

- **Endpoint:** `GET /api/messages/{conversationId}`
  - **Functionality:** Retrieves all messages within a specific conversation.
  - **Output:**
    - `200 OK`: `[ { "senderId": "...", "message": "...", "timestamp": "..." }, ... ]`
    - `404 Not Found`: `{ "error": "Conversation not found." }`

#### Validation Rules

- Messages cannot be empty.
- A user cannot send a message to themselves.
- The `recipientId` must correspond to a valid user in the system.

#### Performance Criteria

- Sending a message should be completed in under **100ms**.
- Retrieving a conversation with up to 100 messages should not exceed **200ms**.
