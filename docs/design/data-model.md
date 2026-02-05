# VibeMeet Data Model

## Entity Relationship Diagram (ERD)
```mermaid
erDiagram
    user {
        string ref PK
        string firebase_uid UK
        string email
        string display_name
        string first_name
        string last_name
        boolean email_verified
        string role
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
        string status "active | canceled | past_due | unpaid"
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
        string status "draft | published | cancelled | completed- index"
        datetime start_time "composite index (status, start_time) AND deleted_at = NULL"
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
```

## Tables

### user
Primary table for user accounts, this is linked to the Firebase Auth.

**Columns**
- `ref` (PK): UUID primary key
- `firebase_uid` (UK): Firebase authentication UID
- `email`: User email address used for verification and email contact to user
- `display_name`: The name the user would like displayed for others- can be different to first_name + last_name
- `first_name`: The users first name
- `last_name`: The users last name
- `email_verified`: Checking if the user has verified, will be used to show others if verified- may limit access to certain things
- `role`: Differentiate between the different authorisation- "user | admin"
- `last_login`: Shows the last login time, will be used for determining account expiration (unsure)
- `updated_at`: Audit trail for last updates
- `deleted_at`: Audit trail for deleted account- soft delete
- `date_created`: Used for determining length of time on the app

**Indexes**
```sql
-- Primary key (auto-created)
CREATE UNIQUE INDEX user_pkey ON user(ref);

-- Unique constraint on Firebase UID for auth lookups
CREATE UNIQUE INDEX user_firebase_uid_key ON user(firebase_uid);

-- Email lookup for active users only
CREATE INDEX idx_user_email_active ON user(email) WHERE deleted_at IS NULL;
```

**Rationale**
- `firebase_uid` index: Used for every authenticated API request
- `email` partial index: Only index non-deleted users for faster queries