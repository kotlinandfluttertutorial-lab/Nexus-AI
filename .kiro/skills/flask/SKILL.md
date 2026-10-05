# Skill: Flask LLM Service

## Purpose

Provides reusable engineering expertise for building the Nexus AI Flask-based LLM service
(roadmap module 06), covering route definitions, request validation, provider wiring,
streaming responses, error handling, and pytest-based tests.

## When to Use

Activate this skill when:
- Creating or modifying Flask routes for the LLM inference API
- Implementing provider service classes called from Flask routes
- Adding request validation with marshmallow or manual validation
- Implementing Server-Sent Events (SSE) streaming from Flask
- Writing pytest or requests-based tests for Flask endpoints
- Configuring Flask via environment variables or a config object

## Core Rules

1. **Application factory pattern.** Always use `create_app()` — never a module-level `app` instance that is hard to test.
2. **Blueprint-per-domain.** Group related routes into Flask Blueprints (`chat_bp`, `models_bp`, `health_bp`).
3. **Provider abstraction.** Routes never import provider SDK classes directly; they call a service layer.
4. **Environment configuration.** All secrets and model names come from environment variables via a `Config` class.
5. **Consistent JSON errors.** All error responses follow `{"error": {"code": ..., "message": ...}}`.
6. **Streaming via generator.** Use `flask.Response` with a generator and `mimetype="text/event-stream"`.

## Project Structure

```
backend/flask-service/
├── app/
│   ├── __init__.py          — create_app() factory
│   ├── config.py            — Config, DevelopmentConfig, ProductionConfig
│   ├── routes/
│   │   ├── chat.py          — Blueprint: /v1/chat
│   │   ├── models.py        — Blueprint: /v1/models
│   │   └── health.py        — Blueprint: /health
│   ├── services/
│   │   ├── chat_service.py
│   │   └── provider_factory.py
│   └── errors.py            — error handlers
├── tests/
│   ├── conftest.py
│   ├── test_chat.py
│   └── test_health.py
└── run.py
```

## Implementation Patterns

### Application Factory

```python
from flask import Flask
from app.routes.chat import chat_bp
from app.routes.health import health_bp
from app.config import config_map

def create_app(config_name: str = "development") -> Flask:
    app = Flask(__name__)
    app.config.from_object(config_map[config_name])

    app.register_blueprint(chat_bp, url_prefix="/v1/chat")
    app.register_blueprint(health_bp)

    from app.errors import register_error_handlers
    register_error_handlers(app)

    return app
```

### Config Class

```python
import os

class Config:
    TESTING = False
    OPENAI_API_KEY = os.environ.get("OPENAI_API_KEY", "")
    DEFAULT_MODEL = os.environ.get("DEFAULT_MODEL", "gpt-4o-mini")
    APP_API_KEY = os.environ.get("APP_API_KEY", "")

class DevelopmentConfig(Config):
    DEBUG = True

class ProductionConfig(Config):
    DEBUG = False

config_map = {
    "development": DevelopmentConfig,
    "production": ProductionConfig,
    "testing": Config,
}
```

### Chat Blueprint with SSE Streaming

```python
from flask import Blueprint, request, Response, jsonify, current_app
import json

chat_bp = Blueprint("chat", __name__)

@chat_bp.post("/completions")
def chat_completions():
    body = request.get_json(force=True)
    if not body or "messages" not in body:
        return jsonify({"error": {"code": "invalid_request", "message": "messages required"}}), 400

    from app.services.chat_service import ChatService
    service = ChatService(current_app.config)

    if body.get("stream"):
        def generate():
            for chunk in service.stream(body):
                yield f"data: {json.dumps(chunk)}\n\n"
            yield "data: [DONE]\n\n"
        return Response(generate(), mimetype="text/event-stream")

    return jsonify(service.complete(body))
```

### Error Handlers

```python
from flask import jsonify

def register_error_handlers(app):
    @app.errorhandler(400)
    def bad_request(e):
        return jsonify({"error": {"code": "bad_request", "message": str(e)}}), 400

    @app.errorhandler(401)
    def unauthorized(e):
        return jsonify({"error": {"code": "unauthorized", "message": "Invalid API key"}}), 401

    @app.errorhandler(500)
    def internal(e):
        return jsonify({"error": {"code": "internal_error", "message": "Internal server error"}}), 500
```

### pytest Test

```python
import pytest
from app import create_app

@pytest.fixture
def client():
    app = create_app("testing")
    app.config["TESTING"] = True
    with app.test_client() as client:
        yield client

def test_health(client):
    response = client.get("/health")
    assert response.status_code == 200

def test_chat_missing_messages(client):
    response = client.post("/v1/chat/completions", json={})
    assert response.status_code == 400
```

## Common Mistakes

| Mistake | Correct Approach |
|---|---|
| Module-level `app = Flask(__name__)` | Use `create_app()` factory |
| Hardcoding API keys in source | Use `os.environ` via `Config` class |
| Calling provider SDK directly in route | Route → service → provider |
| Blocking long inference in request context | Move to background thread or async if latency is critical |
| Returning raw exceptions | Register error handlers; return structured JSON |
| Missing `Content-Type: text/event-stream` | Set `mimetype="text/event-stream"` on `Response` |

## Testing Guidance

- Use `app.test_client()` from the `create_app("testing")` factory
- Create a `FakeProvider` that returns deterministic responses — never call real APIs in tests
- Test SSE by collecting the full response body and splitting on `\n\n`
- Set `TESTING = True` to disable error swallowing
- Use `pytest` with a `conftest.py` that provides the test `client` fixture

## Relationship to Steering

This skill applies to `backend/flask-service/`. Android steering files govern the Android workstream only. Security standards (`05-security-standards.md`) apply to both workstreams — never hardcode credentials.
