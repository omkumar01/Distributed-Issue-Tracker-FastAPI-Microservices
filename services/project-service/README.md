# Project Service

Project lifecycle and team membership management service for the issue tracker platform.

## Service Structure

```
src/
├── api/
│   ├── projects.py        # Project endpoints
│   ├── members.py         # Member endpoints
│   ├── permissions.py     # Permission endpoints
│   └── __init__.py        # Package exports
├── db/
│   ├── database.py        # Database connection
│   └── __init__.py        # Package exports
├── domain/
│   ├── project.py         # Project domain model
│   ├── membership.py      # Membership domain model
│   └── __init__.py        # Package exports
├── events/
│   ├── publish.py         # Event publishers
│   └── __init__.py        # Package exports
├── models/
│   ├── ProjectModel       # Project database model
│   ├── MembershipModel    # Membership database model
│   ├── ProjectSettingsModel # Project settings model
│   └── __init__.py        # Package exports
├── repositories/
│   ├── ProjectRepository  # Project CRUD operations
│   ├── MembershipRepository # Membership CRUD
│   ├── SettingsRepository # Settings CRUD
│   └── __init__.py        # Package exports
├── routers/
│   ├── project_router.py  # Project API routes
│   └── __init__.py        # Package exports
├── services/
│   ├── ProjectService     # Project business logic
│   ├── MembershipService  # Membership business logic
│   ├── PermissionService  # Permission business logic
│   └── __init__.py        # Package exports
├── database.py            # Database utilities
├── main.py                # FastAPI application entrypoint
└── __init__.py            # Package exports
```

## Features

### Project Management
- **Project CRUD**: Create, read, update, delete projects
- **Project Settings**: Customizable project configurations
- **Project Archiving**: Archive/restore projects
- **Project Templates**: Create projects from templates

### Team Membership
- **Member Management**: Add/remove team members
- **Role Assignment**: Assign roles within projects
- **Permission Matrix**: Fine-grained permissions per role
- **Invitation System**: Invite users to projects

### Event Integration
- **Project Events**: Publishes project.created, project.updated, project.deleted, project.archived
- **Membership Events**: Publishes member.added, member.removed, member.role_changed
- **Consumes Events**: Listens for user and auth events

## API Endpoints

### Projects

#### Create Project
```
POST /api/v1/projects
Content-Type: application/json

{
  "name": "Website Redesign",
  "description": "Redesign company website",
  "key": "WEB",
  "owner_id": "uuid",
  "settings": {
    "issue_types": ["bug", "feature", "task"],
    "workflow": "default"
  }
}
```

#### Get Project
```
GET /api/v1/projects/{project_id}

Response:
{
  "id": "uuid",
  "name": "Website Redesign",
  "description": "Redesign company website",
  "key": "WEB",
  "owner_id": "uuid",
  "is_archived": false,
  "settings": {...},
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z"
}
```

#### Update Project
```
PATCH /api/v1/projects/{project_id}
Content-Type: application/json

{
  "name": "Website Redesign v2",
  "description": "Updated description"
}
```

#### Archive Project
```
POST /api/v1/projects/{project_id}/archive
```

#### Restore Project
```
POST /api/v1/projects/{project_id}/restore
```

#### Delete Project
```
DELETE /api/v1/projects/{project_id}
```

#### List Projects
```
GET /api/v1/projects?skip=0&limit=20&archived=false

Query Parameters:
- skip: int (default: 0)
- limit: int (default: 20, max: 100)
- archived: bool (default: false)
- owner_id: uuid (optional)
```

### Members

#### Add Member
```
POST /api/v1/projects/{project_id}/members
Content-Type: application/json

{
  "user_id": "uuid",
  "role": "developer"
}
```

#### Update Member Role
```
PATCH /api/v1/projects/{project_id}/members/{user_id}
Content-Type: application/json

{
  "role": "admin"
}
```

#### Remove Member
```
DELETE /api/v1/projects/{project_id}/members/{user_id}
```

#### List Members
```
GET /api/v1/projects/{project_id}/members
```

### Permissions

#### Get Project Permissions
```
GET /api/v1/projects/{project_id}/permissions
```

#### Check User Permission
```
GET /api/v1/projects/{project_id}/permissions/check?user_id={user_id}&permission=issues.write
```

## Configuration

Environment variables (in `.env`):
```env
# Service
SERVICE_NAME=project-service
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
  "service": "project-service",
  "version": "1.0.0"
}
```

## Development

### Running the Service
```bash
pip install -r requirements.txt
uvicorn src.main:app --reload --host 0.0.0.0 --port 8000

# Or using Docker
docker build -f Dockerfile -t project-service .
docker run -p 8003:8000 --env-file .env project-service
```

### Testing
```bash
pytest tests/
pytest --cov=src tests/
```