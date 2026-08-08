# TechPulse AI — System Prompts & Architecture Specification

## Overview
TechPulse AI is an autonomous AI employee persona specializing in emerging technology research, objective scoring, multi-criteria evaluations, and structured editorial publishing.

---

## 1. System Prompt & Persona Definition

```
You are TechPulse AI, an elite Autonomous Emerging Technology Analyst and Research Persona.

### Core Persona Directives:
1. AUTONOMOUS OPERATION: Continuously scan, evaluate, and synthesize research across Artificial Intelligence, Robotics, Edge Computing, Hardware, and Open Source AI.
2. RIGOROUS EVALUATION: Grade candidate topics across 8 weighted criteria (Recency, Innovation, Engineering Value, Educational Value, Source Credibility, Novelty, Community Interest, Persona Alignment). Reject any topic falling below the minimum threshold score (default: 70/100).
3. EDITORIAL INTEGRITY: Every published insight must include technical analysis, future implications, grounded source attribution, and structured rationale explaining why this topic was chosen over alternatives.
4. MEMORY & CONTINUITY: Store long-term knowledge graphs to detect duplications, reference historical posts, and build continuous domain expertise across cycles.
```

---

## 2. Zod Schema Validation

All incoming HTTP request bodies and query parameters are validated server-side in `server.ts` using **Zod** schemas to prevent parameter tampering, malformed inputs, and injection attacks.

### Request Validation Schemas:

- **Agent Initialization Schema (`InitAgentSchema`)**:
  ```ts
  const InitAgentSchema = z.object({
    agentId: z.string().trim().min(1).max(100).optional(),
    intervalMinutes: z.coerce.number().min(1).max(1440).optional(),
    minScoreThreshold: z.coerce.number().min(0).max(100).optional(),
  });
  ```

- **Agent Manual Trigger Schema (`TriggerAgentSchema`)**:
  ```ts
  const TriggerAgentSchema = z.object({
    agentId: z.string().trim().min(1).max(100).optional(),
  });
  ```

- **Agent Configuration Schema (`ConfigAgentSchema`)**:
  ```ts
  const ConfigAgentSchema = z.object({
    intervalMinutes: z.coerce.number().min(1).max(1440).optional(),
    minScoreThreshold: z.coerce.number().min(0).max(100).optional(),
  });
  ```

- **Feed Query Schema (`FeedQuerySchema`)**:
  ```ts
  const FeedQuerySchema = z.object({
    agentId: z.string().trim().min(1).max(100).optional(),
  });
  ```

- **Candidates Query Schema (`CandidateQuerySchema`)**:
  ```ts
  const CandidateQuerySchema = z.object({
    status: z.string().trim().optional(),
  });
  ```

- **Logs Query Schema (`LogsQuerySchema`)**:
  ```ts
  const LogsQuerySchema = z.object({
    limit: z.coerce.number().min(1).max(500).optional(),
  });
  ```

---

## 3. Database Architecture & Persistence

TechPulse AI utilizes a hybrid storage architecture:

1. **Local SQLite (`techpulse.sqlite`)**: Primary low-latency relational datastore for local runtime cycles, candidate logging, memory nodes, and execution logs.
2. **Cloud Firebase Firestore (`astral-lens-x9v0l`)**: Cloud-synchronized document store provisioned in region `asia-southeast1`. Managed with fine-grained Attribute-Based Access Control (ABAC) in `firestore.rules`.
3. **Supabase Identity & Authentication**: Configurable via `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY` for multi-tenant editorial authorization and user management.

---

## 4. API Specification Endpoints

| Method | Endpoint | Description | Validation |
| :--- | :--- | :--- | :--- |
| `POST` | `/api/agent/init` | Bootstraps agent persona and scheduler | `InitAgentSchema` |
| `GET` | `/api/agent/feed` | Retrieves published articles and research posts | `FeedQuerySchema` |
| `POST` | `/api/agent/trigger` | Triggers an immediate autonomous research cycle | `TriggerAgentSchema` |
| `GET` | `/api/agent/candidates` | Lists evaluated candidate topics and status | `CandidateQuerySchema` |
| `GET` | `/api/agent/memory` | Retrieves agent long-term knowledge graph | N/A |
| `GET` | `/api/agent/logs` | Retrieves system action logs | `LogsQuerySchema` |
| `POST` | `/api/agent/config` | Updates cycle intervals & score thresholds | `ConfigAgentSchema` |
| `GET` | `/api/health` | Service health status check | N/A |

---

## 5. Security & Rule Hardening

- **Firestore Rules**: Enforces payload size caps, strict string type validation, regex pattern matching on IDs (`^[a-zA-Z0-9_\-]+$`), and write protection.
- **Environment Safety**: Secret keys and OAuth credentials are kept server-side and accessed securely via environment variables.
