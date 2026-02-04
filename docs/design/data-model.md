erDiagram
    user {
        string ref PK
        string firebase_uid UK "for auth"
        string email "just for emails"
        string display_name
        string first_name
        string last_name
        boolean email_verified
        string role "user | admin- for now"
        datetime last_login
        datetime updated_at
        datetime deleted_at
        datetime date_created
    }

    user_profile {
        string ref PK
        string user_ref FK UK
        string bio
        string home_city
        geometry home_location "Point, 4326"
        string avatar_url
        jsonb preferences
        datetime created_at
        datetime updated_at
    }


    subscription {
        string ref PK
        string user_ref FK UK
        string stripe_subscription_id UK
        string stripe_customer_id
        string plan_type "free | premium | enterprise"
        string status "active | canceled | past_die | unpaid"
        datetime current_period_start
        datetime current_period_end
        datetime created_at
        datetime updated_at
        datetime canceled_at
    }

    event {
        string ref PK
        string category_ref FK
        string creator_ref FK "created by user"
        string title
        string description
        string location_name "location in human readable"
        geometry location_point "Point, 4326- location in geo"
        int max_attendees
        string visibility "public | private | friends_only"
        string status "draft | published | cancelled | completed"
        datetime start_time
        datetime end_time
        string cover_image_url
        string qr_code_hash UK "unique QR code hash"
        datetime qr_code_generated_at
        jsonb custom_fields
        datetime created_at
        datetime updated_at
        datetime deleted_at
    }

    chat_message {
        string ref PK
        string user_ref FK
        string event_ref FK
        string content
        datetime created_at
    }

    location {
        string ref PK
        string user_ref FK
        geometry geom "Point, 4326"
        datetime created_at
    }

    interest {
        string ref PK
        string name
    }

    category {
        string ref PK
        string name
        string bio
        string icon
    }

    notification {
        string ref PK
        string user_ref FK
        string type
        jsonb payload
        boolean read
        datetime created_at 
    }

    event_attendee {
        string user_ref FK
        string event_ref FK "index"
        string status "signed_up | attended | did_not_attend"
        datetime qr_code_submitted
    }

    user_interests {
        string user_ref PK "FK"
        string interest_ref PK "FK"
    }

    feature_flag {
        string ref PK
        string key UK
        boolean enabled
        jsonb rules
    }

    user ||--o| subscription : "owns"
    user ||--o| location : "updates"
    user ||--o{ user_interests : "selects"
    interest ||--o{ user_interests : "assigned to"
    category ||--o{ event : "contains"

    event ||--o| user : "created by"
    user ||--o{ event_attendee : "checks into"
    event ||--o{ event_attendee : "records"

    user ||--o{ chat_message : "can create"
    event ||--o{ chat_message : "can host"