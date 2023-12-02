# Backend Design Test

## Assumptions

### Functionality

- Auction feature:
  - Will be just a simple booking with payment higher than current active booking amount.
  - The payment must be a minimum current active booking amount plus minimum auction charge for that table.
  - The payment must be made in full.

### Technicallity

- Auth managed with simple design to show case sequence diagram.
  - In reality should use provider like Firebase Identity.
- Venue details (name, address, rating, distance) using Google Maps API.

## Diagrams

### Sequence Diagram

#### Forgot Password

```mermaid
sequenceDiagram
  User ->> Frontend : Forgot Password using Phone/Email
  Frontend ->> Backend : /auth/forgot-password
  Backend -->> Frontend : resetToken
  Frontend -->> User : Ask the code
  Backend ->> User : Send code via phone/email
  User ->> Frontend : Code
  Frontend ->> Backend : /auth/reset-password
  Backend -->> Frontend : changeToken
  Frontend -->> User : Ask new password
  User ->> Frontend : New password
  Frontend ->> Backend : /auth/change-password
```

### Entity Relationship Diagram

- All table have timestamps (create, update, dekete) and it is omitted unless necessary for verbosity.

```mermaid
erDiagram
    USERS {
      uuid id PK
      string username
      string phone
      string email
      string password
      string full_name
      bool is_phone_verified
      bool is_email_verified
      int photo_id FK
      int rank_id FK
    }

    PHOTOS {
      int id PK
      blob photo
    }

    FRIENDS {
      int id PK
      uuid requester_id FK
      uuid addressee_id FK
      datetime approved
    }

    NOTIFICATIONS {
      int id PK
      uuid user_id FK
      string type
      string message
    }

    RANKS {
      int id PK
      int priority
      string name
    }

    RANK_TASKS {
      int id PK
      int rank_id FK
      string title
      string description
    }

    USER_TASKS {
      int id PK
      uuid user_id FK
      int rank_task_id FK
    }

    DJS {
      uuid id PK
      string name
      int photo_id FK
      uuid user_id FK
    }

    VENUES {
      uuid id PK
      string name
      string location_id
    }

    VENUE_FACILITIES {
      int id PK
      uuid venue_id FK
      int priority
      string title
      string description
    }

    VENUE_OFFERS {
      int id PK
      uuid venue_id FK
      int priority
      string title
      string description
      int photo_id FK
    }

    VENUE_BEVERAGES {
      int id PK
      uuid venue_id FK
      int priority
      string title
      string description
      int photo_id FK
    }

    TABLES {
      uuid id PK
      uuid venue_id FK
      string name
      real min_deposit
      real min_auction_charge
    }

    TABLE_FACILITIES {
      int id PK
      uuid table_id FK
      string title
      string description
      blob icon
    }

    EVENTS {
      uuid id PK
      uuid venue_id FK
      string name
      date date
      string description
      id poster_id FK
    }

    EVENT_PHOTOS {
      int id PK
      uuid event_id FK
      int photo_id FK
      int priority
    }

    EVENT_DJS {
      int id PK
      uuid event_id FK
      uuid dj_id FK
      datetime start
      datetime end
    }

    BOOKING {
      uuid id PK
      uuid user_id FK
      uuid table_id FK
      date date
      real auction_amount
      real discount_amount
      real service_amount
      real charge_amount
      real paid_amount
      datetime fully_paid_at
      datetime lost_auction_at
      real refund_amount
    }

    BOOKING_PAYMENTS {
      uuid id PK
      uuid booking_id FK
      int payment_method FK
      real amount
      json charge_payload
      json success_response
      json failed_response
    }

    INVITATIONS {
      int id PK
      uuid booking_id FK
      uuid requester_id FK
      uuid addressee_id FK
      datetime approved
    }

    PAYMENT_METHODS {
      int id PK
      string provider
      string method
    }

    USERS }o--o{ FRIENDS : have
    USERS ||--o| PHOTOS : has
    USERS ||--o{ NOTIFICATIONS : have
    USERS ||--|| RANKS : is
    RANKS ||--|{ RANK_TASKS : have
    USERS ||--o{ USER_TASKS : have
    RANK_TASKS ||--|{ USER_TASKS : have
    DJS ||--o| USERS : is
    DJS ||--o| PHOTOS : has
    VENUES ||--o{ VENUE_FACILITIES : have
    VENUES ||--o{ VENUE_BEVERAGES : have
    VENUES ||--o{ VENUE_OFFERS : have
    VENUES ||--o{ EVENTS : have
    VENUES ||--o{ TABLES : have
    VENUES ||--o| PHOTOS : has
    VENUE_BEVERAGES ||--o| PHOTOS : has
    VENUE_OFFERS ||--o| PHOTOS : has
    EVENTS ||--o| PHOTOS : has
    EVENTS ||--o{ EVENT_PHOTOS : have
    EVENTS ||--o{ EVENT_DJS : have
    EVENT_PHOTOS ||--o| PHOTOS : has
    BOOKING ||--o{ BOOKING_PAYMENTS : have
    BOOKING_PAYMENTS }|--|| PAYMENT_METHODS : have
    TABLES ||--o{ BOOKING : have
    BOOKING ||--o{ INVITATIONS : have
    USERS }o--o{ INVITATIONS : have
    TABLES ||--o{ TABLE_FACILITIES : have
    USERS ||--o{ BOOKING : have
```

## Endpoints

- Common error response:
  ```
  {
    error: string;    // error code
    message: string;  // message for user
  }
  ```

### Auth

- Default error if not specified is `unauthorized`.

#### Login

- Login Token

  ```
  POST /auth/token

  Payload: {
    username | phone | email: string;
    password: string;
  }

  Return: {
    accessToken: string;
    expiresIn: number;
  }
  ```

#### Registration

- Register

  ```
  POST /auth/register

  Payload: {
    username?: string;
    phone?: string;
    email?: string;
    password: string;
  }

  Return: {
    accessToken: string;
    expiresIn: number;
  }
  ```

  Code will be send to phone/email.

- Phone/Email Verification

  ```
  POST /auth/verify

  Payload: {
    phone | email: string;
    code: string;
  }

  Return: No Content

  Error:
  - code-expired
  ```

#### Forgot Password

- Forgot Password

  ```
  POST /auth/forgot-password

  Payload: {
    phone | email: string;
  }

  Return: {
    resetToken: string;
    expiresIn: number;
  }
  ```

  Code will be send to the phone/email.

- Reset Password

  ```
  POST /auth/reset-password

  Payload: {
    resetToken: string;
    code: string;
  }

  Return: {
    changeToken: string;
    expiresIn: number;
  }
  ```

- Change Password

  ```
  POST /auth/change-password

  Payload: {
    oldPassword | changeToken: string;
    newPassword: string;
  }

  Return: No Content
  ```

### User

- Change Profile

  ```
  POST /me/profile

  Payload: {
    email?: string;
    phone?: string;
    fullName?: string;
    photo?: blob;
  }

  Return: No Content
  ```

  Code will be send to the phone/email if its changed.

- Request Code to verify Phone/Email

  ```
  GET /me/verify

  Query: {
    field: 'phone' | 'email';
  }

  Return: No Content
  ```

  Code will be send to the phone/email only if its not verified.
