<div align="center">

# 🛡️ FraudShield

### Real-Time Fraud Detection

**A platform for identifying financial fraud in real time.**

![Status](https://img.shields.io/badge/status-active-brightgreen?style=for-the-badge)
![License](https://img.shields.io/badge/license-MIT-blue?style=for-the-badge)
![PRs Welcome](https://img.shields.io/badge/PRs-welcome-ff69b4?style=for-the-badge)

![Apache Kafka](https://img.shields.io/badge/Apache%20Kafka-231F20?style=for-the-badge&logo=apachekafka&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)

🌐 **English** · [Português (BR)](README.pt-BR.md)

</div>

---

## 📚 Table of Contents

- [Overview](#-overview)
- [Features](#-features)
- [Architecture](#-architecture)
- [How Scoring Works](#-how-scoring-works)
- [Decisions](#-decisions)
- [Rules Engine](#-rules-engine)
- [Tech Stack](#-tech-stack)
- [Getting Started](#-getting-started)
- [Configuration](#-configuration)
- [API](#-api)
- [Case Management](#-case-management)
- [Model Lifecycle](#-model-lifecycle)
- [Monitoring](#-monitoring)
- [Privacy & Compliance](#-privacy--compliance)
- [Testing](#-testing)
- [Security](#-security)
- [Roadmap](#-roadmap)
- [Contributing](#-contributing)
- [License](#-license)

---

## 🧭 Overview

FraudShield evaluates every financial event (card payments, transfers, logins, account changes) as it happens and returns a risk decision in milliseconds. It combines a flexible **rules engine**, **machine learning models**, and **real-time behavioral features** so fraud can be stopped before the money moves, without blocking legitimate customers.

| | Pillar | What it delivers |
| :-: | --- | --- |
| ⚡ | **Real-time scoring** | A risk score and decision for each event, inside the payment flow. |
| 🧠 | **Hybrid detection** | Rules for known patterns, models for the unknown. |
| 🔍 | **Explainability** | Every decision comes with reason codes. |
| 🔄 | **Continuous learning** | Analyst feedback becomes training data. |

## 🚀 Features

### ⚡ Real-Time Scoring
- Synchronous scoring API designed for the critical path of a payment
- Event streaming ingestion for asynchronous signals
- Configurable timeout with a safe fallback decision

### 🧠 Detection
- **Velocity checks:** transaction counts and amounts over sliding windows
- **Behavioral profiling:** deviation from each customer's normal pattern
- **Device and session signals:** new device, emulator, proxy/VPN, risky IP
- **Geo analysis:** impossible travel and high-risk locations
- **Payee risk:** first-time recipients and suspected mule accounts
- **Account takeover signals:** credential changes followed by high-value transfers

### 🔍 Explainability
- Reason codes returned with every score
- Per-decision feature snapshot for audits and disputes
- Rule hit history

### 🕵️ Operations
- Case queue for analyst review
- Allow lists, block lists, and watch lists
- Champion/challenger and shadow-mode model testing

## 🏗️ Architecture

```mermaid
flowchart LR
  T["Transaction Event"] --> API["Scoring API"]
  T --> K{{"Event Stream"}}
  K --> FS[("Feature Store")]
  API --> FE["Feature Enrichment"]
  FS --> FE
  FE --> RE["Rules Engine"]
  FE --> ML["ML Models"]
  RE --> D{"Decision Engine"}
  ML --> D
  D -->|Approve| A["Approve"]
  D -->|Step-up| S["Challenge / MFA"]
  D -->|Review| C["Case Queue"]
  D -->|Decline| X["Decline"]
  C --> AN["Analyst"]
  AN --> L[("Labels")]
  L --> TR["Model Training"]
  TR --> ML
```

## 🧮 How Scoring Works

```mermaid
sequenceDiagram
  autonumber
  participant P as Payment System
  participant A as Scoring API
  participant F as Feature Store
  participant R as Rules Engine
  participant M as ML Model
  participant D as Decision Engine

  P->>A: POST /v1/score (event)
  A->>F: Fetch customer, device, payee features
  F-->>A: Real-time features
  par Evaluate in parallel
    A->>R: Evaluate rules
    A->>M: Predict fraud probability
  end
  R-->>D: Rule hits + score adjustments
  M-->>D: Model score
  D-->>A: Final score, decision, reason codes
  A-->>P: Response
  A-)F: Update features with the new event
```

## ⚖️ Decisions

Each event ends in one of four outcomes. The thresholds below are **example defaults** and are fully configurable.

| Score | Decision | Action |
| :-: | --- | --- |
| 0 – 29 | ✅ `APPROVE` | Let the transaction through |
| 30 – 59 | 🔐 `CHALLENGE` | Request step-up authentication (MFA, biometrics) |
| 60 – 84 | 👀 `REVIEW` | Hold and send to the analyst queue |
| 85 – 100 | ⛔ `DECLINE` | Block the transaction |

## 📏 Rules Engine

Rules are declarative, versioned, and can be tested in shadow mode before going live.

```yaml
- id: high-velocity-large-amount
  description: Many transactions in a short window, with a large amount
  when:
    all:
      - feature: txn_count_10m
        op: ">"
        value: 5
      - field: amount
        op: ">"
        value: 100000
  effect:
    score: 30
    reason_code: HIGH_VELOCITY
    action: REVIEW

- id: impossible-travel
  description: Two events from places too far apart for the elapsed time
  when:
    feature: geo_travel_speed_kmh
    op: ">"
    value: 900
  effect:
    score: 40
    reason_code: IMPOSSIBLE_TRAVEL
```

## 🛠️ Tech Stack

> Adjust this section to match your actual stack.

| Layer | Technology |
| --- | --- |
| API | REST / JSON, OpenAPI |
| Event streaming | Apache Kafka |
| Feature store (online) | Redis |
| Operational database | PostgreSQL |
| Model serving | Any runtime that exposes a prediction endpoint |
| Containers | Docker & Docker Compose |

## 🏁 Getting Started

### Prerequisites

- Docker and Docker Compose
- Git
- The runtime for your chosen language/framework

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/your-org/fraudshield.git
cd fraudshield

# 2. Create your environment file
cp .env.example .env

# 3. Start infrastructure (Kafka, Redis, PostgreSQL)
docker compose up -d

# 4. Install dependencies and run migrations
make setup
make migrate

# 5. Load sample rules and a baseline model
make seed

# 6. Start the scoring API and the stream processors
make run-api
make run-stream
```

The API will be available at `http://localhost:8080` and the docs at `http://localhost:8080/docs`.

### Try it

```bash
curl -X POST http://localhost:8080/v1/score \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d @examples/transfer-event.json
```

## ⚙️ Configuration

| Variable | Description | Example |
| --- | --- | --- |
| `DATABASE_URL` | Database connection string | `postgres://user:pass@localhost:5432/fraudshield` |
| `REDIS_URL` | Feature store connection string | `redis://localhost:6379` |
| `KAFKA_BROKERS` | Kafka bootstrap servers | `localhost:9092` |
| `MODEL_ENDPOINT` | URL of the model serving endpoint | `http://localhost:9000/predict` |
| `SCORING_TIMEOUT_MS` | Maximum time to produce a decision | `150` |
| `FALLBACK_DECISION` | Decision when scoring times out or fails | `REVIEW` |
| `THRESHOLD_CHALLENGE` | Minimum score for `CHALLENGE` | `30` |
| `THRESHOLD_REVIEW` | Minimum score for `REVIEW` | `60` |
| `THRESHOLD_DECLINE` | Minimum score for `DECLINE` | `85` |
| `WEBHOOK_SIGNING_SECRET` | Secret used to sign webhook payloads | *(strong random value)* |

> ⚠️ Never commit real secrets. Use a secrets manager in production.

## 📡 API

| Method | Endpoint | Description |
| --- | --- | --- |
| `POST` | `/v1/score` | Score an event and get a decision |
| `GET` | `/v1/decisions/{id}` | Retrieve a past decision with its features |
| `POST` | `/v1/feedback` | Submit a confirmed fraud / legitimate label |
| `GET` | `/v1/cases` | List cases awaiting review |
| `PATCH` | `/v1/cases/{id}` | Update or resolve a case |
| `POST` | `/v1/rules` | Create or update a rule |
| `GET` | `/v1/models` | List models and their status |
| `POST` | `/v1/models/{id}/promote` | Promote a model to production |

### Score an event

```http
POST /v1/score
Authorization: Bearer <token>
Content-Type: application/json

{
  "event_id": "evt_001",
  "event_type": "transfer",
  "timestamp": "2026-11-10T12:00:00Z",
  "customer_id": "cus_123",
  "amount": 250000,
  "currency": "BRL",
  "payee_id": "payee_456",
  "device": { "id": "dev_789", "ip": "203.0.113.10" }
}
```

```json
{
  "decision_id": "dec_abc123",
  "score": 78,
  "decision": "REVIEW",
  "reason_codes": ["NEW_PAYEE", "HIGH_AMOUNT", "NEW_DEVICE"],
  "model_version": "2026-11-01",
  "latency_ms": 42
}
```

> Amounts are integers in the **smallest currency unit** (e.g. cents).

### Webhooks

| Event | Sent when |
| --- | --- |
| `decision.review` | An event was sent to manual review |
| `decision.declined` | An event was declined |
| `case.resolved` | An analyst closed a case |
| `model.drift_detected` | Monitoring detected model drift |

## 🕵️ Case Management

```mermaid
flowchart LR
  R["Event held for review"] --> Q["Case queue"]
  Q --> A["Analyst investigates"]
  A -->|Confirmed fraud| F["Block and label as fraud"]
  A -->|Legitimate| L["Release and label as legitimate"]
  F --> D[("Training data")]
  L --> D
```

Analysts see the score, reason codes, customer history, and related events in one place. Every resolution creates a label that feeds model retraining.

## 🔄 Model Lifecycle

1. **Train** on labeled historical events
2. **Validate** offline with precision, recall, and cost-weighted metrics
3. **Shadow:** run alongside production without affecting decisions
4. **Champion/challenger:** compare against the current model on live traffic
5. **Promote** with a single call, **roll back** instantly if needed

## 📊 Monitoring

| Area | Metrics |
| --- | --- |
| Performance | Scoring latency (p50/p95/p99), throughput, timeout and fallback rate |
| Detection quality | Precision, recall, false-positive rate, approval rate |
| Model health | Score distribution, feature drift (e.g. PSI), label delay |
| Operations | Case queue size, average time to resolution |

Alerts fire on latency spikes, fallback surges, drift, and sudden changes in approval or decline rates.

## 🔐 Privacy & Compliance

- Personal data masked in logs and traces
- Field-level encryption for sensitive attributes
- Configurable data retention and deletion
- Designed to support **LGPD** and **GDPR** requirements
- Full audit trail of rule, model, and threshold changes

## 🧪 Testing

```bash
make test               # unit tests
make test-integration   # integration tests (infrastructure required)
make test-replay        # replay historical events against a rule or model version
make test-load          # latency and throughput under load
make coverage           # coverage report
```

Critical scenarios to keep covered: scoring timeouts and fallback behavior, feature store outages, out-of-order events, rule conflicts, and replaying known fraud patterns after every rule or model change.

## 🔒 Security

- TLS for all traffic and signed webhooks
- API authentication with scoped keys
- Role-based access for analysts, rule authors, and administrators
- Rate limiting on all endpoints
- Least-privilege access for services and database users

To report a vulnerability, please **do not** open a public issue. Contact: `security@your-org.com`.

## 🗺️ Roadmap

- [ ] Graph analysis for fraud rings and mule networks
- [ ] Device fingerprinting SDKs (web, iOS, Android)
- [ ] Unsupervised anomaly detection
- [ ] Analyst dashboard with investigation tools
- [ ] Consortium signals sharing

## 🤝 Contributing

Contributions are welcome!

1. Fork the repository
2. Create a branch: `git checkout -b feature/my-feature`
3. Commit your changes: `git commit -m "feat: add my feature"`
4. Push the branch: `git push origin feature/my-feature`
5. Open a Pull Request

## 📄 License

Licensed under the **MIT License**. See [LICENSE](LICENSE) for details.

<div align="center">

Made with 🛡️ to protect every transaction.

</div>
