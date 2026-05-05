# Service Communication Architecture

## Overview

This document describes how different components in the AI Customer Ticket Triage system communicate with each other.

The system follows a hybrid communication model using:
- GraphQL for frontend APIs
- REST for inter-service communication
- Kafka or BullMQ for asynchronous processing (env-based switching)

---

## Communication Layers

### 1. Frontend → Backend (GraphQL)

The frontend (`ai-ticket-web-app-react-graphql`) communicates with the gateway (`ai-ticket-main-server`) using GraphQL.

#### Why GraphQL
- Flexible queries
- Reduces over-fetching
- Single endpoint
- Ideal for dashboards

#### Example
```graphql
query GetTickets {
  tickets {
    id
    status
    priority
    assignedTeam
  }
}
```

---

### 2. Gateway → Internal Services (REST)

All internal backend services communicate using REST APIs.

#### Flow
```
main-server → core-server → processor-server → llm-server
```

#### Characteristics
- Synchronous (request–response)
- JSON-based communication
- Used for CRUD and controlled operations

#### Example
```ts
await axios.post('http://core-server:3001/tickets', data);
```

---

### 3. Async Processing (Queue / Kafka)

The scheduler server handles background processing using either BullMQ or Kafka, based on environment configuration.

#### Environment Switch
```bash
QUEUE_TYPE=bullmq   # or kafka
```

---

## BullMQ Flow (Default)

```
core-server → BullMQ → scheduler-server → processor-server
```

- Job-based processing
- Redis-backed queue
- Suitable for moderate workloads

---

## Kafka Flow (Event-driven)

```
core-server → Kafka topic → scheduler-server / processor-server
```

- Event-driven architecture
- Multiple consumers
- High scalability

---

## Scheduler Responsibility

The `ai-ticket-scheduler-server` acts as:
- Queue consumer (BullMQ or Kafka)
- Job orchestrator
- Retry handler
- Cron/SLA processor

---

## Processor Responsibility

The `ai-ticket-processor-server`:
- Applies AI triage logic
- Calls LLM service via REST
- Returns structured output

---

## Hybrid Model Summary

| Layer | Technology | Type |
|------|-----------|------|
| Frontend API | GraphQL | Sync |
| Service-to-Service | REST | Sync |
| Background Processing | BullMQ / Kafka | Async |

---

## Key Design Decisions

- GraphQL used only for frontend interaction
- REST used for internal service communication for simplicity
- Kafka optional for advanced scalability
- BullMQ used as default for simplicity
- Environment-based switching avoids code changes

---

## One-line Summary

Frontend uses GraphQL, backend services communicate via REST, and asynchronous processing is handled by BullMQ or Kafka in the scheduler server using an environment-based switch.