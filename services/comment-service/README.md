# Comment Service

Comments, mentions, and edit history service for the issue tracker platform.

## Service Structure

```
src/
├── core/
│   ├── config.py          # Configuration settings
│   └── __init__.py        # Package exports
├── events/
│   ├── publisher.py       # Event publishers
│   └── __init__.py        # Package exports
├── models/
│   ├── CommentModel       # Comment database model
│   ├── MentionModel       # Mention database model
│   ├── CommentHistoryModel # Edit history model
│   └── __init__.py        # Package exports
├── repositories/
│   ├── CommentRepository  # Comment CRUD operations
│   ├── MentionRepository  # Mention CRUD operations
│   ├── HistoryRepository  # History CRUD operations
│   └── __init__.py        # Package exports
├── routers/
│   ├── comment_router.py  # Comment API routes
│   ├── realtime.py        # Real-time WebSocket routes
│   └── __init__.py        # Package exports
├── schemas/
│   ├── comment.py         # Comment Pydantic schemas
│   └── __init__.py        # Package exports
├── services/
│   ├── CommentService     # Comment business logic
│   ├── MentionService     # Mention business logic
│   ├── HistoryService     # History business logic
│   └── __init__.py        # Package exports
├── database.py            # Database connection
├── main.py                # FastAPI application entrypoint
└── __init__.py            # Package exports
```

## Features

### Comments
- **Comment CRUD**: Create, read, update, delete comments
- **Threaded Replies**: Nested comment threads
- **Rich Text**: Markdown support with sanitization
- **Attachments**: File/image attachments
- **Edit History**: Full edit history with diffs

### Mentions
- **User Mentions**: @username notifications
- **Team Mentions**: @team-name notifications
- **Smart Detection**: Auto-detect mentions in text
- **Mention Index**: Search mentions

### Real-time Updates
- **WebSocket**: Live comment updates
- **Presence**: Online/offline status
- **Typing Indicators**: Show when users are typing

### Event Integration
- **Comment Events**: Publishes comment.created, comment.updated, comment.deleted
- **Mention Events**: Publishes mention.created
- **Consumes Events**: Listens for issue, user events

## API Endpoints

### Comments

#### Create Comment
```
POST /api/v1/issues/{issue_id}/comments
Content-Type: application/json

{
  "content": "This is a comment with @john_doe mention",
  "parent_id": "uuid"  # Optional for replies
}
```

#### Get Comment
```
GET /api/v1/comments/{comment_id}

Response:
{
  "id": "uuid",
  "issue_id": "uuid",
  "author_id": "uuid",
  "content": "This is a comment",
  "content_html": "<p>This is a comment</p>",
  "parent_id": null,
  "mentions": ["uuid1", "uuid2"],
  "attachments": [...],
  "is_edited": false,
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z"
}
```

#### Update Comment
```
PATCH /api/v1/comments/{comment_id}
Content-Type: application/json

{
  "content": "Updated content"
}
```

#### Delete Comment
```
DELETE /api/v1/comments/{comment_id}
```

#### List Comments
```
GET /api/v1/issues/{issue_id}/comments?skip=0&limit=50

Query Parameters:
- skip: int (default: 0)
- limit: int (default: 50, max: 200)
- include_replies: bool (default: true)
```

### Comment History

#### Get Comment History
```
GET /api/v1/comments/{comment_id}/history

Response:
{
  "comment_id": "uuid",
  "history": [
    {
      "version": 1,
      "content": "Original content",
      "edited_by": "uuid",
      "edited_at": "2024-01-15T10:30:00Z",
      "diff": "Original content"
    },
    {
      "version": 2,
      "content": "Updated content",
      "edited_by": "uuid",
      "edited_at": "2024-01-15T11:00:00Z",
      "diff": "- Original content\n+ Updated content"
    }
  ]
}
```

### Mentions

#### Get User Mentions
```
GET /api/v1/users/{user_id}/mentions?skip=0&limit=20
```

### Real-time (WebSocket)

#### Connect
```
WS /ws/comments/{issue_id}?token={jwt_token}

Events:
- comment.created
- comment.updated
- comment.deleted
- user.typing
- user.online
- user.offline
```

#### Send Typing Indicator
```
{
  "type": "typing",
  "comment_id": "uuid"  # null for new comment
}
```

## Configuration

Environment variables (in `.env`):
```env
# Service
SERVICE_NAME=comment-service
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
  "service": "comment-service",
  "version": "1.0.0"
}
```

## Development

### Running the Service
```bash
pip install -r requirements.txt
uvicorn src.main:app --reload --host 0.0.0.0 --port 8000

# Or using Docker
docker build -f Dockerfile -t comment-service .
docker run -p 8005:8000 --env-file .env comment-service
```

### Testing
```bash
pytest tests/
pytest --cov=src tests/
```