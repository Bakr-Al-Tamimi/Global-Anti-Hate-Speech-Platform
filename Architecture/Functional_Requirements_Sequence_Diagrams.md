# Functional Requirements Sequence Diagrams

This document provides PlantUML sequence diagrams illustrating the key interactions and flows for each functional requirement of the Global Anti-Hate Speech Platform (GAHSP).

---

## 1. Data Ingestion & Scraping Module

### 1.1. Social Media Integration (API-Driven)

```plantuml
@startuml
title Social Media Integration (API-Driven)

actor "Social Media Platform" as SMP
participant "Multi-threaded Monitoring Process" as MTMP
participant "Data Ingestion Module" as DIM
database "Database" as DB

MTMP -> SMP: Continuously search for #hate-speech hashtags in replies
activate MTMP
SMP --> MTMP: Detected reply with #hate-speech hashtag
MTMP -> SMP: Request original "reported post" details (API)
activate SMP
SMP --> MTMP: Provide post content, metadata, media
deactivate SMP
MTMP -> DIM: Submit "reported post" for ingestion
activate DIM
DIM -> DIM: Process content (multi-lingual, media)
DIM -> DB: Store reported post details
activate DB
DB --> DIM: Confirmation
deactivate DB
DIM --> MTMP: Ingestion complete
deactivate DIM
@enduml
```

### 1.2. Web Page Scraping

```plantuml
@startuml
title Web Page Scraping

actor Administrator as Admin
participant "Admin UI" as UI
participant "URL Management Service" as UMS
participant "Web Scraper" as WS
participant "Data Ingestion Module" as DIM
database "Database" as DB

Admin -> UI: Define/Manage Web Page URL for scraping
activate UI
UI -> UMS: Submit URL and schedule
activate UMS
UMS -> DB: Store URL configuration
activate DB
DB --> UMS: Confirmation
deactivate DB
UMS --> UI: Configuration saved
deactivate UMS
deactivate UI

loop Scheduled Scraping
    UMS -> WS: Trigger scraping for configured URL
    activate WS
    WS -> WS: Access web page, parse HTML
    WS --> WS: Extract content, comments, author, media (multi-lingual)
    WS -> DIM: Submit extracted content
    activate DIM
    DIM -> DIM: Process content
    DIM -> DB: Store web page content
    activate DB
    DB --> DIM: Confirmation
    deactivate DB
    DIM --> WS: Ingestion complete
    deactivate DIM
    WS --> UMS: Scraping cycle complete
    deactivate WS
end
@enduml
```

---

## 2. Hate Speech Identification & Trigger Mechanism

### 2.6. Multi-threaded Monitoring Process (Triggering Analysis)

```plantuml
@startuml
title Multi-threaded Monitoring Process (Triggering Analysis)

actor "Social Media Platform" as SMP
participant "Multi-threaded Monitoring Process" as MTMP
participant "Data Ingestion Module" as DIM
participant "Hate Speech Analysis Module" as HSAM
database "Database" as DB

loop Continuous Monitoring
    MTMP -> SMP: Search for #hate-speech hashtags in reply posts
    activate MTMP
    SMP --> MTMP: Detected reply with #hate-speech hashtag
    MTMP -> SMP: Retrieve original "reported post"
    activate SMP
    SMP --> MTMP: Original post content and metadata
    deactivate SMP
    MTMP -> DIM: Submit "reported post" for ingestion
    activate DIM
    DIM -> DB: Store initial reported post data
    activate DB
    DB --> DIM: Data stored
    deactivate DB
    DIM -> HSAM: Trigger Initial Assessment for "potential hate-speech"
    activate HSAM
    HSAM --> DIM: Acknowledgment
    deactivate HSAM
    DIM --> MTMP: Ingestion and trigger complete
    deactivate DIM
    deactivate MTMP
end
@enduml
```

---

## 3. Anonymous User Submission & Multi-Modal Content Ingestion Module (CORE FEATURE)

```plantuml
@startuml
title Anonymous User Submission & Multi-Modal Content Ingestion

actor "Anonymous User" as User
participant "Secure Submission Portal (UI)" as Portal
participant "Content Sanitization & Virus Scanner" as CSVS
participant "Temporary Encrypted Storage" as TempStorage
participant "Data Ingestion Module" as DIM
participant "Hate Speech Analysis Module" as HSAM
database "Database" as DB

User -> Portal: Access multi-lingual secure portal
activate Portal
User -> Portal: Submit multi-modal content (URL, audio, video, image) + optional metadata
Portal -> CSVS: Upload content for scanning and sanitization
activate CSVS
CSVS -> CSVS: Perform virus scan, remove hidden metadata (e.g., EXIF)
CSVS --> Portal: Sanitized content or error
deactivate CSVS
alt If content is valid
    Portal -> TempStorage: Store sanitized content securely
    activate TempStorage
    TempStorage --> Portal: Storage confirmation
    deactivate TempStorage
    Portal -> DIM: Submit content and metadata for ingestion
    activate DIM
    DIM -> DB: Store initial submission details (with anonymity flag)
    activate DB
    DB --> DIM: Data stored
    deactivate DB
    DIM -> HSAM: Trigger Initial Assessment for "potential hate-speech"
    activate HSAM
    HSAM --> DIM: Acknowledgment
    deactivate HSAM
    DIM --> Portal: Ingestion and trigger complete
    deactivate DIM
    Portal --> User: Submission successful (anonymous confirmation)
else If content is invalid
    Portal --> User: Submission failed (e.g., virus detected, invalid format)
end
deactivate Portal
@enduml
```

