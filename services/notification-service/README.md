# Notification Service

Async notification delivery service with email, in-app, and webhook channels for the issue tracker platform.

## Service Structure

```
src/
├── channels/
│   ├── email.py           # Email delivery channel
│   ├── in_app.py          # In-app notification channel
│   ├── webhook.py         # Webhook delivery channel
│   └── __init__.py        # Package exports
├── consumers/
│   ├── comment_events.py  # Comment event consumer
│   ├── issue_events.py    # Issue event consumer
│   └── __init__.py        # Package exports
├── models/
│   ├── NotificationModel  # Notification database model
│   ├── PreferenceModel    # User preference model
│   ├── ChannelModel       # Delivery channel model
│   └── __init__.py        # Package exports
├── repositories/
│   ├── NotificationRepository # Notification CRUD
│   ├── PreferenceRepository   # Preference CRUD
│   ├── ChannelRepository      # Channel CRUD
│   └── __init__.py            # Package exports
├── routers/
│   ├── notification_router.py # Notification API routes
│   └── __init__.py            # Package exports
├── services/
│   ├── NotificationService    # Notification business logic
│   ├── DeliveryService        # Multi-channel delivery
│   ├── PreferenceService      # Preference business logic
│   ├── TemplateService        # Template rendering
│   └── __init__.py            # Package exports
├── database.py            # Database connection
├── main.py                # FastAPI application entrypoint
└── __init__.py            # Package exports
```

## Features

### Delivery Channels

#### Email
- **Templates**: Jinja2 templates with variables
- **SMTP**: Configurable SMTP settings
- **Retry Logic**: Exponential backoff retry
- **Tracking**: Open/click tracking
- **Unsubscribe**: One-click unsubscribe

#### In-App
- **Real-time**: WebSocket delivery
- **Persistence**: Stored in database
- **Grouping**: Bundle similar notifications
- **Mark Read**: Individual/bulk mark read

#### Webhooks
- **Custom URLs**: Per-user/webhook endpoints
- **Signatures**: HMAC signature verification
- **Retry**: Configurable retry with backoff
- **Filtering**: Event-type filtering

### Notification Types
- **Issue Events**: Created, updated, assigned, status changed, commented
- **Comment Events**: New comment, mention, reply
- **Project Events**: Added to project, role changed
- **System Events**: Announcements, maintenance
- **Custom Events**: User-defined triggers

### Preferences
- **Per-Channel**: Enable/disable per channel
- **Per-Event**: Granular event subscriptions
- **Quiet Hours**: Do-not-disturb schedule
- **Digest**: Daily/weekly summary emails
- **Priority**: Urgent vs normal delivery

### Celery Integration
- **Async Delivery**: Background task processing
- **Task Routing**: Channel-specific queues
- **Monitoring**: Flower integration
- **Scaling**: Horizontal worker scaling

## API Endpoints

### Notifications

#### Get Notifications
```
GET /api/v1/notifications?skip=0&limit=20&unread_only=false

Query Parameters:
- skip: int (default: 0)
- limit: int (default: 20, max: 100)
- unread_only: bool (default: false)
- channel: string (email|in_app|webhook)
```

#### Get Notification
```
GET /api/v1/notifications/{notification_id}

Response:
{
  "id": "uuid",
  "user_id": "uuid",
  "type": "issue.assigned",
  "title": "Issue assigned to you",
  "message": "You have been assigned to issue WEB-123",
  "data": {
    "issue_id": "uuid",
    "issue_key": "WEB-123",
    "project_id": "uuid"
  },
  "channels": ["in_app", "email"],
  "is_read": false,
  "read_at": null,
  "created_at": "2024-01-15T10:30:00Z"
}
```

#### Mark as Read
```
POST /api/v1/notifications/{notification_id}/read
```

#### Mark All as Read
```
POST /api/v1/notifications/read-all
```

#### Delete Notification
```
DELETE /api/v1/notifications/{notification_id}
```

### Preferences

#### Get Preferences
```
GET /api/v1/users/{user_id}/preferences

Response:
{
  "user_id": "uuid",
  "email_enabled": true,
  "in_app_enabled": true,
  "webhook_enabled": false,
  "webhook_url": "https://...",
  "quiet_hours_start": "22:00",
  "quiet_hours_end": "08:00",
  "timezone": "UTC",
  "digest_frequency": "daily",
  "event_preferences": {
    "issue.created": {"email": true, "in_app": true},
    "issue.assigned": {"email": true, "in_app": true, "webhook": false},
    "comment.created": {"email": false, "in_app": true},
    "mention.created": {"email": true, "in_app": true}
  }
}
```

#### Update Preferences
```
PATCH /api/v1/users/{user_id}/preferences
Content-Type: application/json

{
  "email_enabled": false,
  "quiet_hours_start": "23:00",
  "event_preferences": {
    "issue.created": {"email": false, "in_app": true}
  }
}
```

### Channels

#### Register Webhook
```
POST /api/v1/users/{user_id}/webhooks
Content-Type: application/json

{
  "url": "https://myapp.com/webhook",
  "events": ["issue.created", "issue.assigned"],
  "secret": "webhook-secret"
}
```

#### Test Webhook
```
POST /api/v1/users/{user_id}/webhooks/{webhook_id}/test
```

## Configuration

Environment variables (in `.env`):
```env
# Service
SERVICE_NAME=notification-service
SERVICE_VERSION=1.0.0
ENVIRONMENT=development

# Database
DATABASE_URL=postgresql+asyncpg://postgres:postgres_password@postgres:5432/issue_tracker

# Redis
REDIS_URL=redis://redis:6379/0

# RabbitMQ
RABBITMQ_URL=amqp://guest:guest@rabbitmq:5672/

# Celery
CELERY_BROKER_URL=redis://redis:6379/1
CELERY_RESULT_BACKEND=redis://redis:6379/2

# Email (SMTP)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=noreply@example.com
SMTP_PASSWORD=your-password
SMTP_TLS=true
EMAIL_FROM=noreply@example.com
EMAIL_FROM_NAME=Issue Tracker

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
  "service": "notification-service",
  "version": "1.0.0"
}
```

## Development

### Running the Service
```bash
pip install -r requirements.txt

# Run API server
uvicorn src.main:app --reload --host 0.0.0.0 --port 8000

# Run Celery worker (separate terminal)
celery -A src.main.celery_app worker -l info

# Or using Docker
docker build -f Dockerfile -t notification-service .
docker run -p 8006:8000 --env-file .env notification-service
```

### Testing
```bash
pytest tests/
pytest --cov=src tests/
```