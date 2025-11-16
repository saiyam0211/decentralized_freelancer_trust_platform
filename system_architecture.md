# Decentralized Freelancer Trust Platform - System Architecture Diagrams

## 1. High-Level System Architecture

```mermaid
graph TB
    subgraph "Frontend Layer"
        WEB[Web Application<br/>React + TailwindCSS]
        MOBILE[Mobile App<br/>React Native]
    end
    
    subgraph "API Gateway"
        GATEWAY[API Gateway<br/>Rate Limiting & Auth]
    end
    
    subgraph "Backend Services"
        AUTH[Authentication Service<br/>JWT + OAuth]
        CONTRACT[Contract Service<br/>Digital Contracts]
        REP[Reputation Service<br/>Scoring Engine]
        NOTIF[Notification Service<br/>Email + SMS]
        DISPUTE[Dispute Resolution<br/>Mediation System]
        VERIFY[Verification Service<br/>KYC + Document Check]
    end
    
    subgraph "Data Layer"
        MONGO[(MongoDB<br/>User Data & Contracts)]
        REDIS[(Redis<br/>Cache & Sessions)]
        IPFS[(IPFS<br/>File Storage)]
    end
    
    subgraph "Blockchain Layer"
        POLYGON[Polygon Network<br/>Contract Registry]
        WALLET[Custodial Wallets<br/>Platform Managed]
    end
    
    subgraph "External Services"
        EMAIL[SendGrid<br/>Email Service]
        SMS[Twilio<br/>SMS Service]
        STORAGE[AWS S3<br/>File Storage]
        AI[OpenAI API<br/>Fraud Detection]
    end
    
    WEB --> GATEWAY
    MOBILE --> GATEWAY
    GATEWAY --> AUTH
    GATEWAY --> CONTRACT
    GATEWAY --> REP
    GATEWAY --> NOTIF
    GATEWAY --> DISPUTE
    GATEWAY --> VERIFY
    
    CONTRACT --> MONGO
    CONTRACT --> POLYGON
    REP --> MONGO
    REP --> REDIS
    AUTH --> MONGO
    AUTH --> REDIS
    
    NOTIF --> EMAIL
    NOTIF --> SMS
    VERIFY --> AI
    CONTRACT --> IPFS
    CONTRACT --> STORAGE
    
    POLYGON --> WALLET
```

## 2. User Journey Flow

```mermaid
graph TD
    START([User Lands on Platform]) --> ROLE{User Type?}
    
    ROLE -->|Freelancer| FL_REG[Freelancer Registration]
    ROLE -->|Client| CL_INVITE[Receives Contract Invitation]
    
    FL_REG --> FL_PROFILE[Create Profile<br/>Skills, Portfolio, Bio]
    FL_PROFILE --> FL_VERIFY[Verification Process<br/>Email, Phone, LinkedIn]
    FL_VERIFY --> FL_CONTRACT[Create Digital Contract<br/>Scope, Timeline, Payment]
    FL_CONTRACT --> SEND_INVITE[Send Contract to Client<br/>Email Invitation]
    
    CL_INVITE --> CL_REVIEW[Review Contract<br/>Scope, Freelancer Profile]
    CL_REVIEW --> CL_DECISION{Accept Contract?}
    CL_DECISION -->|No| NEGOTIATE[Request Changes<br/>Back to Freelancer]
    CL_DECISION -->|Yes| CL_SIGN[Digital Signature<br/>Quick Registration]
    
    NEGOTIATE --> FL_CONTRACT
    CL_SIGN --> ACTIVE_CONTRACT[Active Contract<br/>Project Management]
    SEND_INVITE --> CL_INVITE
    
    ACTIVE_CONTRACT --> MILESTONES[Milestone Tracking<br/>Submit/Approve Work]
    MILESTONES --> PAYMENT[Payment Confirmation<br/>Direct Transfer]
    PAYMENT --> RATING[Client Rates Work<br/>1-10 Scale]
    RATING --> CONTRACT_CLOSE[Contract Closure<br/>Blockchain Record]
    CONTRACT_CLOSE --> REP_UPDATE[Reputation Score Update<br/>Portable Profile]
    
    REP_UPDATE --> FL_PORTFOLIO[Enhanced Portfolio<br/>Verified Track Record]
    FL_PORTFOLIO --> NEXT_CLIENT[Share Profile with<br/>Future Clients]
```

## 3. Contract Lifecycle State Machine

