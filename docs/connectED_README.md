# connectED — University Mentorship Knowledge Graph & Automated Matchmaking Engine

> **Distributed alumni-student mentorship network combining a FastAPI graph backend (Neo4j / NetworkX), KCD-v1 heuristic matchmaking, and a 31 KB stateful n8n Telegram Bot automation workflow.**

[![Status](https://img.shields.io/badge/Status-Architecture%20Preview-00D9FF?style=flat-square)](#)
[![Backend](https://img.shields.io/badge/Backend-FastAPI-009688?style=flat-square&logo=fastapi)](#)
[![Graph](https://img.shields.io/badge/Graph-Neo4j%20%2F%20NetworkX-blue?style=flat-square&logo=neo4j)](#)
[![Automation](https://img.shields.io/badge/Automation-n8n%20Workflows-EA4B71?style=flat-square&logo=n8n)](#)
[![Messaging](https://img.shields.io/badge/Bot-Telegram%20Bot%20API-2CA5E0?style=flat-square&logo=telegram)](#)

---

## 1. Problem Statement

University alumni networks suffer from extreme friction:
- **Asymmetric Engagement:** A tiny fraction of famous alumni receive hundreds of outreach requests, while thousands of enthusiastic recent graduates are never discovered.
- **High Friction & Cold Drop-Off:** Over 70% of initial student outreach messages receive no reply due to lack of SLA tracking or structured agendas.
- **Siloed Relational Data:** Relational tables (SQL) struggle to efficiently query multi-hop relational patterns, such as *"Find mentors who studied Computer Science, work at AI companies in Europe, and share an interest in open-source robotics."*

---

## 2. System Architecture

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                            connectED PIPELINE                               │
└─────────────────────────────────────────────────────────────────────────────┘

    [ Student / Alumni Onboarding ]
                  │
                  ▼
         [ FastAPI REST Gateway ]
                  │
                  ├───────────────────────────────┐
                  ▼                               ▼
       [ Neo4j Knowledge Graph ]         [ KCD-v1 Matcher ]
       Nodes: Alumni, Student, Domain    Weighted Cosine / Jaccard
       Edges: MENTORS, EXPERIENCED_IN    Top-K Ranked Mentor List
                  │                               │
                  └──────────────┬────────────────┘
                                 │
                                 ▼
                    [ 31 KB n8n Automation Bus ]
                    ├── Interactive Telegram Bot Nodes
                    ├── Profile Verification Filter
                    ├── Match Notification Dispatch
                    └── 48-Hour SLA Escalation Timer
```

---

## 3. The KCD-v1 Matchmaking Algorithm

The matching engine computes a multi-factor composite alignment score between a student profile $S$ and candidate mentor profile $M$:

$$	ext{Score}(S, M) = w_1 \cdot J(	ext{Skills}_S, 	ext{Skills}_M) + w_2 \cdot 	ext{GraphDistance}(S_{	ext{dept}}, M_{	ext{dept}}) + w_3 \cdot 	ext{Bandwidth}(M)$$

Where:
- $J(A, B) = rac{|A \cap B|}{|A \cup B|}$ represents the Jaccard similarity across declared technical tags.
- $	ext{GraphDistance}$ accounts for cross-departmental affinity in the Neo4j ontology.
- $	ext{Bandwidth}(M)$ penalizes mentors with active concurrent mentees to prevent mentor burnout.

---

## 4. The 31 KB n8n Telegram Automation Bus

The orchestration backbone is an enterprise **n8n workflow (31,482 bytes)** that operates 24/7 without requiring manual human oversight:
1. **Webhook Ingestion:** Receives real-time registration webhooks from the FastAPI backend.
2. **Telegram Bot Dispatch:** Sends rich interactive messages with inline keyboard buttons (`[Accept Mentorship]`, `[Decline]`, `[View Profile]`).
3. **48-Hour SLA Timer:** A stateful wait node tracks the response timestamp. If the mentor does not respond within 48 hours, the request is automatically re-routed to the secondary ranked mentor, ensuring zero orphaned requests.

---

## 5. Neo4j Graph Schema

```cypher
// Sample Graph Query: Discover High-Affinity Mentors
MATCH (s:Student {id: $student_id})-[:INTERESTED_IN]->(d:Domain)<-[:SPECIALIZES_IN]-(m:Alumni)
WHERE m.active_mentees < m.max_capacity
MATCH (s)-[:HAS_SKILL]->(skill:Skill)<-[:HAS_SKILL]-(m)
RETURN m.name AS Mentor, count(skill) AS SharedSkills, d.name AS Domain
ORDER BY SharedSkills DESC
LIMIT 5;
```

---

## 6. Repository Roadmap & Refactoring

> **Note on Repository Naming:** The initial prototype code for this project was hosted under temporary repository names (`Temp` and `AISLOP` on GitHub). In accordance with the portfolio refactoring roadmap, this codebase is being consolidated into a clean, unified public repository: `connectED-system`.