---

## 4. Hate Speech Analysis Module (Enhanced for Multi-Modal & Multi-Lingual)

### 4.1. Initial Assessment & Flagging

```plantuml
@startuml
title Initial Assessment & Flagging

participant "Data Ingestion Module" as DIM
participant "Hate Speech Analysis Module" as HSAM
participant "NLP/ML Models" as MLModels
database "Database" as DB

DIM -> HSAM: Submit ingested content for initial assessment
activate HSAM
HSAM -> MLModels: Analyze content (text, image, audio, video)
activate MLModels
MLModels --> HSAM: Return "potential hate-speech" flag and confidence score
deactivate MLModels
HSAM -> DB: Record "potential hate-speech" instance, confidence score
activate DB
DB --> HSAM: Confirmation
deactivate DB
HSAM --> DIM: Initial assessment complete
deactivate HSAM
@enduml
```

### 4.2. International Anti-Hate Speech Trustees Review & Voting

```plantuml
@startuml
title International Anti-Hate Speech Trustees Review & Voting

participant "Hate Speech Analysis Module" as HSAM
database "Database" as DB
participant "Trustee UI" as TUI
actor "International Anti-Hate Speech Trustee" as Trustee

HSAM -> DB: Query "potential hate-speech" instances for review
activate DB
DB --> HSAM: List of instances
deactivate DB
HSAM -> TUI: Present "potential hate-speech" instances to Trustees
activate TUI
Trustee -> TUI: Review instance, cast "up" or "down" vote
activate Trustee
TUI -> DB: Record Trustee vote
activate DB
DB --> TUI: Vote recorded
deactivate DB
TUI -> HSAM: Notify of new vote
deactivate Trustee
HSAM -> HSAM: Check dynamic threshold for instance
alt If threshold met
    HSAM -> DB: Update instance status to "confirmed hate speech"
    activate DB
    DB --> HSAM: Status updated
    deactivate DB
    HSAM -> HSAM: Trigger "Thorough Analysis"
else If threshold not met
    HSAM -> TUI: Update instance vote count
end
deactivate TUI
deactivate HSAM
@enduml
```

### 4.3. Thorough Analysis for Confirmed Hate Speech

```plantuml
@startuml
title Thorough Analysis for Confirmed Hate Speech

participant "Hate Speech Analysis Module" as HSAM
database "Database" as DB
participant "Entity Extraction Models" as EEM
participant "Jurisdiction Mapping Service" as JMS
participant "Named Entity Recognition Models" as NERM

HSAM -> HSAM: Receive trigger for "confirmed hate speech" instance
activate HSAM
HSAM -> DB: Retrieve confirmed hate speech content and initial analysis
activate DB
DB --> HSAM: Content and initial data
deactivate DB

HSAM -> EEM: Request Multi-Lingual Entity Extraction (targets, subjects, accusations)
activate EEM
EEM --> HSAM: Extracted entities
deactivate EEM

HSAM -> HSAM: Extract Associated Hashtags/Keywords, Temporal Data, Geographic Location, Hate-Manufacturer Identity

HSAM -> JMS: Request Jurisdiction Mapping (manufacturer/source, involved jurisdictions)
activate JMS
JMS --> HSAM: Jurisdictional data
deactivate JMS

HSAM -> NERM: Request Named Entity Recognition (people, characters, organizations, events)
activate NERM
NERM --> HSAM: Extracted named entities
deactivate NERM

HSAM -> HSAM: Identify Victim Identification

HSAM -> DB: Store comprehensive "thorough analysis" results
activate DB
DB --> HSAM: Confirmation
deactivate DB
HSAM --> HSAM: Thorough analysis complete
deactivate HSAM
@enduml
```

---

## 5. Database Management Module (High-Level Interactions)