```mermaid
stateDiagram-v2
    [*] --> Draft: Freelancer creates contract
    
    Draft --> Sent: Send invitation to client
    Draft --> Cancelled: Freelancer cancels
    
    Sent --> UnderReview: Client opens invitation
    Sent --> Expired: 7 days no response
    
    UnderReview --> NeedsChanges: Client requests modifications
    UnderReview --> Signed: Client accepts and signs
    UnderReview --> Rejected: Client rejects contract
    
    NeedsChanges --> Draft: Freelancer makes changes
    
    Signed --> Active: Both parties confirmed
    Active --> InProgress: Work begins
    
    InProgress --> MilestoneSubmitted: Freelancer submits work
    MilestoneSubmitted --> MilestoneApproved: Client approves
    MilestoneSubmitted --> MilestoneRejected: Client requests changes
    MilestoneRejected --> InProgress: Freelancer revises
    
    MilestoneApproved --> PaymentPending: Awaiting payment
    PaymentPending --> PaymentConfirmed: Payment received
    PaymentConfirmed --> InProgress: More milestones
    PaymentConfirmed --> ReadyToClose: All milestones complete
    
    ReadyToClose --> Rated: Client provides rating
    Rated --> Completed: Contract closed successfully
    
    Active --> Disputed: Either party raises dispute
    InProgress --> Disputed: Either party raises dispute
    PaymentPending --> Disputed: Payment issues
    
    Disputed --> Mediation: Platform mediates
    Mediation --> Resolved: Dispute resolved
    Mediation --> Escalated: Legal action needed
    
    Resolved --> Completed: Resume normal flow
    Escalated --> Terminated: Contract terminated
    
    Expired --> [*]
    Cancelled --> [*]
    Rejected --> [*]
    Completed --> [*]
    Terminated --> [*]
```

## 4. Reputation Scoring System

```mermaid
graph TB
    subgraph "Input Factors"
        DELIVERY[On-time Delivery<br/>Weight: 30%]
        RATING[Client Ratings<br/>Weight: 40%]
        COMPLETION[Completion Rate<br/>Weight: 10%]
        VERIFICATION[Verification Level<br/>Weight: 10%]
        EXPERIENCE[Experience Points<br/>Weight: 10%]
    end
    
    subgraph "Scoring Engine"
        CALC[Weighted Calculation<br/>Algorithm]
        BONUS[Bonus Multipliers<br/>Verified Clients: 1.5x]
        PENALTY[Penalty System<br/>Late: -5 RP, Ghost: -50 RP]
    end
    
    subgraph "Output Score"
        SCORE[Reputation Score<br/>0-100 Scale]
        TIER[Score Tier<br/>⭐⭐⭐⭐⭐ Elite to ⭐ New]
        BADGE[Verification Badges<br/>Trusted, Elite, Verified]
    end
    
    subgraph "Blockchain Storage"
        IMMUTABLE[Immutable Record<br/>Polygon Network]
        TIMESTAMP[Timestamp Proof<br/>Contract Events]
        PORTABLE[Portable Profile<br/>Cross-platform]
    end
    
    DELIVERY --> CALC
    RATING --> CALC
    COMPLETION --> CALC
    VERIFICATION --> CALC
    EXPERIENCE --> CALC
    
    CALC --> BONUS
    BONUS --> PENALTY
    PENALTY --> SCORE
    
    SCORE --> TIER
    SCORE --> BADGE
    SCORE --> IMMUTABLE
    
    IMMUTABLE --> TIMESTAMP
    TIMESTAMP --> PORTABLE
```

## 5. Database Entity Relationship Diagram

