# Scheduler & Processor Service Communication

## Overview

This document explains how the `ai-ticket-scheduler-server` and `ai-ticket-processor-server` communicate in the AI Customer Ticket Triage system.

The scheduler is responsible for consuming background jobs and triggering processing, while the processor performs AI-based triage logic.

---

## Communication Pattern

The system uses a hybrid approach:

- **Queue-based (asynchronous)** between Core → Scheduler
- **REST-based (synchronous)** between Scheduler → Processor

---

## Flow Description

### Step 1: Job Creation (Core Server)

When a ticket is created:

- Core server stores ticket in database
- Pushes job to queue

Example:
```ts
queue.add('process-ticket', { ticketId });
```

---

### Step 2: Job Consumption (Scheduler Server)

Scheduler server acts as a worker:

```ts
worker.process(async (job) => {
  await processorService.process(job.data.ticketId);
});
```

Responsibilities:
- Consume jobs
- Handle retries
- Manage failures

---

### Step 3: Call Processor Service (REST)

Scheduler calls processor via HTTP:

```ts
await axios.post('http://processor-server:3002/process', {
  ticketId: job.data.ticketId
});
```

---

### Step 4: Processor Logic

Processor server:
- Fetches ticket data
- Applies business rules
- Calls LLM service
- Generates structured output

Example output:
```json
{
  "category": "billing",
  "priority": "high",
  "sentiment": "frustrated",
  "assignedTeam": "finance-support",
  "confidence": 0.91,
  "needsHumanReview": false,
  "suggestedReply": "We are reviewing your issue.",
  "timestamp": "2026-05-05T14:04:00Z"
}
```

---

### Step 5: Update Core Server

Processor sends result back:

```ts
await axios.post('http://core-server:3001/tickets/update-triage', result);
```

---

## Communication Summary

| From | To | Method | Type |
|------|----|--------|------|
| Core Server | Scheduler | Queue | Async |
| Scheduler | Processor | REST (HTTP) | Sync |
| Processor | Core | REST (HTTP) | Sync |

---

## Error Handling

### Scheduler
- Retry failed jobs
- Move to dead-letter queue after max attempts

### Processor
- Validate input
- Handle LLM failures
- Return fallback response if needed

---

## Scalability Considerations

- Multiple scheduler workers can run in parallel
- Processor service can be horizontally scaled
- Queue ensures load buffering

---

## Optional Kafka Upgrade

Replace queue layer:

```
Core → Kafka → Scheduler/Processor
```

Benefits:
- Event-driven architecture
- Multiple consumers
- High scalability

---

## Key Design Principles

- Loose coupling via queue
- Clear separation of responsibilities
- Async processing for heavy tasks
- REST for controlled service interaction

---

## Hybrid Queue Strategy (Env-based Switching)

The system supports switching between BullMQ (Redis) and Kafka using an environment variable. This allows flexibility between a simple job-queue model and an event-driven architecture.

### Environment Variable

```bash
QUEUE_TYPE=bullmq   # or kafka
```

---

### Abstraction Interface

```ts
export interface QueueClient {
  publishTicketCreated(data: { ticketId: string }): Promise<void>;
}
```

---

### BullMQ Implementation

```ts
import { Queue } from 'bullmq';

export class BullMQClient implements QueueClient {
  private queue = new Queue('ticket');

  async publishTicketCreated(data: { ticketId: string }) {
    await this.queue.add('process-ticket', data);
  }
}
```

---

### Kafka Implementation

```ts
import { Kafka } from 'kafkajs';

export class KafkaClient implements QueueClient {
  private producer;

  constructor() {
    const kafka = new Kafka({ brokers: ['localhost:9092'] });
    this.producer = kafka.producer();
  }

  async publishTicketCreated(data: { ticketId: string }) {
    await this.producer.send({
      topic: 'ticket.created',
      messages: [{ value: JSON.stringify(data) }],
    });
  }
}
```

---

### Factory (Env Switch)

```ts
import { BullMQClient } from './bullmq.client';
import { KafkaClient } from './kafka.client';

export function createQueueClient(): QueueClient {
  const type = process.env.QUEUE_TYPE;

  if (type === 'kafka') {
    return new KafkaClient();
  }

  return new BullMQClient();
}
```

---

### Usage

```ts
const queueClient = createQueueClient();
await queueClient.publishTicketCreated({ ticketId: 't1' });
```

---

### Key Notes

- Both implementations must follow the same interface
- Avoid spreading conditional logic across services
- Initialize Kafka producer once at startup
- Maintain consistent event/job naming

---

## One-line Summary

Scheduler consumes queued jobs (BullMQ or Kafka via env-based switch) and triggers the processor service via REST, which performs AI-based triage and updates the core ticket system.