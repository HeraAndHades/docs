# API Reference

Open WebUI provides a comprehensive REST API for programmatic access to all features. All endpoints require authentication via Bearer token except where noted.

## Base URL

```
http://localhost:3000/api
```

## Authentication

Include your API key in the Authorization header:

```bash
Authorization: Bearer YOUR_API_KEY
```

Generate API keys from **Settings > Account > API Keys**.

## Core Endpoints

### Chat Management

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/chats/new` | Create new chat session |
| GET | `/api/chats` | List all chats |
| GET | `/api/chats/{id}` | Get specific chat |
| POST | `/api/chats/{id}` | Update chat (add messages) |
| DELETE | `/api/chats/{id}` | Delete chat |

#### Create Chat Example

```bash
curl -X POST http://localhost:3000/api/chats/new \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "llama3.1:latest",
    "title": "My New Chat",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

### Chat Completions

The unified completion endpoint accepts OpenAI-compatible requests:

```bash
curl -X POST http://localhost:3000/api/chat/completions \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "llama3.1:latest",
    "messages": [{"role": "user", "content": "Explain quantum computing"}],
    "stream": true
  }'
```

### File & RAG Management

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/v1/files` | Upload file (multipart/form-data) |
| GET | `/api/v1/files` | List uploaded files |
| GET | `/api/v1/files/{id}/process/status` | Check processing status |
| DELETE | `/api/v1/files/{id}` | Delete file |

#### File Upload Workflow

Files are processed asynchronously. Always poll for completion:

```python
import requests
import time

def upload_and_wait(file_path, token):
    # 1. Upload file
    with open(file_path, 'rb') as f:
        response = requests.post(
            'http://localhost:3000/api/v1/files',
            headers={'Authorization': f'Bearer {token}'},
            files={'file': f}
        )
    file_id = response.json()['id']
    
    # 2. Poll until processing completes
    while True:
        status = requests.get(
            f'http://localhost:3000/api/v1/files/{file_id}/process/status',
            headers={'Authorization': f'Bearer {token}'}
        ).json()
        
        if status['status'] == 'completed':
            return file_id
        elif status['status'] == 'failed':
            raise Exception("Processing failed")
        time.sleep(2)
```

### Knowledge Base Management

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/v1/knowledge/create` | Create knowledge base |
| GET | `/api/v1/knowledge` | List knowledge bases |
| POST | `/api/v1/knowledge/{id}/file/add` | Add file to KB |
| DELETE | `/api/v1/knowledge/{id}/delete` | Delete knowledge base |

### Model Management

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/models` | List available models |
| POST | `/api/models/pull` | Pull model from registry |

## OpenAI Compatibility Layer

Open WebUI exposes an OpenAI-compatible API at `/v1`:

```bash
curl http://localhost:3000/v1/models \
  -H "Authorization: Bearer YOUR_TOKEN"

curl -X POST http://localhost:3000/v1/chat/completions \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"model": "llama3.1", "messages": [{"role": "user", "content": "Hi"}]}'
```

## Rate Limits & Error Handling

- Rate limits depend on your configured backend (Ollama, OpenAI, etc.)
- Common HTTP status codes:
  - `200` - Success
  - `401` - Invalid or missing token
  - `403` - Insufficient permissions
  - `404` - Resource not found
  - `422` - Validation error

## WebSocket Events

Real-time updates use WebSocket connections to `/ws/socket.io/`. For multi-replica deployments, Redis-backed WebSockets are required.
