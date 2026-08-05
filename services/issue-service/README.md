# Issue Service

Core issue tracking service with workflow states, assignments, and SLA management for the issue tracker platform.

## Service Structure

```
src/
├── core/
│   ├── config.py          # Configuration settings
│   └── __init__.py        # Package exports
├── domain/
│   ├── issue.py           # Issue domain model
│   ├── sla.py             # SLA domain model
│   ├── workflow.py        # Workflow state machine
│   └── __init__.py        # Package exports
├── events/
│   ├── publisher.py       # Event publishers
│   └── __init__.py        # Package exports
├── models/
│   ├── IssueModel         # Issue database model
│   ├── SLAModel           # SLA database model
│   ├── WorkflowModel      # Workflow definition model
│   └── __init__.py        # Package exports
├── repositories/
│   ├── IssueRepository    # Issue CRUD operations
│   ├── SLARepository      # SLA CRUD operations
│   └── __init__.py        # Package exports
├── routers/
│   ├── issue_router.py    # Issue API routes
│   ├── workflow_router.py # Workflow API routes
│   └── __init__.py        # Package exports
├── services/
│   ├── IssueService       # Issue business logic
│   ├── SLAService         # SLA business logic
│   ├── WorkflowService    # Workflow business logic
│   └── __init__.py        # Package exports
├── database.py            # Database connection
├── main.py                # FastAPI application entrypoint
└── __init__.py            # Package exports
```

## Features

### Issue Management
- **Issue CRUD**: Create, read, update, delete issues
- **Issue Types**: Bug, Feature, Task, Epic, Story
- **Priority Levels**: Critical, High, Medium, Low
- **Status Workflow**: Customizable state transitions
- **Assignments**: Assign issues to users/teams
- **Labels/Tags**: Categorize issues
- **Attachments**: File attachments support
- **Comments**: Threaded discussions
- **Watchers**: Subscribe to issue updates

### Workflow Engine
- **State Machine**: Configurable workflow states
- **Transitions**: Valid state transitions with conditions
- **Automation**: Auto-transition on events
- **Custom Workflows**: Per-project workflows

### SLA Management
- **SLA Policies**: Define response/resolution targets
- **SLA Tracking**: Monitor SLA compliance
- **Escalation**: Auto-escalate breached SLAs
- **Business Hours**: Configure working hours
- **Holiday Calendar**: Exclude holidays from SLA

### Event Integration
- **Issue Events**: Publishes issue.created, issue.updated, issue.deleted, issue.status_changed, issue.assigned
- **SLA Events**: Publishes sla.breached, sla.warning
- **Consumes Events**: Listens for project, user, comment events

## API Endpoints

### Issues

#### Create Issue
```
POST /api/v1/projects/{project_id}/issues
Content-Type: application/json

{
  "title": "Login page not loading",
  "description": "Users cannot access login page",
  "issue_type": "bug",
  "priority": "high",
  "assignee_id": "uuid",
  "labels": ["frontend", "urgent"],
  "due_date": "2024-01-20T10:00:00Z"
}
```

#### Get Issue
```
GET /api/v1/issues/{issue_id}

Response:
{
  "id": "uuid",
  "project_id": "uuid",
  "title": "Login page not loading",
  "description": "Users cannot access login page",
  "issue_type": "bug",
  "priority": "high",
  "status": "open",
  "assignee_id": "uuid",
  "reporter_id": "uuid",
  "labels": ["frontend", "urgent"],
  "due_date": "2024-01-20T10:00:00Z",
  "workflow_state": "open",
  "sla_deadline": "2024-01-18T10:00:00Z",
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z"
}
```

#### Update Issue
```
PATCH /api/v1/issues/{issue_id}
Content-Type: application/json

{
  "status": "in_progress",
  "assignee_id": "uuid",
  "priority": "critical"
}
```

#### Delete Issue
```
DELETE /api/v1/issues/{issue_id}
```

#### List Issues
```
GET /api/v1/issues?project_id={project_id}&status=open&priority=high&skip=0&limit=20

Query Parameters:
- project_id: uuid (required)
- status: string (optional)
- priority: string (optional)
- assignee_id: uuid (optional)
- issue_type: string (optional)
- labels: string (comma-separated)
- skip: int (default: 0)
- limit: int (default: 20, max: 100)
- sort: string (default: -created_at)
```

### Workflow

#### Get Workflow
```
GET /api/v1/projects/{project_id}/workflow
```

#### Update Workflow
```
PUT /api/v1/projects/{project_id}/workflow
Content-Type: application/json

{
  "states": [
    {"name": "open", "type": "start"},
    {"name": "in_progress", "type": "intermediate"},
    {"name": "in_review", "type": "intermediate"},
    {"name": "done", "type": "end"}
  ],
  "transitions": [
    {"from": "open", "to": "in_progress"},
    {"from": "in_progress", "to": "in_review"},
    {"from": "in_review", "to": "done"},
    {"from": "in_review", "to": "in_progress"}
  ]
}
```

#### Transition Issue
```
POST /api/v1/issues/{issue_id}/transition
Content-Type: application/json

{
  "to_state": "in_progress"
}
```

### SLA

#### Create SLA Policy
```
POST /api/v1/projects/{project_id}/sla
Content-Type: application/json

{
  "name": "Critical Bug SLA",
  "issue_type": "bug",
  "priority": "critical",
  "response_time_hours": 1,
  "resolution_time_hours": 4,
  "business_hours": {
    "timezone": "UTC",
    "monday": {"start": "09:00", "end": "17:00"},
    ...
  }
}
```

#### Get SLA Status
```
GET /api/v1/issues/{issue_id}/sla
```

## Configuration

Environment variables (in `.env`):
```env
# Service
SERVICE_NAME=issue-service
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
  "service": "issue-service",
  "version": "1.0.0"
}
```

## Development

### Running the Service
```bash
pip install -r requirements.txt
uvicorn src.main:app --reload --host 0.0.0.0 --port 8000

# Or using Docker
docker build -f Dockerfile -t issue-service .
docker run -p 8004:8000 --env-file .env issue-service
```

### Testing
```bash
pytest tests/
pytest --cov=src tests/
```