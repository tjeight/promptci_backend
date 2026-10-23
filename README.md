# PromptCI Backend

**Regression testing for LLM applications.**

PromptCI is an open-source backend project designed to help developers evaluate prompt and model changes before deploying them. It aims to make LLM testing reproducible by managing prompt versions, test datasets, evaluation runs, and regression results through a single API.

Changing a prompt can improve one response while breaking another. PromptCI aims to catch these regressions before they reach production.

> **Status:** Early development — the initial backend foundation is being built.

## The Problem

Traditional unit tests are effective for deterministic code, but LLM applications can produce different outputs when prompts, models, or generation parameters change.

PromptCI aims to provide a repeatable evaluation workflow that helps developers:

- Maintain reusable test datasets.
- Version prompts and model configurations.
- Execute evaluations in the background.
- Score model outputs against expected behavior.
- Compare baseline and candidate runs.
- Detect regressions before deployment.
- Integrate evaluation results into CI/CD pipelines.

## Planned Features

### 1. Authentication and Project Management
- User registration and authentication.
- User-owned projects.
- Project-level resource isolation.

### 2. Prompt Versioning
- Store prompt templates and variables.
- Maintain immutable prompt versions.
- Associate model configurations with evaluations.
- Reproduce previous evaluations using saved configuration snapshots.

### 3. Test Dataset Management
- Create reusable evaluation datasets.
- Define input variables and expected behavior.
- Configure evaluation assertions.
- Reuse datasets across prompt versions.

### 4. Evaluation Engine
- Execute prompts against an LLM provider.
- Support exact-match evaluation.
- Validate structured outputs and JSON schemas.
- Run custom Python assertions.
- Support rubric-based LLM judging.

### 5. Asynchronous Execution
- Queue evaluation runs.
- Execute test cases concurrently with bounded parallelism.
- Track run and individual test-case statuses.
- Handle timeouts, retries, and provider errors.
- Persist evaluation results.

### 6. Regression Detection
- Compare baseline and candidate runs.
- Identify newly failing and improved test cases.
- Track pass rates, latency, token usage, and errors.
- Apply configurable release-quality thresholds.

### 7. CI/CD Integration
- Provide a command-line interface for running evaluations.
- Integrate regression checks with GitHub Actions.
- Return a failing exit code when configured quality gates are breached.

*These are planned capabilities; their availability will depend on implementation progress.*

## Technology Stack

| Technology | Purpose |
|---|---|
| Python | Core application logic |
| FastAPI | REST API |
| Pydantic | Request validation and structured data |
| PostgreSQL | Persistent application and evaluation data |
| SQLAlchemy | Database access and ORM |
| Alembic | Database schema migrations |
| Redis | Background task broker and supporting cache |
| Celery | Asynchronous evaluation workers |
| OpenAI SDK | Initial LLM provider integration |
| pytest | Automated testing |
| Docker Compose | Local development environment |
| GitHub Actions | CI/CD integration |

PromptCI will use a framework-independent evaluation engine. LangChain and LangGraph are not required by the initial core implementation.

## Architecture

The planned execution flow is:

```text
Client / CLI / GitHub Actions
            |
            v
       FastAPI API
            |
      Authentication
      and Authorization
            |
            v
        PostgreSQL
            |
            v
      Redis + Celery
            |
            v
    Evaluation Workers
            |
      +-----+------+
      |            |
  Model Adapter  Evaluators
      |            |
      +-----+------+
            |
            v
     Evaluation Results
            |
            v
   Comparison and CI Gate
```

The API will create evaluation runs and return a run identifier. Background workers will execute the test cases and persist results, allowing clients to inspect progress without holding an HTTP request open for the entire evaluation.

## Getting Started

### Prerequisites

- Python 3.12 or a compatible supported Python version.
- [uv](https://docs.astral.sh/uv/)
- Docker and Docker Compose.
- An LLM provider API key when model execution is implemented.

### 1. Clone the repository

```bash
git clone https://github.com/tjeight/promptci_backend.git
cd promptci_backend
```

### 2. Install dependencies

```bash
uv sync
```

If the project dependencies have not yet been declared in `pyproject.toml`, install the dependencies specified by the current project setup before continuing.

### 3. Configure environment variables

Create a local `.env` file using `.env.example` as a template.

Example configuration for local development:

```dotenv
POSTGRES_PASSWORD=change-this-local-password
DATABASE_URL=postgresql+asyncpg://promptci:change-this-local-password@localhost:5433/promptci
```

Configure provider credentials only when the relevant integration is implemented. Never commit `.env` files or API keys to version control.

### 4. Start PostgreSQL

If the repository contains the planned `compose.yaml` configuration:

```bash
docker compose up -d postgres
docker compose ps
```

### 5. Run the API

```bash
uv run uvicorn app.main:app --reload
```

When the application and health endpoint are available, open:

- Health check: `http://127.0.0.1:8000/health`
- Interactive API documentation: `http://127.0.0.1:8000/docs`

Follow the actual repository structure and available endpoints as development progresses.

## Development Roadmap

- [ ] Initialize FastAPI application and project structure.
- [ ] Configure PostgreSQL, SQLAlchemy, and Alembic.
- [ ] Implement authentication and project ownership.
- [ ] Implement prompt and prompt-version management.
- [ ] Implement dataset and test-case management.
- [ ] Build the first model adapter.
- [ ] Implement deterministic evaluators.
- [ ] Add asynchronous evaluation execution.
- [ ] Persist per-case and aggregate results.
- [ ] Implement baseline-versus-candidate comparisons.
- [ ] Add LLM-as-judge evaluation.
- [ ] Build the CLI and GitHub Actions quality gate.
- [ ] Add integration tests, deployment instructions, and a reproducible demo.

## Engineering Principles

PromptCI is being designed around a few important principles:

- **Reproducibility:** Preserve the configuration and versions used by each evaluation run.
- **Reliability:** Handle provider failures, worker retries, timeouts, and duplicate task delivery.
- **Isolation:** Prevent users from accessing another user's projects or evaluation results.
- **Observability:** Record useful execution metadata and errors.
- **Extensibility:** Support additional model providers and evaluator types through clear interfaces.
- **Security:** Protect credentials, validate requests, and avoid exposing sensitive data in logs.

## Project Scope

The initial goal is a working backend MVP, not a full enterprise platform. The focus is on reliable evaluation execution, versioned test data, regression detection, and automated quality gates.

A frontend, additional providers, advanced analytics, and team collaboration features can be added after the core evaluation workflow is reliable.

## Contributing

Contributions and suggestions are welcome. As the project evolves, please use issues to discuss bugs, improvements, and proposed features before submitting substantial changes.


**PromptCI — test your prompts before you ship them.**