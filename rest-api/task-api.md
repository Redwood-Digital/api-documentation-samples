# Task Management API

A RESTful API for managing tasks, projects, and team collaboration.

## Base URL
```
https://api.taskmanager.io/v1
```

## Authentication

The API uses Bearer token authentication. Include your API token in all requests:

```
Authorization: Bearer YOUR_API_TOKEN
```

### Getting a Token

```bash
POST /auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "your_password"
}
```

Response:
```json
{
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "expires_at": "2024-07-01T00:00:00Z",
  "user": {
    "id": "usr_123456",
    "email": "user@example.com",
    "name": "John Doe"
  }
}
```

## Resources

### Tasks

#### List Tasks

Get all tasks for the authenticated user.

```
GET /tasks
```

Query parameters:
- `status` - Filter by status: `pending`, `in_progress`, `completed`
- `project_id` - Filter by project
- `assigned_to` - Filter by assignee user ID
- `page` - Page number (default: 1)
- `limit` - Items per page (default: 20, max: 100)

Example:
```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  "https://api.taskmanager.io/v1/tasks?status=pending&limit=10"
```

Response:
```json
{
  "tasks": [
    {
      "id": "tsk_abc123",
      "title": "Update API documentation",
      "description": "Add new endpoints to the docs",
      "status": "pending",
      "priority": "high",
      "project_id": "prj_xyz789",
      "assigned_to": "usr_123456",
      "due_date": "2024-06-30",
      "created_at": "2024-06-25T10:00:00Z",
      "updated_at": "2024-06-25T10:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 42,
    "pages": 5
  }
}
```

#### Create Task

Create a new task.

```
POST /tasks
```

Request body:
```json
{
  "title": "Review pull request",
  "description": "Review and merge the new feature branch",
  "project_id": "prj_xyz789",
  "assigned_to": "usr_654321",
  "prioritty": "medium",
  "due_date": "2024-06-28"
}
```

Response:
```json
{
  "id": "tsk_def456",
  "title": "Review pull request",
  "description": "Review and merge the new feature branch",
  "status": "pending",
  "priority": "medium",
  "project_id": "prj_xyz789",
  "assigned_to": "usr_654321",
  "due_date": "2024-06-28",
  "created_at": "2024-06-26T14:30:00Z",
  "updated_at": "2024-06-26T14:30:00Z"
}
```

#### Update Task

Update an existing task.

```
PATCH /tasks/{task_id}
```

Request body (all fields optional):
```json
{
  "title": "Updated title",
  "status": "in_progress",
  "priority": "high"
}
```

#### Delete Task

Delete a task.

```
DELETE /tasks/{task_id}
```

Response:
```json
{
  "message": "Task deleted successfully"
}
```

### Projects

#### List Projects

Get all projects accessible to the user.

```
GET /projects
```

Response:
```json
{
  "projects": [
    {
      "id": "prj_xyz789",
      "name": "Website Redesign",
      "description": "Complete overhaul of company website",
      "color": "#3B82F6",
      "owner_id": "usr_123456",
      "team_ids": ["usr_123456", "usr_654321"],
      "created_at": "2024-06-01T00:00:00Z"
    }
  ]
}
```

#### Create Project

Create a new project.

```
POST /projects
```

Request body:
```json
{
  "name": "Mobile App Development",
  "description": "Build iOS and Android apps",
  "color": "#10B981",
  "team_ids": ["usr_123456", "usr_654321", "usr_789012"]
}
```

### Comments

#### Add Comment

Add a comment to a task.

```
POST /tasks/{task_id}/comments
```

Request body:
```json
{
  "content": "I've started working on this. Should be done by tomorrow."
}
```

Response:
```json
{
  "id": "cmt_123abc",
  "task_id": "tsk_abc123",
  "user_id": "usr_123456",
  "content": "I've started working on this. Should be done by tomorrow.",
  "created_at": "2024-06-26T15:00:00Z"
}
```

#### List Comments

Get all comments for a task.

```
GET /tasks/{task_id}/comments
```

## Webhooks

Configure webhooks to receive real-time updates.

### Create Webhook

```
POST /webhooks
```

Request body:
```json
{
  "url": "https://your-app.com/webhook",
  "events": ["task.created", "task.updated", "task.completed"],
  "secret": "your_webhook_secret"
}
```

### Webhook Events

- `task.created` - New task created
- `task.updated` - Task details updated
- `task.completed` - Task marked as completed
- `task.deleted` - Task deleted
- `comment.added` - Comment added to task

## Error Handling

The API uses standard HTTP status codes and returns errors in a consistent format.

### Error Response Format

```json
{
  "error": {
    "type": "validation_error",
    "message": "Invalid request data",
    "details": {
      "title": "Title is required",
      "due_date": "Due date must be in the future"
    }
  }
}
```

### Common Errors

| Status | Type | Description |
|--------|------|-------------|
| 400 | validation_error | Invalid request data |
| 401 | authentication_error | Missing or invalid token |
| 403 | permission_error | Insufficient permissions |
| 404 | not_found | Resource not found |
| 429 | rate_limit_error | Too many requests |
| 500 | server_error | Internal server error |

## Rate Limiting

API requests are limited to:
- 1000 requests per hour for standard accounts
- 5000 requests per hour for premium accounts

Rate limit status is included in response headers:
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 950
X-RateLimit-Reset: 1719435600
```

## Pagination

List endpoints support pagination using `page` and `limit` parameters.

Example:
```bash
GET /tasks?page=2&limit=25
```

Pagination info is included in responses:
```json
{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 25,
    "total": 150,
    "pages": 6,
    "has_next": true,
    "has_prev": true
  }
}
```

## Code Examples

### Node.js
```javascript
const fetch = require('node-fetch');

const API_TOKEN = 'your_api_token';
const BASE_URL = 'https://api.taskmanager.io/v1';

async function createTask(taskData) {
  const response = await fetch(`${BASE_URL}/tasks`, {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${API_TOKEN}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(taskData)
  });

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.error.message);
  }

  return response.json();
}

// Usage
createTask({
  title: 'New task',
  project_id: 'prj_xyz789',
  priority: 'high'
}).then(task => {
  console.log('Created task:', task.id);
});
```

### Python
```python
import requests

API_TOKEN = 'your_api_token'
BASE_URL = 'https://api.taskmanager.io/v1'

headers = {
    'Authorization': f'Bearer {API_TOKEN}',
    'Content-Type': 'application/json'
}

def get_tasks(status=None, project_id=None):
    params = {}
    if status:
        params['status'] = status
    if project_id:
        params['project_id'] = project_id
    
    response = requests.get(f'{BASE_URL}/tasks', 
                          headers=headers, 
                          params=params)
    response.raise_for_status()
    return response.json()

# Get all pending tasks
pending_tasks = get_tasks(status='pending')
print(f"Found {len(pending_tasks['tasks'])} pending tasks")
```

## SDKs

Official SDKs:
- JavaScript/TypeScript: `npm install @taskmanager/sdk`
- Python: `pip install taskmanager-sdk`
- Go: `go get github.com/taskmanager/go-sdk`
- Ruby: `gem install taskmanager`

---

*API Version: 1.0 | Last updated: November 2024*
