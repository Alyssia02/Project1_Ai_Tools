# 🤖 Multi-AI REST API Gateway

A production-ready FastAPI application that serves as a unified gateway for multiple AI services (OpenAI, Anthropic Claude, Google Gemini, and Cohere). Interact with different AI models through a single, standardized interface — with built-in authentication, rate limiting, caching, and usage tracking.

---

## 📋 Table of Contents

- [Features](#-features)
- [Tech Stack](#-tech-stack)
- [Architecture](#-architecture)
- [Getting Started](#-getting-started)
- [Configuration](#-configuration)
- [API Endpoints](#-api-endpoints)
- [Database Migrations](#-database-migrations)
- [Testing](#-testing)
- [Docker Deployment](#-docker-deployment)
- [Project Structure](#-project-structure)
- [Roadmap](#-roadmap)
- [Contributing](#-contributing)
- [License](#-license)

---

## ✨ Features

- **Unified AI Interface** — One API to talk to OpenAI, Claude, Gemini, and Cohere
- **JWT Authentication** — Secure token-based auth with bcrypt password hashing
- **Redis Caching** — Cache identical prompts to cut costs and latency
- **Rate Limiting** — Per-user request throttling
- **Usage Tracking** — Track tokens and costs per user in PostgreSQL
- **Async by Default** — Parallel AI calls with `async/await`
- **Auto-Generated Docs** — Swagger UI and ReDoc out of the box
- **Production-Ready** — Structured logging, custom exceptions, comprehensive tests
- **Dockerized** — One command to spin up the whole stack

---

## 🛠 Tech Stack

| Layer | Technology |
|---|---|
| Framework | FastAPI |
| Database | PostgreSQL + SQLAlchemy |
| Cache & Rate Limiting | Redis |
| Validation | Pydantic v2 |
| Authentication | JWT (python-jose) + bcrypt |
| Migrations | Alembic |
| Testing | pytest + pytest-asyncio + httpx |
| Containerization | Docker + Docker Compose |
| AI SDKs | openai, anthropic, google-generativeai, cohere |

---

## 🏗 Architecture

```
        ┌─────────────┐
        │   Client    │
        └──────┬──────┘
               │ HTTPS + JWT
               ▼
        ┌─────────────┐
        │   FastAPI   │
        │   Gateway   │
        └──┬───┬───┬──┘
           │   │   │
     ┌─────┘   │   └─────┐
     ▼         ▼         ▼
 ┌───────┐ ┌──────┐ ┌─────────┐
 │ Redis │ │ PgSQL│ │ AI APIs │
 │(cache)│ │(data)│ │(4 prov.)│
 └───────┘ └──────┘ └─────────┘
```

The gateway sits between clients and four AI providers. Every request goes through authentication, rate limiting, and cache lookup before hitting an upstream provider. Usage is logged to PostgreSQL for billing and analytics.

---

## 🚀 Getting Started

### Prerequisites

- Python 3.11+
- PostgreSQL 14+ (or Docker)
- Redis 6+ (or Docker)
- API keys for the AI providers you want to use

### Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/ai-gateway.git
cd ai-gateway

# Create virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Copy environment file and fill in your values
cp .env.example .env
```

### Run the Database

If you don't have PostgreSQL and Redis locally, run them with Docker:

```bash
docker run -d --name ai-gateway-db \
  -e POSTGRES_USER=user \
  -e POSTGRES_PASSWORD=password \
  -e POSTGRES_DB=ai_gateway \
  -p 5432:5432 postgres:16

docker run -d --name ai-gateway-redis \
  -p 6379:6379 redis:7-alpine
```

### Apply Migrations

```bash
alembic upgrade head
```

### Start the Server

```bash
uvicorn app.main:app --reload
```

The API is now live at `http://localhost:8000`.

- **Swagger UI:** http://localhost:8000/docs
- **ReDoc:** http://localhost:8000/redoc
- **Health Check:** http://localhost:8000/health

---

## ⚙️ Configuration

All configuration lives in `.env`. Generate a strong secret key with:

```bash
openssl rand -hex 32
```

Example `.env`:

```bash
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/ai_gateway

# Redis
REDIS_URL=redis://localhost:6379/0

# JWT
SECRET_KEY=your-generated-secret-here
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# AI Provider Keys
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
GOOGLE_API_KEY=...
COHERE_API_KEY=...

# Rate Limiting
RATE_LIMIT_PER_MINUTE=10
```

> ⚠️ Never commit `.env` to version control. It's already in `.gitignore`.

---

## 📡 API Endpoints

### Authentication

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/v1/auth/register` | Register a new user |
| POST | `/api/v1/auth/login` | Login and receive JWT token |

### Chat (coming in Sprint 2)

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/v1/chat/openai` | Send message to OpenAI |
| POST | `/api/v1/chat/claude` | Send message to Claude |
| POST | `/api/v1/chat/gemini` | Send message to Gemini |
| POST | `/api/v1/chat/cohere` | Send message to Cohere |
| POST | `/api/v1/chat/compare` | Send to multiple providers in parallel |

### Users (coming in Sprint 4)

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/v1/users/me` | Get current user profile |
| GET | `/api/v1/users/me/usage` | Get usage and cost stats |

### Quick Test

```bash
# Register
curl -X POST "http://localhost:8000/api/v1/auth/register" \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","username":"testuser","password":"testpass123"}'

# Login (get token)
curl -X POST "http://localhost:8000/api/v1/auth/login" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=testuser&password=testpass123"
```

---

## 🗄 Database Migrations

This project uses Alembic for schema versioning.

```bash
# Create a new migration after changing models
alembic revision --autogenerate -m "Description of change"

# Apply migrations
alembic upgrade head

# Rollback last migration
alembic downgrade -1

# View migration history
alembic history
```

---

## 🧪 Testing

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=app --cov-report=html

# Run a specific test file
pytest tests/test_auth.py -v
```

Test fixtures (in-memory DB, test client, sample users) live in `tests/conftest.py`.

---

## 🐳 Docker Deployment

The full stack — app, PostgreSQL, and Redis — can be started with Docker Compose:

```bash
# Build and start everything
docker-compose up --build

# Run in detached mode
docker-compose up -d

# View logs
docker-compose logs -f app

# Stop everything
docker-compose down

# Stop and remove volumes (clean slate)
docker-compose down -v
```

After startup, the API is available at `http://localhost:8000`.

---

## 📁 Project Structure

```
ai-gateway/
├── app/
│   ├── main.py              # FastAPI entry point
│   ├── config.py            # Settings management
│   ├── database.py          # DB connection
│   │
│   ├── api/v1/
│   │   ├── router.py        # Main router
│   │   └── endpoints/       # Auth, chat, users, health
│   │
│   ├── core/                # Auth, security, cache, rate limit
│   ├── models/              # SQLAlchemy models
│   ├── schemas/             # Pydantic schemas
│   ├── services/            # AI provider integrations
│   └── utils/               # Logger, exceptions, helpers
│
├── tests/                   # pytest test suite
├── alembic/                 # Database migrations
├── .env.example             # Environment template
├── docker-compose.yml       # Multi-container setup
├── Dockerfile               # App container
└── requirements.txt         # Python dependencies
```

---

## 🗺 Roadmap

The project is built in six sprints:

- [x] **Sprint 1** — Foundation: project setup, PostgreSQL, JWT authentication
- [ ] **Sprint 2** — AI service layer: unified interface for 4 providers
- [ ] **Sprint 3** — Redis caching and rate limiting
- [ ] **Sprint 4** — Usage tracking and cost calculation
- [ ] **Sprint 5** — Comprehensive testing and error handling
- [ ] **Sprint 6** — Dockerization and deployment

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

Please make sure your code passes `pytest` and follows the existing structure.

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---

## 👤 Author

Built as a learning project to master FastAPI, async Python, and production API design.

If you find it useful, give it a ⭐ on GitHub!
