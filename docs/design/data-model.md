erDiagram
    user {
        int id
        string ref PK
        string firebase_uid "unique"
        string email "unique"
        string display_name
        string first_name
        string last_name
        string bio
        string home_city
        boolean email_verified
        string avatar "URL to image"
        string role "user | admin- for now"
        string tier "free | paid | premium"
        datetime updated_at
        datetime deleted_at
        datetime date_created
    }

    subscription {
        int id
        string ref PK
        string user_ref FK
        string stripe_id "id for stripe connection"

        datetime created_at
        datetime updated_at
        datetime deleted_at
    }

    event {
        int id
        string ref PK
        string category_ref FK
        string user_ref FK "index as created_by"
        string title
        string bio
        string location "location in human readable"
        geometry geom "Point, 4326- location in geo"
        int users_allowed "max number users can join"
        int users_joined "current amount of users joined"
        datetime start_time
        datetime end_time
        int duration "in minutes"
        string qr_code "unique QR code hash"
        datetime qr_code_generated_at
        datetime created_at
        datetime updated_at
        datetime deleted_at
    }

    location {
        int id
        string ref PK
        string user_ref FK
        geometry geom "Point, 4326"
        datetime created_at
    }

    interest {
        int id
        string ref PK
        string name
    }

    category {
        int id
        string ref PK
        string name
        string bio
        string icon
    }

    event_attendance {
        int id PK
        string user_ref FK
        string event_ref FK "index"
        datetime qr_code_submitted
    }

    user_interests {
        string user_ref PK "FK"
        string interest_ref PK "FK"
    }

    user ||--o| subscription : "owns"
    user ||--o| location : "updates"
    user ||--o{ user_interests : "selects"
    interest ||--o{ user_interests : "assigned to"
    category ||--o{ event : "contains"

    event ||--o| user : "created by"
    user ||--o{ event_attendance : "checks into"
    event ||--o{ event_attendance : "records"