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
- `POST` to a specific resource performs actions

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