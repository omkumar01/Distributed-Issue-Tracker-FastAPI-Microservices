# API Gateway

Central entry point for all API requests, providing request routing, TLS termination, rate limiting, and JWT validation for the issue tracker platform.

## Service Structure

```
├── main.py                    # FastAPI application entrypoint
├── config/
│   ├── traefik.yml           # Traefik static configuration
│   └── dynamic.yml           # Traefik dynamic configuration
└── requirements.txt           # Python dependencies (httpx)
```

## Features

### Request Routing
- **Path-based routing**: Routes requests to appropriate backend services
- **Service discovery**: Automatic service endpoint resolution
- **Load balancing**: Round-robin load distribution

### Security
- **JWT Validation**: Validates JWT tokens before forwarding requests
- **Rate Limiting**: Per-client rate limiting
- **CORS**: Configurable Cross-Origin Resource Sharing
- **TLS Termination**: Handles HTTPS termination

### Observability
- **Request Logging**: Structured logging for all requests
- **Health Checks**: Service health endpoints
- **Metrics**: Request latency, error rates, throughput

## Service Mapping

| Path Prefix | Backend Service | Port |
|-------------|-----------------|------|
| /api/v1/auth | auth-service | 8001 |
| /api/v1/users | user-service | 8002 |
| /api/v1/projects | project-service | 8003 |
| /api/v1/issues | issue-service | 8004 |
| /api/v1/comments | comment-service | 8005 |
| /api/v1/notifications | notification-service | 8006 |
| /api/v1/search | search-service | 8007 |
| /api/v1/audit | audit-service | 8008 |

## API Endpoints

### Health Check
```
GET /health

Response:
{
  "status": "healthy",
  "service": "api-gateway",
  "version": "1.0.0"
}
```

### Ping (Traefik)
```
GET /ping

Response: 200 OK
```

## Configuration

Environment variables (in `.env`):
```env
# Service
SERVICE_NAME=api-gateway
SERVICE_VERSION=1.0.0
ENVIRONMENT=development

# Backend Services
AUTH_SERVICE_URL=http://auth-service:8000
USER_SERVICE_URL=http://user-service:8000
PROJECT_SERVICE_URL=http://project-service:8000
ISSUE_SERVICE_URL=http://issue-service:8000
COMMENT_SERVICE_URL=http://comment-service:8000
NOTIFICATION_SERVICE_URL=http://notification-service:8000
SEARCH_SERVICE_URL=http://search-service:8000
AUDIT_SERVICE_URL=http://audit-service:8000

# Security
SECRET_KEY=your-secret-key-change-in-production
ALGORITHM=HS256

# Rate Limiting
RATE_LIMIT_REQUESTS=100
RATE_LIMIT_WINDOW=60
```

## Traefik Configuration

The gateway uses Traefik v2.10 as a reverse proxy:

### Static Configuration (`config/traefik.yml`)
- Entry points (HTTP/HTTPS)
- API dashboard
- Providers (file-based)

### Dynamic Configuration (`config/dynamic.yml`)
- Routers with path matching
- Middleware (rate limiting, auth)
- Services (load balancers)
- TLS certificates

## Development

### Running the Service

```bash
# Install dependencies
pip install -r requirements.txt

# Run with uvicorn
uvicorn main:app --reload --host 0.0.0.0 --port 8000

# Or using Docker
docker build -f Dockerfile -t api-gateway .
docker run -p 8000:8000 --env-file .env api-gateway
```

### Testing

```bash
# Health check
curl http://localhost:8000/health

# Test routing
curl -H "Authorization: Bearer <token>" http://localhost:8000/api/v1/auth/me
```

## Security Considerations

1. **JWT Validation**: All protected routes require valid JWT
2. **Rate Limiting**: Prevents abuse and DDoS
3. **HTTPS**: Use TLS in production
4. **CORS**: Configure `allow_origins` appropriately
5. **Secrets**: Change `SECRET_KEY` in production

## Future Enhancements

- [ ] WebSocket support for real-time features
- [ ] Request/response transformation
- [ ] Circuit breaker pattern
- [ ] Advanced rate limiting (token bucket)
- [ ] API versioning support
- [ ] Request caching
- [ ] GraphQL gateway support