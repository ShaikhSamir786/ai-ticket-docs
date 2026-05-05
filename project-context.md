# AI Customer Ticket Triage Project Context

## Project overview

AI Customer Ticket Triage is a microservice-based support automation project that receives customer support tickets, understands the issue using AI, assigns priority, routes the ticket to the right team, and can optionally generate a suggested reply for the support agent.

The project is designed to simulate a real business support platform instead of a simple AI demo. Modern AI triage systems typically focus on ticket understanding, prioritization, categorization, routing, and response assistance, which makes this a practical backend-heavy project.

## Main goal

The goal of this project is to reduce manual ticket sorting and make support operations faster and more consistent. AI-based triage platforms are commonly used to auto-route tickets, detect urgency, and support agents with suggested actions or replies.

This project is also a strong portfolio piece for backend and microservice architecture because it combines APIs, background processing, AI classification, queue handling, and dashboard-based review workflows.

## Repositories in this system

| Repo name | Type | Role |
|---|---|---|
| `ai-ticket-web-nextjs` | Frontend (Next.js) | Project homepage with only project details, no dashboard or ticket management features. |
| `ai-ticket-web-app-react-graphql` | Frontend (React, GraphQL) | Agent-facing web app for viewing tickets, triage results, status, assignments, and AI suggestions. |
| `ai-ticket-shared-schema` | Shared library | Database schema structure and definitions shared across services. Shared-library. |
| `ai-ticket-shared-lib` | Shared library | Shared code utilities: logger, config loader, error handling, helpers, common decorators, queue constants. Shared-library. |
| `ai-ticket-main-server` | Backend service | Gateway for all API communication between repos, request validation, auth, rate limiting, and internal routing. |
| `ai-ticket-llm-server` | Backend service | LLM/model management server for model selection, prompt templates, token usage tracking, and API key management. |
| `ai-ticket-processor-server` | Backend service | AI service for LLM classification, priority prediction, reply suggestion, and routing recommendations. |
| `ai-ticket-core-server` | Backend service | Core ticket management service for CRUD, assignment, status tracking, and audit history. |
| `ai-ticket-scheduler-server` | Backend service | Handles queue consumers, retry flows, cron jobs, SLA checks, stale-ticket scans, and async processing. |

## Frontend repositories

### `ai-ticket-web-nextjs`

Next.js project homepage repository. Contains only project details, overview, and static information. No dashboard or ticket management features.

### `ai-ticket-web-app-react-graphql`

React + GraphQL frontend repository for the system. It acts as the support team dashboard where agents or admins can review incoming tickets, see AI-generated category and priority predictions, inspect confidence scores, override decisions, and update final status.

Typical frontend screens include:
- Login and role-based access
- Ticket list with filters
- Ticket detail page
- AI triage result panel
- Manual override and reassignment UI
- SLA and analytics dashboard
- Activity/audit history

## Shared libraries

Both shared libraries are included as **shared-library** dependencies in each backend service, allowing centralized management of shared code while keeping each service's dependencies clean.

### `ai-ticket-shared-schema`

Shared database schema structure and definitions used across all backend services. Ensures consistency in data models and relationships across microservices.

Suggested contents:
- Prisma schema files or SQL migrations
- Entity definitions (Ticket, User, Assignment, etc.)
- Relationship mappings
- Index definitions
- Shared type exports for TypeScript

### `ai-ticket-shared-lib`

Shared code utilities library used by all backend services to avoid duplication and ensure consistency.

Suggested utilities:
- Logger: Structured logging with correlation IDs
- Config loader: Environment-based configuration management
- Error handling: Custom error classes and error factory
- Helpers: Common utility functions
- Common decorators: Reusable NestJS  decorators
- Queue constants: Job names, queue names, exchange keys

## Backend repositories

### `ai-ticket-main-server`

Gateway for all API communication between repos. Manages authentication, request validation, rate limiting, response shaping, and routes requests to internal services.

Suggested responsibilities:
- Accept requests from frontend
- Validate payloads
- Handle auth and middleware
- Forward ticket operations to `ai-ticket-core-server`
- Trigger internal workflows
- Expose unified API for the frontend

### `ai-ticket-llm-server`

LLM/model management server for centralized AI operations. Provides provider abstraction, model routing, and resilience layer for AI calls.

Suggested responsibilities:
- Provider abstraction for OpenAI, Anthropic, Ollama, etc.
- Model selection and routing based on request type
- Prompt template management and versioning
- Token usage tracking and cost monitoring
- API key rotation and management
- Circuit breakers and retry logic for resilience
- Model performance monitoring and fallbacks

### `ai-ticket-processor-server`