```plantuml
@startuml
title Database Management Module (High-Level Interactions)

participant "Data Ingestion Module" as DIM
participant "Hate Speech Analysis Module" as HSAM
participant "Aggregation & Reporting Module" as ARM
participant "Public Exposure & Scoring Module" as PESM
database "SQL Database" as DB

DIM -> DB: Store raw ingested data (social media, web, user submissions)
activate DB
DB --> DIM: Data stored
deactivate DB

HSAM -> DB: Store "potential hate-speech" flags, confidence scores
activate DB
DB --> HSAM: Data stored
deactivate DB

HSAM -> DB: Retrieve "potential hate-speech" for Trustee review
activate DB
DB --> HSAM: Instances for review
deactivate DB

HSAM -> DB: Store Trustee votes, update "confirmed hate speech" status
activate DB
DB --> HSAM: Status updated
deactivate DB

HSAM -> DB: Store comprehensive "thorough analysis" results
activate DB
DB --> HSAM: Results stored
deactivate DB

ARM -> DB: Retrieve data for search, filtering, aggregation
activate DB
DB --> ARM: Query results
deactivate DB

ARM -> DB: Persist aggregated findings as new data records
activate DB
DB --> ARM: Record saved
deactivate DB

PESM -> DB: Retrieve published web pages and scores
activate DB
DB --> PESM: Page data
deactivate DB

PESM -> DB: Update public hate-speech scale scores
activate DB
DB --> PESM: Score updated
deactivate DB

DB -> DB: Perform Index Optimization, Transaction Management, Multi-Lingual Search Indexing (internal processes)
@enduml
```

---

## 6. Aggregation & Reporting Module

### 6.3. Search & Filtering

```plantuml
@startuml
title Search & Filtering

actor User as Analyst
participant "Web UI" as UI
participant "Aggregation & Reporting Module" as ARM
database "Database" as DB

Analyst -> UI: Input search/filter criteria (language, modality, victim, date range, etc.)
activate UI
UI -> ARM: Submit search/filter request
activate ARM
ARM -> DB: Execute complex multi-lingual query
activate DB
DB --> ARM: Return filtered records
deactivate DB
ARM --> UI: Display search results
deactivate ARM
UI --> Analyst: Display results
deactivate UI
@enduml
```

### 6.4. Aggregation Workbench (Interactive Aggregation)

```plantuml
@startuml
title Aggregation Workbench (Interactive Aggregation)

actor User as Analyst
participant "Web UI" as UI
participant "Aggregation & Reporting Module" as ARM
database "Database" as DB

Analyst -> UI: Select aggregation criteria (same event, victim, manufacturer, topic, tags, jurisdiction)
activate UI
UI -> ARM: Submit aggregation request
activate ARM
ARM -> DB: Retrieve relevant data for aggregation
activate DB
DB --> ARM: Raw data
deactivate DB
ARM -> ARM: Perform interactive grouping, cross-lingual topic modeling, multi-modal summarization
ARM --> UI: Display interactive graphical representations and statistical summaries
deactivate ARM
UI --> Analyst: Display visualizations
deactivate UI
@enduml
```

### 6.5. Aggregation Output Options (Generate Report)

```plantuml
@startuml
title Aggregation Output Options (Generate Report)

actor User as Analyst
participant "Web UI" as UI
participant "Aggregation & Reporting Module" as ARM
participant "Report Generator Service" as RGS

Analyst -> UI: Select aggregated data, choose "Generate Report" option
activate UI
UI -> ARM: Request report generation (specify format: PDF, CSV, DOCX, template, language)
activate ARM
ARM -> ARM: Retrieve aggregated data (if not already in memory)
ARM -> RGS: Send data and report specifications
activate RGS
RGS -> RGS: Create formal, customizable report
RGS --> ARM: Generated report file
deactivate RGS
ARM --> UI: Provide report for download
deactivate ARM
UI --> Analyst: Download report
deactivate UI
@enduml
```

### 6.5. Aggregation Output Options (Display as Web Page)

```plantuml
@startuml
title Aggregation Output Options (Display as Web Page)

actor User as Analyst
participant "Web UI" as UI
participant "Aggregation & Reporting Module" as ARM
participant "Public Web Page Display Module" as PWPDM
database "Database" as DB

Analyst -> UI: Select aggregated data, choose "Display as Web Page" option
activate UI
UI -> ARM: Request to publish aggregated findings as web page
activate ARM
ARM -> ARM: Prepare content for public display
ARM -> DB: Store web page content and metadata
activate DB
DB --> ARM: Confirmation
deactivate DB
ARM -> PWPDM: Notify of new public web page
activate PWPDM
PWPDM --> ARM: Acknowledgment
deactivate PWPDM
ARM --> UI: Web page published successfully, provide URL
deactivate ARM
UI --> Analyst: Display URL for public web page
deactivate UI
@enduml
```

---

## 7. Public Exposure & Scoring Module (New)

