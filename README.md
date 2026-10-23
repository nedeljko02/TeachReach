# TeachReach

> A social network connecting experts with those who need their knowledge.

TeachReach automatically matches people who **offer** a service or tutoring with people who are **looking** for one, based on interest tags. Service descriptions are classified into topics by an LLM, so a provider writes what they do in plain language and the platform routes it to the right audience.

> **Status:** MVP concept — product definition and AWS system design. No implementation yet; this repository documents the proposed architecture. The full pitch deck is in [`TeachReach_MVP_presentation.pdf`](TeachReach_MVP_presentation.pdf).

---

## The problem

Knowledge exchange today happens across scattered Facebook groups, classifieds and word of mouth. Both sides lose:

**Those who offer knowledge** struggle to reach people who actually need what they know. Advertising is spread across dozens of platforms, and there is no structured way to present what they offer.

**Those who need knowledge** don't know who to turn to. They search chaotically, with no filtering by area of interest, and waste time before finding the right person.

## The solution

A single platform built around three steps:

1. **Sign-up & profile** — authentication via AWS Cognito, user selects interest tags during registration
2. **Smart classification** — AWS Bedrock reads a service description and assigns it a topic tag automatically
3. **Matching** — services surface to users whose interest tags match; request → acceptance → contact

---

## Features

**Sign-up and login** — AWS Cognito handles user management, with email notification of registration status.

**Interest profile** — users select one or more tags during registration, which drives everything they see afterwards.

**Service creation** — a provider writes a description and either picks a tag manually or lets AWS Bedrock classify it automatically.

**Personalised recommendations** — the feed is filtered by the user's interest tags rather than showing everything.

**Request system** — a requester sends a service request, the provider is notified, and on acceptance the requester is notified back so the two can connect.

## User flow

```
Sign-up + tags  →  Creates service  →  Bedrock classification
                                                │
                                                ▼
Notification  ←  Service request  ←  Recommended to other users
 to provider
      │
      └──→  acceptance  →  notification to requester
```

---

## Proposed architecture

Fully serverless on AWS. The frontend talks to a single REST layer; all business logic runs in Lambda.

```
Frontend (Web/Mobile)
        │
        ▼
   API Gateway ──────→ Cognito (auth)
        │
        ▼
     Lambda ─────────→ Bedrock (classification)
    (logic)
        │
        ├──────→ DynamoDB
        │
        └──────→ SES / SNS (notifications)
```

### AWS services

| Service | Role |
|---|---|
| **Cognito** | Sign-up, login, user management |
| **API Gateway** | REST API layer |
| **Lambda** | Business logic (serverless) |
| **DynamoDB** | Database — users, services, requests |
| **Bedrock** | Classifies a service description into a tag |
| **SES** | Email notifications for registration status and requests |
| **SNS** | Push notifications for requests |

### Data model

Single-table DynamoDB design. A global secondary index on the tag makes tag-based recommendations a single query instead of a scan.

| Entity | PK / SK | GSI |
|---|---|---|
| User | `USER#id` | `TAG#interest` |
| Service | `SERVICE#id` | `TAG#category` |
| Request | `REQUEST#id` | `USER#id (status)` |

Request status transitions: `pending → accepted` / `declined`

---

## Team

- Bogdan Micić
- Luka Kizić
- Luka Nedeljković