```mermaid
erDiagram
    USERS {
        uuid id PK
        string email UK
        string password_hash
        string full_name
        string user_type
        jsonb profile_data
        boolean email_verified
        boolean phone_verified
        boolean linkedin_verified
        timestamp created_at
        timestamp updated_at
    }
    
    FREELANCER_PROFILES {
        uuid id PK
        uuid user_id FK
        text bio
        jsonb skills
        jsonb portfolio_links
        string linkedin_url
        string github_url
        decimal reputation_score
        integer reputation_points
        string tier
        jsonb badges
        timestamp created_at
        timestamp updated_at
    }
    
    CLIENT_PROFILES {
        uuid id PK
        uuid user_id FK
        string company_name
        string company_domain
        boolean business_verified
        decimal client_score
        integer contracts_created
        timestamp created_at
        timestamp updated_at
    }
    
    CONTRACTS {
        uuid id PK
        uuid freelancer_id FK
        uuid client_id FK
        string title
        text description
        decimal amount
        string currency
        jsonb milestones
        string status
        string blockchain_hash
        timestamp created_at
        timestamp signed_at
        timestamp completed_at
        timestamp updated_at
    }
    
    MILESTONES {
        uuid id PK
        uuid contract_id FK
        string title
        text description
        decimal amount
        string status
        timestamp due_date
        timestamp submitted_at
        timestamp approved_at
        jsonb deliverables
        text client_feedback
    }
    
    RATINGS {
        uuid id PK
        uuid contract_id FK
        uuid rater_id FK
        uuid ratee_id FK
        integer overall_rating
        integer communication_rating
        integer quality_rating
        integer timeliness_rating
        text feedback
        boolean is_public
        timestamp created_at
    }
    
    DISPUTES {
        uuid id PK
        uuid contract_id FK
        uuid raised_by FK
        string dispute_type
        text description
        jsonb evidence
        string status
        text resolution
        uuid mediator_id FK
        timestamp created_at
        timestamp resolved_at
    }
    
    VERIFICATIONS {
        uuid id PK
        uuid user_id FK
        string verification_type
        string status
        jsonb verification_data
        timestamp verified_at
        timestamp expires_at
    }
    
    BLOCKCHAIN_RECORDS {
        uuid id PK
        uuid contract_id FK
        string transaction_hash
        string event_type
        jsonb event_data
        integer block_number
        timestamp block_timestamp
        timestamp created_at
    }
    
    NOTIFICATIONS {
        uuid id PK
        uuid user_id FK
        string type
        string title
        text message
        jsonb data
        boolean read
        string channel
        timestamp sent_at
        timestamp created_at
    }
    
    USERS ||--o{ FREELANCER_PROFILES : "has"
    USERS ||--o{ CLIENT_PROFILES : "has"
    USERS ||--o{ CONTRACTS : "creates/receives"
    USERS ||--o{ RATINGS : "gives/receives"
    USERS ||--o{ DISPUTES : "raises"
    USERS ||--o{ VERIFICATIONS : "has"
    USERS ||--o{ NOTIFICATIONS : "receives"
    
    CONTRACTS ||--o{ MILESTONES : "contains"
    CONTRACTS ||--o{ RATINGS : "generates"
    CONTRACTS ||--o{ DISPUTES : "may_have"
    CONTRACTS ||--o{ BLOCKCHAIN_RECORDS : "recorded_on"
    
    FREELANCER_PROFILES ||--o{ CONTRACTS : "works_on"
    CLIENT_PROFILES ||--o{ CONTRACTS : "creates"
```

## 6. Payment & Verification Flow

```mermaid
sequenceDiagram
    participant F as Freelancer
    participant P as Platform
    participant C as Client
    participant B as Blockchain
    participant V as Verification Service
    
    Note over F,V: Contract Creation & Signing
    F->>P: Create digital contract
    P->>V: Verify freelancer credentials
    V-->>P: Verification status
    P->>C: Send contract invitation (email)
    C->>P: Review contract & freelancer profile
    C->>P: Digital signature (e-sign)
    P->>B: Record contract on blockchain
    B-->>P: Transaction hash & confirmation
    
    Note over F,V: Project Execution
    F->>P: Submit milestone deliverable
    P->>C: Notify client of submission
    C->>P: Approve/reject milestone
    P->>F: Payment request notification
    
    Note over F,V: Payment Process (Direct)
    F->>F: Receive payment (bank/UPI/PayPal)
    F->>P: Mark "Payment Received" + proof
    P->>V: Verify payment proof (AI scan)
    V-->>P: Payment verification result
    P->>C: Request payment confirmation
    C->>P: Confirm payment (1-click)
    P->>B: Record payment on blockchain
    B-->>P: Payment record hash
    
    Note over F,V: Contract Closure
    C->>P: Rate freelancer work (1-10)
    P->>P: Calculate reputation score update
    P->>B: Record rating & score on blockchain
    B-->>P: Final contract record
    P->>F: Updated reputation score
    P->>C: Contract completion confirmation
```

## 7. Dispute Resolution Workflow

