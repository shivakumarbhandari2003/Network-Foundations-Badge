# 💼 Smart Expense Approval Platform
### *Enterprise-Grade, Event-Driven & Scalable Network Architecture*

[![Network Foundations](https://img.shields.io/badge/Network%20Foundations-Graded%20Project-007acc?style=for-the-badge&logo=cisco&logoColor=white)](#-network-foundations-mapping)
[![Architecture](https://img.shields.io/badge/Architecture-Event--Driven%20%7C%20Microservices-blueviolet?style=for-the-badge&logo=apachekafka&logoColor=white)](#-proposed-solution-architecture)
[![Security](https://img.shields.io/badge/Security-HTTPS%20%2F%20TLS%201.3-success?style=for-the-badge&logo=letsencrypt&logoColor=white)](#-network-foundations-mapping)
[![Deployment](https://img.shields.io/badge/Containers-Docker%20%26%20Kubernetes-326ce5?style=for-the-badge&logo=kubernetes&logoColor=white)](#-key-components--technology-stack)

---

## 📖 Table of Contents
- [📌 Executive Summary](#-executive-summary)
- [🚨 Problem Statement & Legacy Bottlenecks](#-problem-statement--legacy-bottlenecks)
- [🏗️ Proposed Solution Architecture](#-proposed-solution-architecture)
- [🧩 Key Components & Technology Stack](#-key-components--technology-stack)
- [🔄 End-to-End Request & Event Lifecycle](#-end-to-end-request--event-lifecycle)
- [🌐 Network Foundations Mapping](#-network-foundations-mapping)
- [🔗 Problem vs. Solution Impact Matrix](#-problem-vs-solution-impact-matrix)
- [📋 System Assumptions & Production Readiness](#-system-assumptions--production-readiness)
- [🌟 Key Architectural Benefits](#-key-architectural-benefits)

---

## 📌 Executive Summary

The **Smart Expense Approval Platform** is a distributed, high-throughput enterprise solution designed to streamline employee expense submissions and real-time managerial approvals.

Transitioning from a constrained monolithic infrastructure, this modernized architecture leverages **Layer 7 Load Balancing, Containerization, Distributed Event Streaming (Apache Kafka), Full-Duplex WebSockets, and Microservices** to guarantee sub-second latencies, fault tolerance, and zero-loss notification delivery under high concurrency.

### 🎯 Core Objectives
* ⚡ **Ultra-Low Latency:** Eliminate submission stalls via asynchronous event offloading.
* 🔔 **Guaranteed Delivery:** Instant, real-time manager alerts powered by WebSockets and message queues.
* 📈 **Elastic Scalability:** Independent horizontal auto-scaling of decoupled microservices.
* 🔐 **Zero-Trust Security:** Strict HTTPS/TLS encryption across all ingress and internal boundaries.
* 🟢 **High Availability (99.99%):** Eliminate single points of failure (SPOF) across services and databases.

---

## 🚨 Problem Statement & Legacy Bottlenecks

Under peak corporate cycles (e.g., month-end expense filings), the legacy monolithic architecture suffered severe operational degradation:

```
[Legacy Monolith]
Employees (HTTPS) ──▶ [ Monolithic App Server (All-in-One) ] ──▶ [ Single DB Instance ] ──▶ Manager (Polling/Delayed)
                      (SPOF • Sync Processing • CPU Bound)     (Connection Exhaustion)
```

### Critical Failure Vectors:
1. **Synchronous Submission Stalls:** Expense submission, database persistence, and notification dispatch executed on a single thread blocking the HTTP client.
2. **Notification Drops & Delays:** Reliance on direct transactional email/push led to missed managerial approvals and timeouts.
3. **Database Connection Exhaustion:** Simultaneous submissions saturated the single database connection pool, causing 504 Gateway Timeouts.
4. **Single Point of Failure (SPOF):** Any localized crash in notification logic brought down the entire authentication and submission engine.

---

## 🏗️ Proposed Solution Architecture

The revised architecture adopts a decoupled, event-driven microservices pattern with edge load balancing and real-time streaming:

```mermaid
flowchart TD
    subgraph Clients ["👥 Client Layer"]
        EMP["👨‍💼 Employees (Mobile / Web)"]
        MGR["👩‍💼 Manager Dashboard"]
    end

    subgraph Edge ["🌐 Edge & Ingress Layer"]
        ALB["⚖️ AWS Application Load Balancer (ALB)\nHTTPS / TLS Termination (Port 443)"]
    end

    subgraph CoreServices ["🐳 Containerized Microservices (Kubernetes Cluster)"]
        EXP1["💼 Expense Service - Pod 1"]
        EXP2["💼 Expense Service - Pod 2"]
        EXP3["💼 Expense Service - Pod 3"]
    end

    subgraph Storage ["🗄️ Persistence Layer"]
        DB[("🗄️ PostgreSQL\nPrimary-Replica Cluster")]
    end

    subgraph EventStream ["📨 Event Backbone (Apache Kafka)"]
        TOPIC["Topic: expense-events\n• ExpenseSubmitted\n• ExpenseApproved\n• ExpenseRejected"]
    end

    subgraph Consumers ["⚙️ Asynchronous Event Consumers"]
        NOTIF["🔔 Notification Service\n(WebSocket Gateway)"]
        APPV["✅ Approval Engine\n(State Machine)"]
        AUDIT["📋 Audit & Compliance Service\n(Immutable Log)"]
    end

    %% Ingress Flow
    EMP -->|HTTPS / TLS 1.3| ALB
    ALB -->|Round Robin / Least Conn| EXP1
    ALB -->|Round Robin / Least Conn| EXP2
    ALB -->|Round Robin / Least Conn| EXP3

    %% Core Service Actions
    EXP1 & EXP2 & EXP3 -->|ACID Persistence| DB
    EXP1 & EXP2 & EXP3 -->|Produce Event| TOPIC

    %% Event Consumption
    TOPIC -->|Subscribe| NOTIF
    TOPIC -->|Subscribe| APPV
    TOPIC -->|Subscribe| AUDIT

    %% Real-time Push & Actions
    NOTIF -->|WebSocket Stream (WSS)| MGR
    MGR -->|Approve / Reject via HTTPS| ALB
```

---

## 🧩 Key Components & Technology Stack

| Layer | Technology | Architectural Role | Network Protocol / Spec |
| :--- | :--- | :--- | :--- |
| **Edge Ingress** | AWS ALB | SSL Termination, path routing, health checks | `HTTPS / TLS 1.3`, `HTTP/2` |
| **Compute & Runtime** | Docker & Kubernetes | Container isolation, horizontal pod autoscaling | Overlay Network (CNI), `TCP` |
| **Core Ingestion** | Expense Service | Validates and ingests claims synchronously | RESTful API (`JSON / HTTP/2`) |
| **Message Broker** | Apache Kafka | Distributed event log, message queuing | Binary TCP Protocol (Port 9092) |
| **Real-Time Push** | Notification Gateway | Bi-directional manager state synchronization | `WebSocket (WSS)` |
| **Internal RPC** | gRPC *(Optional)* | High-performance synchronous inter-service calls | `HTTP/2` over `gRPC / Protobuf` |
| **Data Tier** | PostgreSQL Cluster | Relational transactional persistence | `TCP / IP` (Connection Pooling) |

---

## 🔄 End-to-End Request & Event Lifecycle

```
[1. Submit]  Employee ──(HTTPS/TLS)──▶ ALB ──▶ Expense Service (Pod)
                                                    │
[2. Persist]                                        ├──▶ PostgreSQL (Commit Record)
                                                    │
[3. Publish]                                        └──▶ Kafka Topic: ExpenseSubmitted
                                                                │
[4. Fan-Out]                ┌───────────────────────────────────┼──────────────────────────────────┐
                            ▼                                   ▼                                  ▼
                 [Notification Service]                 [Approval Engine]                 [Audit Service]
                            │                                   │                                  │
[5. Real-Time Push]         ▼ (WSS Push)                        ▼                                  ▼
                     Manager Dashboard                 Evaluate Rules & SLAs               Append Audit Trail
                            │
[6. Approval Cycle]         └──▶ Manager clicks 'Approve' ──(HTTPS)──▶ Kafka (ExpenseApproved)
```

1. **Secure Ingress:** The employee initiates an expense claim over **HTTPS/TLS 1.3**.
2. **Dynamic Load Distribution:** AWS ALB inspects health and routes the payload to the least-utilized Expense Service container.
3. **Optimistic Persistence:** The Expense Service writes the draft claim to PostgreSQL with an ACID-compliant transaction.
4. **Non-Blocking Event Publication:** The service publishes an `ExpenseSubmitted` event to the `expense-events` Kafka topic and immediately returns `202 Accepted` to the client, keeping HTTP latency under 50ms.
5. **Parallel Asynchronous Consumption:**
   * **Notification Service** consumes the event and broadcasts a live alert to the assigned manager over an active **WebSocket (WSS)** connection.
   * **Approval Service** checks compliance thresholds and escalates if required.
   * **Audit Service** appends an immutable compliance entry.
6. **Decision Dispatch:** The manager approves/rejects the claim from their dashboard, generating an `ExpenseApproved`/`ExpenseRejected` event for downstream payroll and ledger processing.

---

## 🌐 Network Foundations Mapping

| Network Concept | Platform Implementation | Operational Impact |
| :--- | :--- | :--- |
| **OSI Layer 4 (Transport)** | TCP connection pooling, Keep-Alive, congestion control | Efficient socket reuse, lower handshake overhead |
| **OSI Layer 7 (Application)** | HTTP/2, REST APIs, JSON data formatting | Multiplexing, header compression, granular routing |
| **Transport Layer Security** | TLS 1.3 cipher suites, automated certificate renewal | Confidentiality, data integrity, prevention of MITM attacks |
| **Layer 7 Load Balancing** | AWS Application Load Balancer with health checks | Even traffic distribution, seamless failover |
| **Full-Duplex Streaming** | Persistent WebSocket (`wss://`) channels | Instant push notifications; eliminates wasteful HTTP polling |
| **Event Streaming Protocol** | Kafka binary protocol over TCP | High-throughput, partitioned consumer group scaling |
| **Microservices RPC** | gRPC with Protocol Buffers | Low-latency, strongly-typed internal service communications |

---

## 🔗 Problem vs. Solution Impact Matrix

| Legacy Pain Point | Modern Solution Implementation | Measurable Architectural Outcome |
| :--- | :--- | :--- |
| ⏱️ **Slow Expense Submissions** | Decoupled async processing + Kafka event offloading | ⚡ **90% reduction** in client-perceived response time |
| 🔔 **Missed Manager Updates** | WebSocket streaming + reliable broker queues | 🎯 **100% reliable** alert delivery with zero page refreshes |
| 📉 **High-Load Responsiveness** | Multi-instance containers + AWS ALB distribution | 🚀 **Zero request drops** during traffic spikes |
| 🔐 **Data Privacy Vulnerabilities** | End-to-end HTTPS/TLS encryption & RBAC | 🛡️ **Zero cleartext data** across external & internal networks |
| ⚠️ **Single Server Dependency** | Multi-AZ Kubernetes pod deployment | 🟢 **Zero single points of failure (99.99% Availability)** |

---

## 📋 System Assumptions & Production Readiness

* **Zero-Trust Network:** All traffic crossing the boundary terminates TLS at the ingress controller, with mTLS enabled for sensitive inter-service mesh communication.
* **Idempotent Consumers:** Kafka consumer services implement idempotency keys to guarantee exactly-once processing semantics during retries.
* **Multi-AZ Resilience:** Microservice replicas and PostgreSQL instances are distributed across multiple Availability Zones (AZs) for instant disaster recovery.
* **Observability by Design:** Distributed tracing (OpenTelemetry), centralized logging (ELK / CloudWatch), and Prometheus/Grafana metrics monitor end-to-end system health.

---

## 🌟 Key Architectural Benefits

```
      ┌────────────────────────────────────────────────────────┐
      │         SMART EXPENSE APPROVAL PLATFORM               │
      │                                                        │
      │   ⚡ High Performance  •  📈 Elastic Scalability       │
      │   🔔 Real-Time Sync   •  🔐 Military-Grade Security    │
      │   🟢 High Availability •  🔗 Loosely Coupled Events    │
      └────────────────────────────────────────────────────────┘
```

* **⚡ Ultra-Low Latency:** Asynchronous event delegation keeps UI interactions instantaneous.
* **📈 Elastic Scalability:** Containerized pods scale horizontally in response to CPU/memory/queue-depth triggers.
* **🔔 Real-Time Event Sync:** Instant managerial updates eliminate approval bottlenecks.
* **🔐 End-to-End Security:** Industry-standard encryption safeguards all financial and personal records.
* **🛠️ Maintainability & Modularity:** Individual microservices can be deployed, tested, and upgraded independently without downtime.

---

## 🏷️ Metadata & Keywords

`Network Foundations` • `TCP/IP` • `HTTPS` • `TLS 1.3` • `AWS ALB` • `Load Balancing` • `Docker` • `Kubernetes` • `Apache Kafka` • `WebSocket` • `gRPC` • `Microservices` • `Distributed Systems` • `Event-Driven Architecture` • `High Availability`