AI orchestration service that applies business rules and coordinates triage logic. Transforms raw ticket content into structured output by orchestrating calls to the LLM server and processing the responses.

Suggested responsibilities:
- Apply triage business rules and validation
- Orchestrate calls to LLM server for classification
- Detect intent (billing, login, bug, refund, cancellation)
- Predict urgency or priority
- Suggest target team based on category rules
- Generate confidence score and review flags
- Draft suggested reply from LLM responses
- Return structured JSON output for `ai-ticket-core-server`

Example output shape:

```json
{
  "category": "billing",
  "priority": "high",
  "sentiment": "frustrated",
  "assignedTeam": "finance-support",
  "confidence": 0.91,
  "needsHumanReview": false,
  "suggestedReply": "Sorry for the inconvenience. We are reviewing the duplicate charge and will update you shortly.",
  "timestamp": "2026-05-05T14:04:00Z",
  "modelUsed": "gpt-4-turbo"
}
```

### `ai-ticket-core-server`

Core business service for ticket lifecycle management. Manages ticket records including CRUD operations, assignment, status tracking, and audit history.

Suggested responsibilities:
- Create new tickets
- Update status
- Store category, priority, and team assignment
- Save audit history
- Manage manual overrides from agents
- Expose ticket queries for dashboard views

Suggested data examples:
- ticket id
- customer id
- source channel
- subject
- message
- category
- priority
- assigned team
- assigned agent
- status
- triage confidence
- created at / updated at

### `ai-ticket-scheduler-server`

Combines background workers and scheduler logic in one repo. Handles queue consumers, retry flows, cron jobs, SLA checks, stale-ticket scans, and async processing.

Suggested responsibilities:
- Consume ticket-processing jobs from queue
- Call processor service asynchronously
- Retry failed jobs
- Move failed jobs to dead-letter flow if needed
- Run scheduled SLA checks
- Scan stale or unassigned tickets
- Trigger reminder, escalation, or cleanup jobs
- Generate periodic operational reports

Internal module idea:
- `workers/` for queue consumers
- `schedulers/` for cron jobs
- `jobs/` for shared job definitions
- `adapters/` for Redis, email, webhooks, and integrations

## How the system works end to end

1. A user, email parser, chatbot, or external integration submits a support ticket.
2. The request reaches `ai-ticket-main-server`.
3. Main server forwards the request to `ai-ticket-core-server`.
4. Core server stores the raw ticket and pushes a background job to the queue.
5. `ai-ticket-scheduler-server` consumes the job from the queue.
6. Scheduler server sends the content to `ai-ticket-processor-server` for triage orchestration.
7. Processor server applies business rules and forwards the request to `ai-ticket-llm-server`.
8. LLM server routes the request to the appropriate AI provider (OpenAI, Anthropic, Ollama, etc.) and returns the response.
9. Processor server processes the AI output into structured results (category, priority, confidence, reply suggestion).
10. Core server updates the ticket with triage results.
11. `ai-ticket-web-app-react-graphql` shows the result to the support agent.
12. Agents can approve, override, reassign, or respond.

## Why this architecture is good

This architecture is useful because it clearly separates concerns: project homepage, agent dashboard, API gateway, LLM management, ticket business logic, AI reasoning, and asynchronous processing. That makes the system easier to scale, test, deploy, and explain in interviews or portfolio walkthroughs.

Using a dedicated scheduler server for queue and cron jobs is a practical choice that separates async processing from the main API flows, making the system more maintainable as it grows.

## Recommended stack

Suggested stack for this project:
- Frontend (ai-ticket-web-nextjs): Next.js, TypeScript, Tailwind CSS — for marketing and portfolio 
- Frontend (ai-ticket-web-app-react-graphql): React, GraphQL, TypeScript, TanStack Query
- Gateway (ai-ticket-main-server): Node.js, NestJS, TypeScript
- LLM service (ai-ticket-llm-server): Node.js, TypeScript, OpenAI or Anthropic API
- AI service (ai-ticket-processor-server): Node.js, TypeScript, OpenAI or Anthropic API
- Ticket service (ai-ticket-core-server): Node.js, NestJS, PostgreSQL, Prisma
- Scheduler (ai-ticket-scheduler-server): Node.js, BullMQ, Redis, node-cron
- Infra: Docker, Docker Compose, GitHub Actions

## Suggested elevator pitch

AI Customer Ticket Triage is a microservice-based support automation platform with a project homepage (ai-ticket-web-nextjs), an agent-facing dashboard (ai-ticket-web-app-react-graphql), and centralized LLM management (ai-ticket-llm-server). It uses AI to classify incoming customer tickets, predict urgency, route them to the right team, and assist support agents with AI-generated suggestions and triage results.[cite:15][cite:22][cite:49]