### 7.2. Public Hate-Speech Scale Scoring

```plantuml
@startuml
title Public Hate-Speech Scale Scoring

actor "Authenticated User" as PublicUser
participant "Public Web Page Display Module" as PWPDM
participant "Web UI" as UI
database "Database" as DB

PublicUser -> UI: View publicly displayed web page
activate UI
UI -> PublicUser: Display web page content and current score
PublicUser -> UI: Submit score on "hate-speech scale" (up/down vote)
activate PublicUser
UI -> PWPDM: Send scoring request with user authentication
activate PWPDM
PWPDM -> DB: Update web page score
activate DB
DB --> PWPDM: Score updated
deactivate DB
PWPDM --> UI: Confirmation of score update
deactivate PWPDM
UI --> PublicUser: Display updated score
deactivate PublicUser
deactivate UI
@enduml
```

### 7.3. Dynamic Listing & Reporting (Top 10 for UN Consideration)

```plantuml
@startuml
title Dynamic Listing & Reporting (Top 10 for UN Consideration)

participant "Public Web Page Display Module" as PWPDM
database "Database" as DB
participant "Web UI" as UI
actor "UN Security Council (Conceptual)" as UNSC

loop Continuous Update
    PWPDM -> DB: Query for "Top 10" highest-scoring hate-speech web pages
    activate DB
    DB --> PWPDM: List of top pages with scores
    deactivate DB
    PWPDM -> UI: Update dynamic listing of web pages
    activate UI
    UI --> UI: Display pages in descending order of score
    UI -> UI: Highlight "Top 10" for UN consideration
    deactivate UI
    PWPDM -> UNSC: (Conceptual) Present "Top 10" highest-scoring pages
    activate UNSC
    UNSC --> PWPDM: (Conceptual) Acknowledgment
    deactivate UNSC
end
@enduml
```

---

## 8. Data Replication & Recovery Module (Globally Distributed & Secure)

### 8.1. Real-time Replication (Geographic Redundancy)

```plantuml
@startuml
title Real-time Replication (Geographic Redundancy)

database "Primary Database" as PrimaryDB
participant "Replication Process" as RP
database "Replica Database" as ReplicaDB
participant "Immutable Log" as ILog

loop Continuous Replication
    PrimaryDB -> RP: Notify of database transaction (insert, update, delete)
    activate RP
    RP -> ILog: Log transaction in immutable, cryptographically secured log
    activate ILog
    ILog --> RP: Log entry confirmed
    deactivate ILog
    RP -> ReplicaDB: Apply transaction to replica database
    activate ReplicaDB
    ReplicaDB --> RP: Transaction applied, ensure consistency
    deactivate ReplicaDB
    RP --> PrimaryDB: Acknowledgment of replication
    deactivate RP
end
@enduml
```

### 8.3. Point-in-Time Recovery & 8.4. Special Case Backup for Rollback & Legal Archiving

```plantuml
@startuml
title Point-in-Time Recovery & Special Case Backup

actor Administrator as Admin
participant "Recovery Tool" as RT
database "Primary Database" as PrimaryDB
database "Replica Database" as ReplicaDB
participant "Immutable Log" as ILog
participant "Backup Service" as BS
participant "Legal Archive Storage" as LAS

Admin -> RT: Initiate Point-in-Time Recovery to specific transaction point
activate RT
RT -> ILog: Retrieve transaction log up to specified point
activate ILog
ILog --> RT: Transaction log data
deactivate ILog
RT -> PrimaryDB: Roll back Primary Database to specified point
activate PrimaryDB
PrimaryDB --> RT: Rollback complete
deactivate PrimaryDB

RT -> BS: Request Pre-Synchronization Backup of PrimaryDB (distinct state)
activate BS
BS -> PrimaryDB: Read current state
BS -> BS: Create cryptographically signed backup
BS -> LAS: Store PrimaryDB backup for legal archiving
activate LAS
LAS --> BS: Backup stored
deactivate LAS
BS --> RT: PrimaryDB backup complete
deactivate BS

RT -> ReplicaDB: Roll back Replica Database to specified point
activate ReplicaDB
ReplicaDB --> RT: Rollback complete
deactivate ReplicaDB

RT -> BS: Request Pre-Synchronization Backup of ReplicaDB (distinct state)
activate BS
BS -> ReplicaDB: Read current state
BS -> BS: Create cryptographically signed backup
BS -> LAS: Store ReplicaDB backup for legal archiving
activate LAS
LAS --> BS: Backup stored
deactivate LAS
BS --> RT: ReplicaDB backup complete
deactivate BS

RT -> RP: Re-enable Replication Process (if it was paused)
activate RP
RP --> RT: Replication re-enabled
deactivate RP
RT --> Admin: Recovery and archiving complete
deactivate RT
@enduml
