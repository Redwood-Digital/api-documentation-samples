# User Management GraphQL API

A GraphQL API for managing users, roles, and permissions in your application.

## Endpoint

```
https://api.usermanager.com/graphql
```

## Authentication

Include your API key in the Authorization header:

```
Authorization: Bearer YOUR_API_KEY
```

## Schema Overview

The API provides queries for fetching data and mutations for modifying data.

## Types

### User

The User type represents a user account in the system.

```graphql
type User {
  id: ID!
  email: String!
  username: String!
  firstName: String
  lastName: String
  fullName: String
  avatar: String
  role: Role!
  permissions: [Permission!]!
  isActive: Boolean!
  emailVerified: Boolean!
  createdAt: DateTime!
  updatedAt: DateTime!
  lastLogin: DateTime
  teams: [Team!]!
}
```

### Role

```graphql
type Role {
  id: ID!
  name: String!
  description: String
  permissions: [Permission!]!
  userCount: Int!
  isSystem: Boolean!
  createdAt: DateTime!
}
```

### Permission

```graphql
type Permission {
  id: ID!
  name: String!
  resource: String!
  action: String!
  description: String
}
```

### Team

Teams group users together for collaborative work.

```graphql
type Team {
  id: ID!
  name: String!
  description: String
  members: [TeamMember!]!
  owner: User!
  createdAt: DateTime!
}

type TeamMember {
  user: User!
  role: TeamRole!
  joinedAt: DateTime!
}

enum TeamRole {
  OWNER
  ADMIN
  MEMBER
  VIEWER
}
```

## Queries

### Get Current User

Fetch the authenticated user's information.

```graphql
query GetCurrentUser {
  me {
    id
    email
    username
    fullName
    avatar
    role {
      name
      permissions {
        name
        resource
        action
      }
    }
    teams {
      id
      name
      role
    }
  }
}
```

### Get User by ID

```graphql
query GetUser($id: ID!) {
  user(id: $id) {
    id
    email
    username
    firstName
    lastName
    role {
      name
    }
    isActive
    lastLogin
  }
}
```

Variables:
```json
{
  "id": "usr_123456789"
}
```

### List Users

Get a paginated list of users with filtering options.

```graphql
query ListUsers(
  $first: Int
  $after: String
  $filter: UserFilter
  $orderBy: UserOrderBy
) {
  users(
    first: $first
    after: $after
    filter: $filter
    orderBy: $orderBy
  ) {
    edges {
      node {
        id
        email
        username
        fullName
        role {
          name
        }
        isActive
        createdAt
      }
      cursor
    }
    pageInfo {
      hasNextPage
      endCursor
      totalCount
    }
  }
}
```

## Mutations

### Create User

Create a new user account.

```graphql
mutation CreateUser($input: CreateUserInput!) {
  createUser(input: $input) {
    user {
      id
      email
      username
      role {
        name
      }
    }
    errors {
      field
      message
    }
  }
}
```

Variables:
```json
{
  "input": {
    "email": "newuser@example.com",
    "username": "newuser",
    "password": "SecurePassword123!",
    "firstName": "John",
    "lastName": "Doe",
    "roleId": "rol_default"
  }
}
```

### Update User

Update an existing user's information.

```graphql
mutation UpdateUser($id: ID!, $input: UpdateUserInput!) {
  updateUser(id: $id, input: $input) {
    user {
      id
      email
      firstName
      lastName
      updatedAt
    }
    errors {
      field
      message
    }
  }
}
```

### Assign Role

Assign a role to a user.

```graphql
mutation AssignRole($userId: ID!, $roleId: ID!) {
  assignRole(userId: $userId, roleId: $roleId) {
    user {
      id
      role {
        name
        permissions {
          name
        }
      }
    }
    errors {
      message
    }
  }
}
```

## Input Types

### UserFilter

Filter options for user queries.

```graphql
input UserFilter {
  isActive: Boolean
  emailVerified: Boolean
  role: String
  teamId: ID
  createdAfter: DateTime
  createdBefore: DateTime
}
```

### UserOrderBy

Sorting options for user lists.

```graphql
input UserOrderBy {
  field: UserOrderField!
  direction: OrderDirection!
}

enum UserOrderField {
  CREATED_AT
  UPDATED_AT
  EMAIL
  USERNAME
  LAST_LOGIN
}

enum OrderDirection {
  ASC
  DESC
}
```

## Error Handling

All mutations return an errors field with detailed error information.

```graphql
type UserError {
  field: String
  message: String!
  code: String
}
```

Example error response:
```json
{
  "data": {
    "createUser": {
      "user": null,
      "errors": [
        {
          "field": "email",
          "message": "Email already exists",
          "code": "DUPLICATE_EMAIL"
        }
      ]
    }
  }
}
```

## Pagination

The API uses cursor-based pagination for list queries.

```graphql
type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
  totalCount: Int
}
```

## Rate Limiting

- Anonymous: 100 requests per hour
- Authenticated: 1000 requests per hour
- Premium: 10000 requests per hour

Rate limit information is included in response headers:
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 998
X-RateLimit-Reset: 1719436800
```

## Example Client Code

### JavaScript (Apollo Client)

```javascript
import { ApolloClient, InMemoryCache, gql } from '@apollo/client';

const client = new ApolloClient({
  uri: 'https://api.usermanager.com/graphql',
  headers: {
    authorization: `Bearer ${YOUR_API_KEY}`
  },
  cache: new InMemoryCache()
});

// Query example
const GET_USERS = gql`
  query GetUsers($first: Int!) {
    users(first: $first) {
      edges {
        node {
          id
          email
          username
        }
      }
    }
  }
`;

client.query({
  query: GET_USERS,
  variables: { first: 10 }
}).then(result => {
  console.log(result.data.users.edges);
});
```

## GraphQL Playground

You can explore the API using our GraphQL Playground:
https://api.usermanager.com/playground

The playground provides:
- Interactive query builder
- Schema documentation
- Query history

---

*API Version: 2.0 | GraphQL Schema Version: 2025-01-08*
