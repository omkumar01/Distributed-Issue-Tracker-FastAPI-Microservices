# User Service

User profile, team, and preference management service for the issue tracker platform.

## Service Structure

```
src/
├── api/
│   ├── users.py           # User endpoints
│   ├── teams.py           # Team endpoints
│   ├── roles.py           # Role endpoints
│   └── __init__.py        # Package exports
├── db/
│   ├── database.py        # Database connection
│   └── __init__.py        # Package exports
├── domain/
│   ├── user.py            # User domain model
│   ├── role.py            # Role domain model
│   └── __init__.py        # Package exports
├── events/
│   ├── consume.py         # Event consumers
│   ├── publish.py         # Event publishers
│   └── __init__.py        # Package exports
├── models/
│   ├── UserModel          # User database model
│   ├── TeamModel          # Team database model
│   ├── RoleModel          # Role database model
│   ├── MembershipModel    # Team membership model
│   └── __init__.py        # Package exports
├── repositories/
│   ├── UserRepository     # User CRUD operations
│   ├── TeamRepository     # Team CRUD operations
│   ├── RoleRepository     # Role CRUD operations
│   ├── MembershipRepository # Membership CRUD
│   └── __init__.py        # Package exports
├── routers/
│   ├── user_router.py     # User API routes
│   └── __init__.py        # Package exports
├── services/
│   ├── UserService        # User business logic
│   ├── TeamService        # Team business logic
│   ├── RoleService        # Role business logic
│   └── __init__.py        # Package exports
├── database.py            # Database utilities
├── main.py                # FastAPI application entrypoint
└── __init__.py            # Package exports
```

## Features

### User Management
- **User Profiles**: Create, read, update, delete user profiles
- **User Preferences**: Customizable user settings
- **Avatar Support**: Profile image management
- **Status Tracking**: Active/inactive user status

### Team Management
- **Team Creation**: Create and manage teams
- **Team Membership**: Add/remove members with roles
- **Team Hierarchy**: Nested team support
- **Team Settings**: Team-level configuration

### Role-Based Access Control
- **Role Definitions**: Define custom roles with permissions
- **Role Assignment**: Assign roles to users in teams
- **Permission Checking**: Verify user permissions
- **Default Roles**: Predefined roles (admin, member, viewer)

### Event Integration
- **User Events**: Publishes user.created, user.updated, user.deleted
- **Team Events**: Publishes team.created, team.updated, team.deleted
- **Membership Events**: Publishes membership changes
- **Consumes Events**: Listens for auth and project events

## API Endpoints

### Users

#### Create User
```
POST /api/v1/users
Content-Type: application/json

{
  "email": "user@example.com",
  "username": "john_doe",
  "first_name": "John",
  "last_name": "Doe",
  "preferences": {
    "theme": "dark",
    "notifications": true
  }
}
```

#### Get User
```
GET /api/v1/users/{user_id}

Response:
{
  "id": "uuid",
  "email": "user@example.com",
  "username": "john_doe",
  "first_name": "John",
  "last_name": "Doe",
  "is_active": true,
  "preferences": {...},
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z"
}
```

#### Update User
```
PATCH /api/v1/users/{user_id}
Content-Type: application/json

{
  "first_name": "Jane",
  "preferences": {
    "theme": "light"
  }
}
```

#### Delete User
```
DELETE /api/v1/users/{user_id}
```

#### List Users
```
GET /api/v1/users?skip=0&limit=20&search=john

Query Parameters:
- skip: int (default: 0)
- limit: int (default: 20, max: 100)
- search: string (optional, searches email/username/name)
```

### Teams

#### Create Team
```
POST /api/v1/teams
Content-Type: application/json

{
  "name": "Engineering",
  "description": "Engineering team",
  "avatar_url": "https://..."
}
```

#### Get Team
```
GET /api/v1/teams/{team_id}
```

#### Update Team
```
PATCH /api/v1/teams/{team_id}
```

#### Delete Team
```
DELETE /api/v1/teams/{team_id}
```

#### List Teams
```
GET /api/v1/teams?skip=0&limit=20
```

### Team Membership

#### Add Member
```
POST /api/v1/teams/{team_id}/members
Content-Type: application/json

{
  "user_id": "uuid",
  "role_id": "uuid"
}
```

#### Update Member Role
```
PATCH /api/v1/teams/{team_id}/members/{user_id}
Content-Type: application/json

{
  "role_id": "uuid"
}
```

#### Remove Member
```
DELETE /api/v1/teams/{team_id}/members/{user_id}
```

#### List Members
```
GET /api/v1/teams/{team_id}/members
```

### Roles

#### Create Role
```
POST /api/v1/roles
Content-Type: application/json

{
  "name": "developer",
  "description": "Developer role",
  "permissions": ["issues.read", "issues.write", "comments.write"]
}
```

#### Get Role
```
GET /api/v1/roles/{role_id}
```

#### List Roles
```
GET /api/v1/roles
```

## Configuration

Environment variables (in `.env`):
```env
# Service
SERVICE_NAME=user-service
SERVICE_VERSION=1.0.0
ENVIRONMENT=development

# Database
DATABASE_URL=postgresql+asyncpg://postgres:postgres_password@postgres:5432/issue_tracker

# Redis
REDIS_URL=redis://redis:6379/0

# RabbitMQ
RABBITMQ_URL=amqp://guest:guest@rabbitmq:5672/

# Observability
JAEGER_ENABLED=false
JAEGER_HOST=jaeger
JAEGER_PORT=6831
```

## Health Check
```
GET /health

Response:
{
  "status": "healthy",
  "service": "user-service",
  "version": "1.0.0"
}
```

## Development

### Running the Service
```bash
# Install dependencies
pip install -r requirements.txt

# Run with uvicorn
uvicorn src.main:app --reload --host 0.0.0.0 --port 8000

# Or using Docker
docker build -f Dockerfile -t user-service .
docker run -p 8002:8000 --env-file .env user-service
```

### Testing
```bash
pytest tests/
pytest --cov=src tests/
```