```mermaid
graph TD
    ISSUE[Project Issue Arises] --> RAISE[Either Party Raises Dispute]
    RAISE --> EVIDENCE[Submit Evidence<br/>Screenshots, Files, Chat Logs]
    EVIDENCE --> AUTO_CHECK[Automated Checks<br/>Contract Terms, Timeline]
    
    AUTO_CHECK --> SIMPLE{Simple Resolution?}
    SIMPLE -->|Yes| AUTO_RESOLVE[Automatic Resolution<br/>Clear Contract Violation]
    SIMPLE -->|No| MANUAL[Manual Review Required]
    
    AUTO_RESOLVE --> OUTCOME[Apply Resolution<br/>RP Adjustment, Refund]
    
    MANUAL --> MEDIATOR[Platform Mediator Review]
    MEDIATOR --> INVESTIGATION[Investigate Evidence<br/>Contact Both Parties]
    INVESTIGATION --> DECISION[Mediation Decision]
    
    DECISION --> ACCEPTED{Both Parties Accept?}
    ACCEPTED -->|Yes| OUTCOME
    ACCEPTED -->|No| ESCALATE[Legal Escalation Option]
    
    ESCALATE --> EVIDENCE_PACKAGE[Generate Evidence Package<br/>Blockchain Proofs, Chat Logs]
    EVIDENCE_PACKAGE --> LEGAL_NOTICE[Legal Notice Template<br/>Small Claims Court Ready]
    
    OUTCOME --> BLOCKCHAIN_RECORD[Record Resolution on Blockchain]
    BLOCKCHAIN_RECORD --> REP_UPDATE[Update Reputation Scores]
    REP_UPDATE --> NOTIFY[Notify Both Parties]
    
    LEGAL_NOTICE --> COURT[Small Claims Court<br/>Outside Platform]
```

## 8. Technical Infrastructure & Deployment

```mermaid
graph TB
    subgraph "CDN & Load Balancing"
        CDN[CloudFlare CDN<br/>Global Distribution]
        LB[Load Balancer<br/>Traffic Distribution]
    end
    
    subgraph "Application Tier"
        WEB1[Web Server 1<br/>Node.js + Express]
        WEB2[Web Server 2<br/>Node.js + Express]
        WEB3[Web Server 3<br/>Node.js + Express]
    end
    
    subgraph "Microservices"
        AUTH_SVC[Auth Service<br/>JWT + OAuth]
        CONTRACT_SVC[Contract Service<br/>Business Logic]
        REP_SVC[Reputation Service<br/>Scoring Engine]
        NOTIF_SVC[Notification Service<br/>Email + SMS]
        BLOCKCHAIN_SVC[Blockchain Service<br/>Polygon Integration]
    end
    
    subgraph "Data Storage"
        MONGO_PRIMARY[(MongoDB Primary<br/>User & Contract Data)]
        MONGO_SECONDARY[(MongoDB Secondary<br/>Read Replicas)]
        REDIS_CLUSTER[(Redis Cluster<br/>Cache & Sessions)]
    end
    
    subgraph "File Storage"
        S3[AWS S3<br/>Document Storage]
        IPFS_NODE[IPFS Node<br/>Decentralized Storage]
    end
    
    subgraph "Blockchain Infrastructure"
        POLYGON_NODE[Polygon RPC Node<br/>Blockchain Connection]
        WALLET_SERVICE[Custodial Wallet Service<br/>Key Management]
    end
    
    subgraph "Monitoring & Analytics"
        LOGS[Centralized Logging<br/>ELK Stack]
        METRICS[Metrics Collection<br/>Prometheus + Grafana]
        ALERTS[Alert Manager<br/>PagerDuty Integration]
    end
    
    CDN --> LB
    LB --> WEB1
    LB --> WEB2
    LB --> WEB3
    
    WEB1 --> AUTH_SVC
    WEB2 --> CONTRACT_SVC
    WEB3 --> REP_SVC
    
    AUTH_SVC --> MONGO_PRIMARY
    CONTRACT_SVC --> MONGO_PRIMARY
    REP_SVC --> REDIS_CLUSTER
    NOTIF_SVC --> MONGO_SECONDARY
    
    CONTRACT_SVC --> S3
    CONTRACT_SVC --> IPFS_NODE
    BLOCKCHAIN_SVC --> POLYGON_NODE
    BLOCKCHAIN_SVC --> WALLET_SERVICE
    
    WEB1 --> LOGS
    WEB2 --> METRICS
    WEB3 --> ALERTS
```
