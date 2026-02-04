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

    event_attendee {
        string ref PK
        string user_ref FK
        string event_ref FK "index"
        string status "pending | confirmed | checked_in | no_show | cancelled"
        datetime joined_at
        datetime checked_in_at
        datetime cancelled_at
        datetime created_at
        datetime updated_at
    }

    event_invitation {
        string ref PK
        string event_ref FK
        string inviter_ref FK
        string invitee_ref FK
        string status "pending | accepted | declined"
        datetime sent_at
        datetime responded_at
        datetime created_at
    }

    event_image {
        string ref PK
        string event_ref FK
        string url
        int display_order
        boolean is_cover
        datetime created_at
    }

    chat_message {
        string ref PK
        string user_ref FK
        string event_ref FK
        string content
        string message_type "text | image | system"
        boolean edited
        datetime edited_at
        datetime created_at
        datetime deleted_at
    }

    category {
        string ref PK
        string name UK
        string description
        string icon_url
    }

    interest {
        string ref PK
        string name UK
        string description
    }

    user_interest {
        string user_ref PK FK
        string interest_ref PK FK
    }

    user_relationship {
        string ref PK
        string user_ref FK
        string related_user_ref FK
        string type "friend | blocked | following"
        string status "pending | accepted | blocked"
        datetime created_at
        datetime updated_at
    }

    notification {
        string ref PK
        string user_ref FK
        string type
        jsonb payload
        boolean read
        datetime created_at 
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