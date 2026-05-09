# Agent Architecture Documentation

## 🤖 Agent Overview

| Agent | Role | Communication |
|-------|------|---------------|
| **Orchestrator** | Coordinator & planner | Calls all agents via HTTP |
| **Classifier** | Categorizes tickets | HTTP → Orchestrator |
| **Priority** | Sets urgency level | HTTP → Orchestrator |
| **Sentiment** | Analyzes customer emotion | HTTP → Orchestrator |
| **Router** | Assigns team/agent | HTTP → Orchestrator |
| **Reply** | Generates response drafts | HTTP → Orchestrator |
| **Escalation** | Flags human intervention | HTTP → Orchestrator |
| **QA** | Reviews AI decisions | HTTP → Orchestrator |
| **Learning** | Improves from feedback | Listens to event bus |

---

## 📋 Agent Details

### 1. Orchestrator Agent
- **Input:** Raw ticket (subject + message + metadata)
- **Output:** Complete triage decision
- **Communication:** Calls all other agents synchronously
- **Pattern:** Plan → Delegate → Aggregate → Decide

```yaml
endpoint: POST /orchestrate
timeout: 5s
retry: 3 attempts
```

### 2. Classifier Agent
- **Input:** Ticket text
- **Output:** `{category, subcategory, intent, confidence}`
- **Communication:** Called by Orchestrator

```yaml
endpoint: POST /classify
timeout: 2s
```

### 3. Priority Agent
- **Input:** Ticket + customer context
- **Output:** `{priority: critical/high/medium/low, score, reasoning}`
- **Communication:** Called by Orchestrator

```yaml
endpoint: POST /prioritize
timeout: 1.5s
```

### 4. Sentiment Agent
- **Input:** Ticket message + history
- **Output:** `{primary_emotion, churn_risk, urgency_override}`
- **Communication:** Called by Orchestrator

```yaml
endpoint: POST /sentiment
timeout: 1s
```

### 5. Router Agent
- **Input:** Category, priority, team skills
- **Output:** `{assigned_team, assigned_agent, wait_time}`
- **Communication:** Called by Orchestrator

```yaml
endpoint: POST /route
timeout: 1s
```

### 6. Reply Agent
- **Input:** Ticket + classification + context
- **Output:** `{suggested_reply, confidence, alternatives}`
- **Communication:** Called by Orchestrator

```yaml
endpoint: POST /generate-reply
timeout: 3s
```

### 7. Escalation Agent
- **Input:** All agent outputs + confidence scores
- **Output:** `{needs_escalation: bool, triggers: [], level}`
- **Communication:** Called by Orchestrator

```yaml
endpoint: POST /should-escalate
timeout: 500ms
```

### 8. QA Agent
- **Input:** Original ticket + AI decisions
- **Output:** `{score, decision: approve/flag/reject, feedback}`
- **Communication:** Called by Orchestrator (when confidence < 0.8)

```yaml
endpoint: POST /review
timeout: 2s
```

### 9. Learning Agent
- **Input:** Human override data (async)
- **Output:** None (updates models)
- **Communication:** Listens to `override.events` queue

```yaml
event: override.created
consumer: learning-agent
batch: weekly retraining
```

---

## 🔄 Communication Flow

```
Customer Ticket
       ↓
┌─────────────────────────────────────┐
│      Orchestrator Agent             │
│    (Receives & Plans)               │
└─────────────────────────────────────┘
       ↓
   ┌───┴───┬───────┬────────┐
   ↓       ↓       ↓        ↓
Classifier Priority Sentiment Router
   │       │       │        │
   └───┬───┴───────┴────────┘
       ↓
┌─────────────────────────────────────┐
│         Reply Agent                 │
│    (Generates response)             │
└─────────────────────────────────────┘
       ↓
┌─────────────────────────────────────┐
│      Escalation Agent               │
│   (Safety check)                    │
└─────────────────────────────────────┘
       ↓
   High Confidence? 
    ↓ Yes        ↓ No
   Send      ┌─────────────┐
             │  QA Agent   │
             │ (Review)    │
             └─────────────┘
                  ↓
            Human Agent
                  ↓
┌─────────────────────────────────────┐
│       Learning Agent                │
│   (Listens & improves)              │
└─────────────────────────────────────┘
```

---

## 📡 Communication Protocols

| Agent Type | Protocol | Data Format | Pattern |
|------------|----------|-------------|---------|
| All analysis agents | HTTP/REST | JSON | Sync request-response |
| Orchestrator ↔ Agents | HTTP | JSON | Sync |
| Learning Agent | Message Queue | JSON | Async (pub-sub) |
| Agent ↔ External LLMs | HTTPS | JSON | Sync with timeout |

---

## 🔌 API Communication Examples

### Orchestrator calling Classifier
```http
POST /classify
{
  "ticket_id": "T-1234",
  "message": "I was charged twice",
  "subject": "Duplicate charge"
}

Response:
{
  "category": "billing",
  "confidence": 0.94
}
```

### Orchestrator calling Router
```http
POST /route
{
  "category": "billing",
  "priority": "high",
  "required_skills": ["refund", "payment"]
}

Response:
{
  "assigned_team": "finance-support",
  "assigned_agent": "agent_456"
}
```

### Learning Agent listening
```javascript
// Subscribes to override events
queue.subscribe('override.created', async (event) => {
  const { ticket_id, ai_decision, human_decision } = event.data;
  await learningAgent.learn(ticket_id, ai_decision, human_decision);
});
```

---

## ⚙️ Agent Configuration

```yaml
# docker-compose.agents.yml
agents:
  orchestrator:
    port: 8001
    replicas: 3
    timeout: 5s
    
  classifier:
    port: 8002
    replicas: 5
    model: gpt-3.5-turbo
    
  priority:
    port: 8003
    replicas: 3
    model: gpt-4
    
  router:
    port: 8004
    replicas: 2
    
  reply:
    port: 8005
    replicas: 10
    model: gpt-4-turbo
```

---

## 📊 Summary

| Agent | Sync/Async | Called By | Calls |
|-------|------------|-----------|-------|
| Orchestrator | Sync | Gateway | All agents |
| Classifier | Sync | Orchestrator | None |
| Priority | Sync | Orchestrator | Customer DB |
| Sentiment | Sync | Orchestrator | None |
| Router | Sync | Orchestrator | Team DB |
| Reply | Sync | Orchestrator | Vector DB |
| Escalation | Sync | Orchestrator | None |
| QA | Sync | Orchestrator | None |
| Learning | Async | Event bus | Training pipeline |

---

**Document Version:** 1.0  
**Last Updated:** May 2026