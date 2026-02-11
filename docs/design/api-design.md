# API Design

## Overview
Establishing rules and patterns that the overall project will adhere to ensuring consistency.

## RESTful Conventions
### Resource Naming
All resources use **plural nouns** in URLs, even for single-item operations.

**Correct:**
- `/users` - Collection of users
- `/users/:id` - Single user
- `/plans` - Collection of plans
- `/plans/:id/attendees` - Nested collection

**Incorrect:**
- `/user` - Don't use singular
- `/getUser/:id` - Don't use verbs in resource names
- `/users/:id/getPlans` - Don't use verbs in paths

### HTTP Verbs
Use the standard HTTP verbs for indicating the action being made:

| Verb | Purpose | Example |
|------|---------|---------|
| `GET` | Retrieve resource(s) | `GET /plans` - List all plans |
| `POST` | Create new resource | `POST /plans` - Create a plan |
| `PATCH` | Partial update | `PATCH /plans/:id` - Update plan fields |
| `PUT` | Full replacement | `PUT /plans/:id` - Replace entire plan |
| `DELETE` | Remove resource | `DELETE /plans/:id` - Delete a plan |

**Best Practices:**
- Use `PATCH` for partial updates
- Use `PUT` only when replacing the entire resource
- `POST` to collections creates new items
- `POST` to a specific resource performs actions (see Custom Actions below)

### URL Structure
**Pattern:** `/{resourece}/{id}/{sub-resource}/{id}`

**Examples**
```http
# Users
POST /users # Create new user
GET /users/:id # Get user profile
PATCH /users/:id # Update part of user profile
DELETE /users/:id # Delete user profile

# Nested Resource (Plan attendees)
GET /plans/:id/attendees # List plan attendees
POST /plans/:id/attendees # Join a plan
DELETE /plans/:id/attendees/:userId # Remove an attendee

# Notifications
GET /users/:id/notifications # Get all notifications for user
PATCH /users/:id/notifications/:notificationId/read # Mark notification as read
```

### Custom Actions
Actions that don't fit the regular CRUD operations. Use `POST` to a descriptive endpoint:
**Pattern** `POST /{resource}/{id}/{action}`

**Examples**
```http
# Plan actions
POST /plans/:id/join # Join a plan
POST /plans/:id/check-in # QR code check in

# User actions
POST /users/:id/block # Block a user
POST /users/:id/follow # Follow a user

# Subscriptions
POST /subscriptions/portal # Get customer portal URL
```

**Rationale:**
- These are **RPC-style** operations that don't map cleanly to resource updates as RPC is more action focused, rather than entity oriented
- Using `POST` makes it clear these are actions, not just data updates
- Action names use verbs (join, leave, block) since they're operations, not resources

### Query Parameters
Use query parameters for filtering, sorting and pagination:

**Filtering**
```http
GET /plans?interests=bouldering,coffee # Names of activities
GET /plans?distance=5&lat=34.05&lng=-118.24 # Location based filtering
GET /plans?startTimeAfter=2024-02-15T00:00:00z # Time based filtering
GET /plans?status=visible # Visibility based filtering
```

**Sorting**
```http
GET /plans?sort=startTime # Default ascending
GET /plans?sort=-startTime # Descending using minus prefix
GET /plans?sort=distance,startTime # Multiple fields
```

**Pagination**
```http
GET /plans?limit=20&cursor=djnASdNJJN # Cursor-based (preference for feeds)
GET /reports?limit=50&offset=100 # Offset based
```

**Search**
```http
GET /plans?q=coffee # Free text search in title/description
```