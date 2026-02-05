erDiagram
    user {
        string ref PK
        string firebase_uid UK "for auth | index"
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
        string user_ref FK "UK"
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
        string user_ref FK "UK"
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
        string category_ref FK "index"
        string creator_ref FK "created by user | index"
        string title
        string description
        string location_name "location in human readable"
        geometry location_point "Point, 4326- location in geo | index"
        int max_attendees
        string visibility "public | private | friends_only- index"
        string status "draft | published | cancelled | completed"
        datetime start_time "index"
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
        string user_ref FK "index"
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
        string event_ref FK "index"
        string inviter_ref FK
        string invitee_ref FK
        string status "pending | accepted | declined"
        datetime sent_at
        datetime responded_at
        datetime created_at
    }

    event_image {
        string ref PK
        string event_ref FK "index"
        string url
        int display_order
        boolean is_cover
        datetime created_at
    }

    chat_message {
        string ref PK
        string user_ref FK "index"
        string event_ref FK "index"
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
        string user_ref PK "FK"
        string interest_ref PK "FK"
    }

    user_relationship {
        string ref PK
        string user_ref FK "index"
        string related_user_ref FK "index:
        string type "friend | blocked | following"
        string status "pending | accepted | blocked"
        datetime created_at
        datetime updated_at
    }

    notification {
        string ref PK
        string user_ref FK "index"
        string type "event_invite | event_reminder | message | system"
        string title
        string body
        jsonb data
        boolean read
        string action_url
        datetime read_at
        datetime created_at 
        datetime deleted_at
    }

    report {
        string ref PK
        string reporter_ref FK
        string reported_event_ref FK
        string reported_user_ref FK
        string reported_message_ref FK
        string reason
        string description
        string status "pending | reviewed | resolved"
        datetime created_at
        datetime resolved_at
        datetime updated_at
    }

    feature_flag {
        string ref PK
        string key UK
        boolean enabled
        jsonb rules
    }

    user ||--|| user_profile : "has"
    user ||--o| subscription : "subscribed to"
    user ||--o{ event : "created"
    user ||--o{ event_attendee : "attends"
    user ||--o{ event_invitation : "sends"
    user ||--o{ event_invitation : "receives"
    user ||--o{ chat_message : "writes"
    user ||--o{ notification : "receives"
    user ||--o{ user_interest : "has"
    user ||--o{ user_relationship : "initates"
    user ||--o{ report : "files"

    event ||--o{ event_attendee : "has"
    event ||--o{ event_invitation : "for"
    event ||--o{ event_image : "has"
    event ||--o{ chat_message : "contains"
    event ||--|| category : "belongs_to"

    interest ||--o{ user_interest : "tagged_to"