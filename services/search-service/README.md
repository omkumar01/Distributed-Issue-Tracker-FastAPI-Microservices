# Search Service

Full-text search and faceted query service powered by Elasticsearch for the issue tracker platform.

## Service Structure

```
src/
├── api/
│   ├── search.py          # Search API endpoints
│   └── __init__.py        # Package exports
├── consumers/
│   ├── event_consumers.py # Event consumers for indexing
│   └── __init__.py        # Package exports
├── indexers/
│   ├── es_client.py       # Elasticsearch client
│   ├── issue_indexer.py   # Issue indexing logic
│   ├── project_indexer.py # Project indexing logic
│   ├── user_indexer.py    # User indexing logic
│   ├── comment_indexer.py # Comment indexing logic
│   └── __init__.py        # Package exports
├── models/
│   ├── SearchQuery        # Search query model
│   ├── SearchResult       # Search result model
│   ├── FacetResult        # Facet aggregation result
│   └── __init__.py        # Package exports
├── repositories/
│   ├── SearchRepository   # Search operations
│   └── __init__.py        # Package exports
├── routers/
│   ├── search_router.py   # Search API routes
│   └── __init__.py        # Package exports
├── services/
│   ├── SearchService      # Search business logic
│   ├── IndexService       # Index management
│   ├── FacetService       # Facet aggregation
│   └── __init__.py        # Package exports
├── database.py            # Database connection (for sync)
├── main.py                # FastAPI application entrypoint
└── __init__.py            # Package exports
```

## Features

### Full-Text Search
- **Multi-field Search**: Search across title, description, comments
- **Fuzzy Matching**: Typo-tolerant search
- **Phrase Search**: Exact phrase matching
- **Wildcard Search**: Prefix/suffix wildcards
- **Boosting**: Field-level relevance boosting

### Faceted Search
- **Filter Facets**: Filter by status, priority, assignee, labels
- **Range Facets**: Date ranges, numeric ranges
- **Term Facets**: Categorical aggregations
- **Histogram Facets**: Time-based distributions

### Filtering & Sorting
- **Compound Filters**: AND/OR/NOT combinations
- **Geo Filters**: Location-based (if applicable)
- **Sort Options**: Relevance, date, priority, custom
- **Pagination**: Cursor-based and offset-based

### Index Management
- **Real-time Indexing**: Event-driven indexing
- **Bulk Reindex**: Full reindex capability
- **Index Aliases**: Zero-downtime reindex
- **Mapping Management**: Dynamic schema updates

### Event-Driven Indexing
- **Issue Events**: Index on create/update/delete
- **Project Events**: Index on create/update/delete
- **User Events**: Index on create/update/delete
- **Comment Events**: Index on create/update/delete
- **Sync API**: Manual sync endpoint

## API Endpoints

### Search

#### Search Issues
```
POST /api/v1/search/issues
Content-Type: application/json

{
  "query": "login page error",
  "project_id": "uuid",
  "filters": {
    "status": ["open", "in_progress"],
    "priority": ["high", "critical"],
    "assignee_id": "uuid",
    "labels": ["frontend", "bug"],
    "created_after": "2024-01-01T00:00:00Z",
    "created_before": "2024-12-31T23:59:59Z"
  },
  "facets": ["status", "priority", "assignee", "labels"],
  "sort": "_score",
  "page": 1,
  "size": 20,
  "highlight": true
}

Response:
{
  "total": 150,
  "hits": [
    {
      "id": "uuid",
      "type": "issue",
      "title": "Login page not loading",
      "description": "Users cannot access login page...",
      "project_id": "uuid",
      "status": "open",
      "priority": "high",
      "assignee_id": "uuid",
      "labels": ["frontend", "bug"],
      "highlights": {
        "title": ["Login page not <em>loading</em>"],
        "description": ["access <em>login</em> page <em>error</em>"]
      },
      "score": 12.5,
      "created_at": "2024-01-15T10:30:00Z"
    }
  ],
  "facets": {
    "status": [
      {"value": "open", "count": 80},
      {"value": "in_progress", "count": 45},
      {"value": "done", "count": 25}
    ],
    "priority": [
      {"value": "high", "count": 60},
      {"value": "critical", "count": 20},
      {"value": "medium", "count": 50},
      {"value": "low", "count": 20}
    ]
  },
  "page": 1,
  "size": 20,
  "total_pages": 8
}
```

