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

    plan {
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
        datetime qr_code_expires_at
        jsonb custom_fields
        datetime created_at
        datetime updated_at
        datetime deleted_at
    }

    plan_attendee {
        string ref PK
        string user_ref FK "index"
        string plan_ref FK "index"
        string status "pending | confirmed | checked_in | no_show | cancelled"
        datetime joined_at
        datetime checked_in_at
        datetime cancelled_at
        datetime created_at
        datetime updated_at
    }

    plan_invitation {
        string ref PK
        string plan_ref FK
        string inviter_ref FK
        string invitee_ref FK
        string status
        datetime sent_at
        datetime responded_at
    }

    plan_image {
        string ref PK
        string plan_ref FK
        string url
        int display_order
        boolean is_cover
        datetime created_at
    }

    chat_message {
        string ref PK
        string user_ref FK
        string plan_ref FK
        string content
        string message_type
        string status
        datetime created_at
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
        string type "plan_invite | plan_reminder | message | system"
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
        string reported_plan_ref FK
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
    user ||--o{ plan : "created"
    user ||--o{ plan_attendee : "attends"
    user ||--o{ plan_invitation : "sends"
    user ||--o{ plan_invitation : "receives"
    user ||--o{ chat_message : "writes"
    user ||--o{ notification : "receives"
    user ||--o{ user_interest : "has"
    user ||--o{ user_relationship : "initates"
    user ||--o{ report : "files"

    plan ||--o{ plan_attendee : "has"
    plan ||--o{ plan_invitation : "for"
    plan ||--o{ plan_image : "has"
    plan ||--o{ chat_message : "contains"
    plan ||--|| category : "belongs_to"

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
- `user_ref` (FK): UID foreign key for relationship with a `user`
- `stripe_subscription_id`: Unique identifier for the users stripe subscription
- `stripe_customer_id`: Unique identifier for the users stripe id
- `plan_type`: Which subscription plan the user is on "free | premium | enterprise"
- `status`: The status of their subscription "active | canceled | past_due | unpaid"
- `current_period_start`: Starting date of the current subscription plan
- `current_period_end`: End date of the current subscription plan
- `cancled_at`: Date of canceled subscription plan or NULL

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

### plan
Primary entity for an plan which contains the details for the plan created by a user.

**Columns**
- `ref` (PK): UUID for the primary key
- `category_ref` (FK): Foreign key which creates relation to `category` table
- `creator_ref` (FK): Foreign key which creates the relation to `user` table for who created the plan
- `title`: Title for the plan
- `description`: Description for the plan
- `location_name`: Human readable format for the location of the plan
- `location_point`: Geometric point data for the location of the plan
- `max_attendees`: Max number of users who can sign up and attend
- `visibility`: Which type of users can view the plan "public | members | premium | friends_only"
- `status`: What status the plan is currently in "draft | published | cancelled | completed"
- `start_time`: Date and time plan starts
- `end_time`: Date and time plan ends
- `cover_image_url`: URL for the image of the plan cover
- `qr_code_hash`: Unique QR code created for the plan attendees need to scan to join face-to-face
- `qr_code_generated_at`: Date and time QR code was generated at to have expiration
- `qr_code_expires_at`: Date and time QR code expires so no one can scan again
- `custom_fields`: JSON for any custom fields related to the plan
- `created_at`: Audit log for when plan was created it
- `updated_at`: Audit log for when updates made to the plan
- `deleted_at`: Audit log for when plan is deleted

**Indexes**
```sql
-- Primary key (auto-created)
CREATE UNIQUE INDEX plan ON plan(ref);

-- Foreign key indexes (for joins)
CREATE INDEX idx_plan_category_fk ON plan(category_ref);
CREATE INDEX idx_plan_creator_fk ON plan(creator_ref);

-- Plan discovery
CREATE INDEX idx_plan_discovery ON plan(status, start_time) WHERE deleted_at IS NULL;

-- Category browsing
CREATE INDEX idx_plan_category ON plan(category_ref, start_time) WHERE deleted_at IS NULL;

-- Find plan locations that are active (map view)
CREATE INDEX idx_active_plan_location ON plan USING GIST (location_point) WHERE deleted_at IS NULL AND status = 'published';

-- QR code verification for attendee check in
CREATE UNIQUE INDEX idx_plan_qr_code ON plan(qr_code_hash) WHERE qr_code_hash IS NOT NULL;
```

**Rationale**
- `idx_plan_discovery` partial index: Finding plans that are published and starting soon that are also not deleted.
- `idx_plan_category` partial index: Find plans under a certain category that are not deleted.
- `idx_active_plan_location` GIST index: Find the geo location of an plan quickly for location searching for active plans.
- `idx_plan_qr_code` unique index: Ensure that the QR code for each plan is unique.

### plan_attendee
Table that is for users who are attending an plan.

**Columns**
- `ref` (PK): UUID for primary key
- `user_ref` (FK): Foreign key relating to the `user` attending the plan
- `plan_ref` (FK): Foreign key relating to the `plan` in attendance
- `status`: Attendee status "pending | confirmed | checked_in | no_show | cancelled"
- `joined_at`: Date and time the user joined the plan
- `checked_in_at`: Date and time the user checks in at
- `cancelled_at`: Date and time the user cancels attending plan
- `created_at`: Audit for when the user selects attending
- `updated_at`: Date and time for when a user updates their attendance status

**Indexes**
```sql
-- Primary key (auto-created)
CREATE UNIQUE INDEX plan_attendee_pk ON plan_attendee(ref);

-- Foreign key for relating to plan
CREATE INDEX idx_plan_ref_fk ON plan_attendee(plan_ref);

-- Composite Unique Constraint 
CREATE UNIQUE INDEX idx_user_plan_unique ON plan_attendee(plan_ref, user_ref);

-- View all users with a particular status for an plan
CREATE INDEX idx_user_status_for_plan ON plan_attendee(plan_ref, status);

-- View all statuses for user
CREATE INDEX idx_plan_attendee_user_status ON plan_attendee(user_ref, status);

-- Attendees pending check-in
CREATE INDEX idx_plan_attendee_pending_checkin ON plan_attendee(plan_ref, status) WHERE checked_in_at IS NULL AND status IN ('confirmed', 'pending');
```

**Rationale**
- `idx_user_plan_unique` composite unique index: Ensuring that only one user can only attend to a plan once.
- `idx_user_status_for_plan` index: Finding particular user statuses for a plan.
- `idx_plan_attendee_user_status` index: Finding all statuses for a user.
- `idx_plan_attendee_pending_checkin` partial index: Users who have joined by not checked in yet.

### plan_invitation
Users can send invites to other users to attend their plan, this table stores that invite information.

**Columns**
- `ref` (PK): UUID primary key
- `plan_ref` (FK): Foreign key for joining to a `plan`
- `inviter_ref` (FK): Foreign key for joining to `user` who is sending the invite
- `invitee_ref` (FK): Foreign ley for joining to `user` who is receiving the invite
- `status`: Current status of the invitation "pending | accepted | declined"
- `sent_at`: Date and time for when the invite was sent by the inviter
- `responded_at`: Date and time for when the invite was responded by the invitee

**Indexes**
```sql
-- Primary key (auto-created)
CREATE UNIQUE INDEX plan_invitation_pk ON plan_invitation(ref);

-- Composite unique constraint 
CREATE UNIQUE INDEX idx_user_plan_unique ON plan_invitation(plan_ref, inviter_ref, invitee_ref);

-- Invitee's invitations by status
CREATE INDEX idx_plan_invitation_invitee_status ON plan_invitation(invitee_ref, status, sent_at DESC);

-- Inviter's invitations by status
CREATE INDEX idx_plan_invitation_inviter_status ON plan_invitation(inviter_ref, status, sent_at DESC);

-- View the status of invitations to a plan
CREATE INDEX idx_plan_invite_statuses ON plan_invitation(plan_ref, status, sent_at);

-- Send follow up notifications to respond
CREATE INDEX idx_plan_invite_follow_up ON plan_invitation(sent_at, responded_at) WHERE responded_at IS NULL AND status = 'pending';
```

**Rationale**
- `idx_user_plan_unique` composite unique constraint: ensure that an plan can only have an invitation sent by a user to another user once.
- `idx_plan_invitation_invitee_status` index: Sort the invites sent out by a and sort by status.
- `idx_plan_invitation_inviter_status` index: Sort the invites received by a and sort by status.
- `idx_plan_invite_statuses` index: View the different statuses for a particular plan.
- `idx_plan_invite_follow_up` partial index: View the time since an invitation was sent and if it was responded to send a follow up notification.

### plan_image
Users can upload images for an plan to show what is going on.

**Columns**
- `ref` (PK): UUID primary key
- `plan_ref` (FK): Foreign key to join to the `plan` table
- `url`: URL for where the image is hosted
- `display_order`: The order in which the images are displayed on the frontend
- `is_cover`: If the image is the cover picture for the `plan`
- `created_at`: Audit log for creation

**Indexes**
```sql
-- Primary key (auto-created)
CREATE UNIQUE INDEX plan_image_pk ON plan_image(ref);

-- View in order the images for an plan
CREATE INDEX idx_plan_images_ordered ON plan_image(plan_ref, display_order);

-- View the cover image
CREATE INDEX idx_plan_image_coverr ON plan_image(plan_ref) WHERE is_cover = true;
```

**Rationale**
- `idx_plan_images_ordered` index: Order the images returned for a plan by the display order.
- `idx_plan_image_coverr` partial index: Return the cover image for the plan.

### chat_message
Users are able to communicate to each other for a specific plan.

**Columns**
- `ref` (PK): UUID primary key
- `user_ref` (FK): Foreign key for joining to the `user` table
- `plan_ref` (FK): Foreign key for joining to the `plan` table
- `content`: Content of the message
- `message_type`: Metadata about the type of message "text | image | system"
- `status`: Soft deletion of messages if moderation removes "visible | deleted"
- `created_at`: Able to view order of messages based on when it was created

**Indexes**
```sql
-- Primary key (auto-created)
CREATE UNIQUE INDEX chat_message_pk ON chat_message(ref);

-- Retrieve all chat messages for a plan order latest
CREATE INDEX idx_plan_chat_messages ON chat_message(plan_ref, created_at DESC) WHERE status = 'visible';
```

**Rationale**
- `idx_plan_chat_messages` partial index: Index to retrieve all chat messages in a plan and order by `created_at` to show by recency.

### category
Table which represents categories for the plans to fall under, plan can only have 1 category.

**Columns**
- `ref` (PK): UUID for the primary key
- `name` (UK): Unique category name
- `description`: Explains what the category is
- `icon_url`: URL for the icon

**Indexes**
```sql
-- Primary key (auto-created)
CREATE UNIQUE INDEX category_pk ON category(ref);
```

**Rationale**
Table will be very small with less than 20 categories, no indexing needed.

### interest
Table for interests which a user can say they have, to help with suggesting categories or finding friends with similar interests.

**Columns**
- `ref` (PK): UUID primary key
- `name` (UK): The unique name for the interest
- `description`: More information on the interest

**Indexes**
```sql
-- Primary key (auto-created)
CREATE UNIQUE INDEX interest_pk ON interest(ref);
```

**Rationale**
Will have less than 40 items in the interest table so don't need to index.

### user_interest
Joining table for the M:M relationship between `user` and `interest` as a user can have many interests.

**Columns**
- `user_ref` (PK) (FK): Composite key relating to the `user` table
- `interest_ref` (PK) (FK): Composite key relating to the `interest` table

**Indexes**
```sql
-- Primary key composite (auto-created)
CREATE UNIQUE INDEX user_interest_unique_pk ON user_interest(user_ref, interest_ref)
;
-- Users that selected an interest
CREATE INDEX idx_interest_users ON user_interest(interest_ref, user_ref);
```

**Rationale**
- `user_interest_unique_pk` primary key composite unique index: To find the interests that a particular user has.
- `idx_interest_users` index: A user can only have one version of the interest.
