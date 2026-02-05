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
        datetime updated_at
    }

    subscription {
        string ref PK
        string user_ref FK "UK"
        string stripe_subscription_id UK
        string stripe_customer_id UK
        string plan_type
        string status
        datetime current_period_start
        datetime current_period_end
        datetime created_at
        datetime updated_at
        datetime canceled_at
    }

    event {
        string ref PK
        string category_ref FK
        string creator_ref FK
        string title
        string description
        string location_name
        geometry location_point "Point, 4326"
        int max_attendees
        string visibility
        string status
        datetime start_time
        datetime end_time
        string cover_image_url
        string qr_code_hash UK
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
- `user_firebase_uid_key` index: Used for every authenticated API request
- `idx_user_email_active` partial index: Only index non-deleted users for faster queries

### user_profile
Extension table which provides descriptive information for a user with separation of concerns from the user table as it is frequently updated.

**Columns**
- `ref` (PK): UUID primary key
- `user_ref` (FK): UUID foreign key from `user` table 1:1 relationship
- `bio`: Description the user can write about themselves
- `home_city`: Selected dropdown of cities globally
- `home_location`: Geometry points of the location where they are
- `avatar_url`: URL for the location of their profile picture avatar
- `preferences`: JSON format of their selected preferences
- `updated_at`: Audit log for last updated

**Indexes**
```sql
-- Primary key (auto-created)
CREATE UNIQUE INDEX user_profile_pkey ON user_profile(ref);

-- Unique constraint on foreign key for join
CREATE UNIQUE INDEX user_ref_key ON user_profile(user_ref);
```

**Rationale**
- `user_ref_key` unique index: Enforces 1:1 relationship as one profile per user and enables faster lookup for a user and their joining user_profile.

### subscription
Table that holds the information related to the users subscription with Stripe.

**Columns**
- `ref` (PK): UUID for primary key
- `user_ref` (FK): UID foreign key for relationship with a user
- `stripe_subscription_id`: Unique identifier for the users stripe subscription
- `stripe_customer_id`: Unique identifier for the users stripe id
- `plan_type`: Which plan the user is on "free | premium | enterprise"
- `status`: The status of their subscription "active | canceled | past_due | unpaid"
- `current_period_start`: Starting date of the current plan
- `current_period_end`: End date of the current plan
- `cancled_at`: Date of canceled plan or NULL

**Indexes**
```sql
-- Primary key (auto-created)
CREATE UNIQUE INDEX subscription_pkey ON subscription(ref);

-- Identify subscription associated with user
CREATE UNIQUE INDEX user_ref_idx ON subscription(user_ref);

-- Stripe webhook lookups (frequent)
CREATE UNIQUE INDEX idx_subscription_stripe_id ON subscription(stripe_subscription_id) WHERE stripe_subscription_id IS NOT NULL;

-- Stripe customer lookup
CREATE INDEX idx_subscription_stripe_customer ON subscription(stripe_customer_id) WHERE stripe_customer_id IS NOT NULL;

-- View active subscriptions by tier
CREATE INDEX idx_subscription_active_tier ON subscription(plan_type, status) WHERE status = 'active';

-- View expiring subscriptions
CREATE INDEX idx_subscription_expiring ON subscription(current_period_end) WHERE status = 'active';

-- View past due subscriptions
CREATE INDEX idx_subscription_past_due ON subscription(status, updated_at) WHERE status = 'past_due'
```

**Rationale**
- `user_ref_idx` unique index: FK with unique constraint that enforces 1:1 relationship with `user` table.
- `idx_subscription_stripe_id` unique index: Processing stripe webhooks by subscription.
- `idx_subscription_stripe_customer` index: Finding users subscription by customer id.
- `idx_subscription_active_tier` partial index: Finding active subscriptions by all plan types.
- `idx_subscription_expiring` partial index: Finding active subscriptions period ending.
- `idx_subscription_past_due` partial index: Finding subscriptions past due and checking `updated_at` for any that haven't been retrieved recently.

### event
Primary entity for an event which contains the details for the event created by a user.

**Columns**
- `ref` (PK): UUID for the primary key
- `category_ref` (FK): Foreign key which creates relation to `category` table
- `creator_ref` (FK): Foreign key which creates the relation to `user` table for who created the event
- `title`: Title for the event
- `description`: Description for the event
- `location_name`: Human readable format for the location of the event
- `location_point`: Geometric point data for the location of the event
- `max_attendees`: Max number of users who can sign up and attend
- `visibility`: Which type of users can view the event "public | private | friends_only"
- `status`: What status the event is currently in "draft | published | cancelled | completed"
- `start_time`: Date and time event starts
- `end_time`: Date and time event ends
- `cover_image_url`: URL for the image of the event cover
- `qr_code_hash`: Unique QR code created for the event attendees need to scan to join face-to-face
- `qr_code_generated_at`: Date and time QR code was generated at to have expiration
- `custom_fields`: JSON for any custom fields related to the event
- `created_at`: Audit log for when event was created it
- `updated_at`: Audit log for when updates made to the event
- `deleted_at`: Audit log for when event is deleted

**Indexes**
```sql
-- Primary key (auto-created)
CREATE UNIQUE INDEX event_pkey ON event(ref);

-- Foreign key indexes (for joins)
CREATE INDEX idx_event_category_fk ON event(category_ref);
CREATE INDEX idx_event_creator_fk ON event(creator_ref);

-- Event discovery
CREATE INDEX idx_event_discovery ON event(status, start_time) WHERE deleted_at IS NULL;

-- Category browsing
CREATE INDEX idx_event_category ON event(category_ref, start_time) WHERE deleted_at IS NULL;

-- Find event locations that are active (map view)
CREATE INDEX idx_active_event_location ON event USING GIST (location_point) WHERE deleted_at IS NULL AND status = 'published';

-- QR code verification for attendee check in
CREATE UNIQUE INDEX idx_event_qr_code ON event(qr_code_hash) WHERE qr_code_hash IS NOT NULL;
```

**Rationale**
- `idx_event_discovery` partial index: Finding events that are published and starting soon that are also not deleted.
- `idx_event_category` partial index: Find events under a certain category that are not deleted.
- `idx_active_event_location` GIST index: Find the geo location of an event quickly for location searching for active events.
- `idx_event_qr_code` unique index: Ensure that the QR code for each event is unique.