#### Search Projects
```
POST /api/v1/search/projects
Content-Type: application/json

{
  "query": "website redesign",
  "filters": {
    "owner_id": "uuid",
    "is_archived": false
  },
  "facets": ["owner"],
  "sort": "updated_at",
  "page": 1,
  "size": 10
}
```

#### Search Users
```
POST /api/v1/search/users
Content-Type: application/json

{
  "query": "john",
  "filters": {
    "is_active": true
  },
  "facets": [],
  "page": 1,
  "size": 20
}
```

#### Search Comments
```
POST /api/v1/search/comments
Content-Type: application/json

{
  "query": "meeting notes",
  "issue_id": "uuid",
  "page": 1,
  "size": 20
}
```

### Global Search
```
POST /api/v1/search
Content-Type: application/json

{
  "query": "api key",
  "types": ["issues", "projects", "users", "comments"],
  "filters": {
    "project_id": "uuid"
  },
  "page": 1,
  "size": 10
}

Response:
{
  "issues": {"total": 5, "hits": [...]},
  "projects": {"total": 1, "hits": [...]},
  "users": {"total": 2, "hits": [...]},
  "comments": {"total": 3, "hits": [...]}
}
```

### Suggestions

#### Autocomplete
```
GET /api/v1/search/suggest?q=log&type=issues&size=5

Response:
{
  "suggestions": [
    {"text": "login error", "type": "issues"},
    {"text": "logout issue", "type": "issues"},
    {"text": "log aggregation", "type": "projects"}
  ]
}
```

### Index Management

#### Reindex All
```
POST /api/v1/index/reindex
Content-Type: application/json

{
  "types": ["issues", "projects", "users", "comments"]
}
```

#### Reindex Type
```
POST /api/v1/index/reindex/issues
```

#### Get Index Status
```
GET /api/v1/index/status

Response:
{
  "issues": {"doc_count": 10000, "size": "50mb", "health": "green"},
  "projects": {"doc_count": 500, "size": "5mb", "health": "green"},
  "users": {"doc_count": 2000, "size": "10mb", "health": "green"},
  "comments": {"doc_count": 50000, "size": "100mb", "health": "green"}
}
```

## Configuration

Environment variables (in `.env`):
```env
# Service
SERVICE_NAME=search-service
SERVICE_VERSION=1.0.0
ENVIRONMENT=development

# Elasticsearch
ELASTICSEARCH_URL=http://elasticsearch:9200
ELASTICSEARCH_USERNAME=elastic
ELASTICSEARCH_PASSWORD=changeme
ELASTICSEARCH_VERIFY_CERTS=false

# Database (for sync)
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
  "service": "search-service",
  "version": "1.0.0",
  "elasticsearch": "connected"
}
```

## Development

### Running the Service
```bash
pip install -r requirements.txt
uvicorn src.main:app --reload --host 0.0.0.0 --port 8000

# Or using Docker
docker build -f Dockerfile -t search-service .
docker run -p 8007:8000 --env-file .env search-service
```

### Testing
```bash
pytest tests/
pytest --cov=src tests/
```

### Elasticsearch Index Mappings

#### Issues Index
```json
{
  "mappings": {
    "properties": {
      "id": {"type": "keyword"},
      "project_id": {"type": "keyword"},
      "title": {"type": "text", "analyzer": "standard"},
      "description": {"type": "text", "analyzer": "standard"},
      "issue_type": {"type": "keyword"},
      "priority": {"type": "keyword"},
      "status": {"type": "keyword"},
      "assignee_id": {"type": "keyword"},
      "reporter_id": {"type": "keyword"},
      "labels": {"type": "keyword"},
      "created_at": {"type": "date"},
      "updated_at": {"type": "date"},
      "due_date": {"type": "date"}
    }
  }
}
```

#### Projects Index
```json
{
  "mappings": {
    "properties": {
      "id": {"type": "keyword"},
      "name": {"type": "text", "analyzer": "standard"},
      "description": {"type": "text", "analyzer": "standard"},
      "key": {"type": "keyword"},
      "owner_id": {"type": "keyword"},
      "is_archived": {"type": "boolean"},
      "created_at": {"type": "date"},
      "updated_at": {"type": "date"}
    }
  }
}
```