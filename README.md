# Data Ingestion and Aggregation Service

A production-grade Python backend service for ingesting, storing, and aggregating event data with idempotency, concurrency safety, and high performance.

## Features

- **Idempotent Event Ingestion**: Duplicate events are safely ignored
- **Bulk Processing**: Handle up to 5,000 events per request
- **Async I/O**: Non-blocking operations for high throughput
- **Background Aggregation**: Incremental time-bucketed metrics
- **Concurrency Safe**: Database constraints prevent race conditions
- **Comprehensive Tests**: Full pytest coverage

## Tech Stack

- **Framework**: FastAPI
- **Database**: PostgreSQL (SQLite supported)
- **ORM**: SQLAlchemy 2.0 (async)
- **Testing**: Pytest with async support

## Setup Instructions

### 1. Install Dependencies

```bash
extract the zip file
cd /Users/sachinagarwal/Desktop/assessment
source ass_env/bin/activate
pip install -r requirements.txt
```

### 2. Configure Database

Create a `.env` file:

```bash
cp .env.example .env
```

For SQLite (simplest):
```
DATABASE_URL=sqlite+aiosqlite:///./events.db
```

For PostgreSQL:
```
DATABASE_URL=postgresql+asyncpg://user:password@localhost/events_db
```

### 3. Initialize Database

```bash
python -c "from app.database import init_db; import asyncio; asyncio.run(init_db())"
```

### 4. Run the API Server

```bash
uvicorn app.main:app --reload --port 8000
```

API will be available at: http://localhost:8000
Interactive docs: http://localhost:8000/docs

### 5. Run Background Aggregator (Optional)

In a separate terminal:

```bash
python -m app.aggregator
```

### 6. Run Tests

```bash
pytest -v
```

## API Endpoints

### POST /events
Create a single event (idempotent)

```bash
curl -X POST http://localhost:8000/events \
  -H "Content-Type: application/json" \
  -d '{
    "event_id": "evt_001",
    "tenant_id": "tenant_1",
    "source": "web",
    "event_type": "click",
    "timestamp": "2024-01-15T10:30:00Z",
    "payload": {"page": "home"}
  }'
```

### POST /events/bulk
Bulk create events (up to 5,000)

```bash
curl -X POST http://localhost:8000/events/bulk \
  -H "Content-Type: application/json" \
  -d '{
    "events": [
      {
        "event_id": "evt_001",
        "tenant_id": "tenant_1",
        "source": "mobile",
        "event_type": "view",
        "timestamp": "2024-01-15T10:30:00Z",
        "payload": {}
      }
    ]
  }'
```

### GET /events
Query events with filters

```bash
curl "http://localhost:8000/events?tenant_id=tenant_1&source=web&page=1&page_size=100"
```

### GET /metrics
Get aggregated metrics

```bash
curl "http://localhost:8000/metrics?tenant_id=tenant_1&bucket_size=minute"
```

### GET /health
Health check

```bash
curl http://localhost:8000/health
```

### GET /ready
Readiness check (includes DB connectivity)

```bash
curl http://localhost:8000/ready
```

## Performance Considerations

1. **Bulk Inserts**: Uses PostgreSQL's `ON CONFLICT DO NOTHING` for efficient idempotent bulk operations
2. **Indexes**: Composite indexes on (tenant_id, timestamp) and (tenant_id, source, event_type)
3. **Async I/O**: All database operations are non-blocking
4. **Pagination**: Stable sorting with offset/limit for large result sets
5. **Aggregation**: Incremental processing with upserts to avoid full table scans

## Testing

Run all tests:
```bash
pytest -v
```

Run specific test:
```bash
pytest tests/test_api.py::test_create_event_idempotent -v
```

Run with coverage:
```bash
pytest --cov=app --cov-report=html
```

## Architecture Highlights

- **Idempotency**: Primary key constraint on event_id ensures no duplicates
- **Concurrency**: Database-level constraints handle race conditions
- **Async**: SQLAlchemy async engine with asyncpg driver
- **Aggregation**: Background worker processes events into time buckets
- **Validation**: Pydantic schemas enforce UTC timestamps and payload limits
