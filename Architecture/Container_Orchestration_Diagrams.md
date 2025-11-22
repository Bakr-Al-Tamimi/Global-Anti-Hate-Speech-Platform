# Container Orchestration Architecture - Visual Diagrams

This document provides visual representations of the Kubernetes orchestration architecture for the Anti-Hate Platform.

---

## Diagram 1: Multi-Cluster Hierarchy

![Multi-Cluster Hierarchy](diagrams/multi-cluster-hierarchy.svg)

<details>
<summary>Mermaid Source Code</summary>

```mermaid
graph TD
    %% International Level - Stadium shape for prominence
    INTERNATIONAL(["🌍 INTERNATIONAL COORDINATION CLUSTER<br/>────────────────────<br/>International Control Plane<br/>Smart Feed Engine<br/>Model Distribution<br/>UN Reporting"])
    
    %% Ellipsis - More countries on sides (circles)
    ELLIPSIS_COUNTRIES_L(("⋯"))
    ELLIPSIS_COUNTRIES_R(("⋯"))
    
    %% National Level - Rounded rectangles
    NATIONAL_USA("🇺🇸 USA<br/>National<br/>Cluster")
    NATIONAL_RUS("🇷🇺 Russia<br/>National<br/>Cluster")
    NATIONAL_CHN("🇨🇳 China<br/>National<br/>Cluster")
    NATIONAL_IND("🇮🇳 India<br/>National<br/>Cluster")
    NATIONAL_BRA("🇧🇷 Brazil<br/>National<br/>Cluster")
    NATIONAL_ZAF("🇿🇦 South Africa<br/>National<br/>Cluster")
    NATIONAL_EGY("🇪🇬 Egypt<br/>National<br/>Cluster")
    
    %% Geography Level - Circles for geographies, rounded for ellipses
    ELLIPSIS_GEO_USA("⋯ more<br/>geographies")
    GEOGRAPHY_NY(("📍 New York<br/>Geography"))
    
    ELLIPSIS_GEO_RUS("⋯ more<br/>geographies")
    GEOGRAPHY_MSK(("📍 Moscow<br/>Geography"))
    
    ELLIPSIS_GEO_CHN("⋯ more<br/>geographies")
    GEOGRAPHY_BJ(("📍 Beijing<br/>Geography"))
    
    ELLIPSIS_GEO_IND("⋯ more<br/>geographies")
    GEOGRAPHY_MUM(("📍 Mumbai<br/>Geography"))
    
    %% Local Level - Rounded rectangles for locals, rounded for ellipses
    ELLIPSIS_LOC_NY("⋯ more<br/>local")
    LOCAL_NYC("💾 NYC<br/>Local")
    
    ELLIPSIS_LOC_MSK("⋯ more<br/>local")
    LOCAL_MSK("💾 Moscow<br/>Local")
    
    ELLIPSIS_LOC_BJ("⋯ more<br/>local")
    LOCAL_SHA("💾 Shanghai<br/>Local")
    
    ELLIPSIS_LOC_MUM("⋯ more<br/>local")
    LOCAL_DEL("💾 Delhi<br/>Local")
    
    %% International to National (with ellipses for more countries)
    INTERNATIONAL ==>|Config & Models| ELLIPSIS_COUNTRIES_L
    INTERNATIONAL ==>|Config & Models| NATIONAL_USA
    INTERNATIONAL ==>|Config & Models| NATIONAL_RUS
    INTERNATIONAL ==>|Config & Models| NATIONAL_CHN
    INTERNATIONAL ==>|Config & Models| NATIONAL_IND
    INTERNATIONAL ==>|Config & Models| NATIONAL_BRA
    INTERNATIONAL ==>|Config & Models| NATIONAL_ZAF
    INTERNATIONAL ==>|Config & Models| NATIONAL_EGY
    INTERNATIONAL ==>|Config & Models| ELLIPSIS_COUNTRIES_R
    
    %% National to Geography (showing multiple per country)
    NATIONAL_USA -->|Coordination| ELLIPSIS_GEO_USA
    NATIONAL_USA -->|Coordination| GEOGRAPHY_NY
    
    NATIONAL_RUS -->|Coordination| ELLIPSIS_GEO_RUS
    NATIONAL_RUS -->|Coordination| GEOGRAPHY_MSK
    
    NATIONAL_CHN -->|Coordination| ELLIPSIS_GEO_CHN
    NATIONAL_CHN -->|Coordination| GEOGRAPHY_BJ
    
    NATIONAL_IND -->|Coordination| ELLIPSIS_GEO_IND
    NATIONAL_IND -->|Coordination| GEOGRAPHY_MUM
    
    %% Geography to Local (showing multiple per geography)
    GEOGRAPHY_NY -->|Tasks| ELLIPSIS_LOC_NY
    GEOGRAPHY_NY -->|Tasks| LOCAL_NYC
    
    GEOGRAPHY_MSK -->|Tasks| ELLIPSIS_LOC_MSK
    GEOGRAPHY_MSK -->|Tasks| LOCAL_MSK
    
    GEOGRAPHY_BJ -->|Tasks| ELLIPSIS_LOC_BJ
    GEOGRAPHY_BJ -->|Tasks| LOCAL_SHA
    
    GEOGRAPHY_MUM -->|Tasks| ELLIPSIS_LOC_MUM
    GEOGRAPHY_MUM -->|Tasks| LOCAL_DEL
    
    %% Data Flow (Bottom-Up)
    LOCAL_NYC -.->|Metadata| GEOGRAPHY_NY
    ELLIPSIS_LOC_NY -.->|Metadata| GEOGRAPHY_NY
    
    LOCAL_MSK -.->|Metadata| GEOGRAPHY_MSK
    ELLIPSIS_LOC_MSK -.->|Metadata| GEOGRAPHY_MSK
    
    LOCAL_SHA -.->|Metadata| GEOGRAPHY_BJ
    ELLIPSIS_LOC_BJ -.->|Metadata| GEOGRAPHY_BJ
    
    LOCAL_DEL -.->|Metadata| GEOGRAPHY_MUM
    ELLIPSIS_LOC_MUM -.->|Metadata| GEOGRAPHY_MUM
    
    GEOGRAPHY_NY -.->|Smart Feeds| NATIONAL_USA
    ELLIPSIS_GEO_USA -.->|Smart Feeds| NATIONAL_USA
    
    GEOGRAPHY_MSK -.->|Smart Feeds| NATIONAL_RUS
    ELLIPSIS_GEO_RUS -.->|Smart Feeds| NATIONAL_RUS
    
    GEOGRAPHY_BJ -.->|Smart Feeds| NATIONAL_CHN
    ELLIPSIS_GEO_CHN -.->|Smart Feeds| NATIONAL_CHN
    
    GEOGRAPHY_MUM -.->|Smart Feeds| NATIONAL_IND
    ELLIPSIS_GEO_IND -.->|Smart Feeds| NATIONAL_IND
    
    NATIONAL_USA -.->|National Feeds| INTERNATIONAL
    NATIONAL_RUS -.->|National Feeds| INTERNATIONAL
    NATIONAL_CHN -.->|National Feeds| INTERNATIONAL
    NATIONAL_IND -.->|National Feeds| INTERNATIONAL
    NATIONAL_BRA -.->|National Feeds| INTERNATIONAL
    NATIONAL_ZAF -.->|National Feeds| INTERNATIONAL
    NATIONAL_EGY -.->|National Feeds| INTERNATIONAL
    ELLIPSIS_COUNTRIES_L -.->|Feeds| INTERNATIONAL
    ELLIPSIS_COUNTRIES_R -.->|Feeds| INTERNATIONAL
    
    %% Styling
    classDef internationalStyle fill:#1e3a8a,stroke:#1e40af,stroke-width:3px,color:#fff
    classDef nationalStyle fill:#7c3aed,stroke:#6d28d9,stroke-width:2px,color:#fff
    classDef geographyStyle fill:#059669,stroke:#047857,stroke-width:2px,color:#fff
    classDef localStyle fill:#dc2626,stroke:#b91c1c,stroke-width:2px,color:#fff
    classDef ellipsisStyle fill:#666,stroke:#444,stroke-width:1px,color:#fff,stroke-dasharray:5
    
    class INTERNATIONAL internationalStyle
    class NATIONAL_USA,NATIONAL_RUS,NATIONAL_CHN,NATIONAL_IND,NATIONAL_BRA,NATIONAL_ZAF,NATIONAL_EGY nationalStyle
    class GEOGRAPHY_NY,GEOGRAPHY_MSK,GEOGRAPHY_BJ,GEOGRAPHY_MUM geographyStyle
    class LOCAL_NYC,LOCAL_MSK,LOCAL_SHA,LOCAL_DEL localStyle
    class ELLIPSIS_COUNTRIES_L,ELLIPSIS_COUNTRIES_R,ELLIPSIS_GEO_USA,ELLIPSIS_GEO_RUS,ELLIPSIS_GEO_CHN,ELLIPSIS_GEO_IND,ELLIPSIS_LOC_NY,ELLIPSIS_LOC_MSK,ELLIPSIS_LOC_BJ,ELLIPSIS_LOC_MUM ellipsisStyle
```

</details>

---

## Diagram 2: Local Node - Pod Architecture

```mermaid
graph TB
    subgraph "Namespace: antihate-observers"
        CT1["Pod: observer-twitter-1"]
        CT2["Pod: observer-twitter-2"]
        CT3["Pod: observer-telegram-1"]
        CF1["Pod: observer-weibo-1"]
        CF2["Pod: observer-vk-1"]
    end
    
    subgraph "Namespace: antihate-analyzers"
        AT1["Pod: analyzer-text-1<br/>(4 CPU, 16GB, 1 GPU)"]
        AT2["Pod: analyzer-text-2<br/>(4 CPU, 16GB, 1 GPU)"]
        AI1["Pod: analyzer-image-1"]
        AV1["Pod: analyzer-video-1"]
    end
    
    subgraph "Namespace: antihate-storage"
        PG1["Pod: postgres-0 (Primary)<br/>(8 CPU, 64GB, 2TB)"]
        PG2["Pod: postgres-1 (Replica)<br/>(8 CPU, 64GB, 2TB)"]
        PG3["Pod: postgres-2 (Replica)<br/>(8 CPU, 64GB, 2TB)"]
        REDIS["Pod: redis-cluster"]
    end
    
    subgraph "Namespace: antihate-coordination"
        SYNC["Pod: metadata-sync"]
        CTRL["Pod: control-agent"]
        HEALTH["Pod: health-monitor"]
    end
    
    subgraph "Namespace: antihate-system"
        KAFKA["Pod: kafka-broker"]
    end
    
    CT1 & CT2 & CT3 & CF1 & CF2 -->|Write Posts| KAFKA
    KAFKA -->|Analysis Queue| AT1 & AT2 & AI1 & AV1
    AT1 & AT2 & AI1 & AV1 -->|Query Content| PG1
    AT1 & AT2 & AI1 & AV1 -->|Write Results| PG1
    PG1 -.->|Replication| PG2 & PG3
    PG1 -->|Metadata| SYNC
    SYNC -.->|Push to Geography| REMOTE["Geography Cluster"]
    CTRL -.->|Pull Config| REMOTE
    
    style PG1 fill:#ffcccc
    style PG2 fill:#ffdddd
    style PG3 fill:#ffdddd
    style AT1 fill:#ccffcc
    style AT2 fill:#ccffcc
```

---

## Diagram 3: Service Mesh (Istio) - Traffic Flow

```mermaid
graph LR
    subgraph "Pod: observer-twitter"
        APP1[App Container]
        PROXY1[Envoy Sidecar]
    end
    
    subgraph "Pod: analyzer-text"
        APP2[App Container]
        PROXY2[Envoy Sidecar]
    end
    
    subgraph "Pod: postgres-0"
        APP3[PostgreSQL]
        PROXY3[Envoy Sidecar]
    end
    
    subgraph "Istio Control Plane"
        PILOT[Pilot<br/>Config Distribution]
        CITADEL[Citadel<br/>Certificate Management]
        MIXER[Mixer<br/>Telemetry]
    end
    
    APP1 -->|HTTP| PROXY1
    PROXY1 -.->|mTLS| PROXY3
    PROXY3 --> APP3
    
    APP2 -->|HTTP| PROXY2
    PROXY2 -.->|mTLS| PROXY3
    
    PILOT -->|VirtualService<br/>DestinationRule| PROXY1 & PROXY2 & PROXY3
    CITADEL -->|TLS Certificates| PROXY1 & PROXY2 & PROXY3
    PROXY1 & PROXY2 & PROXY3 -.->|Metrics| MIXER
    
    style PROXY1 fill:#e1e1ff
    style PROXY2 fill:#e1e1ff
    style PROXY3 fill:#e1e1ff
    style PILOT fill:#ffe1e1
    style CITADEL fill:#ffe1e1
    style MIXER fill:#ffe1e1
```

---

## Diagram 4: Auto-scaling Flow

```mermaid
sequenceDiagram
    participant User as Twitter API
    participant Kafka as Kafka Queue
    participant HPA as Horizontal Pod Autoscaler
    participant KEDA as KEDA ScaledObject
    participant Analyzer as Analyzer Pods
    participant Metrics as Metrics Server
    participant Cluster as Cluster Autoscaler
    
    User->>Kafka: High volume of posts
    Note over Kafka: Queue depth increases to 5000 msgs
    
    KEDA->>Kafka: Poll queue depth every 10s
    Kafka-->>KEDA: Lag = 5000 messages
    
    KEDA->>HPA: Update external metric
    HPA->>Metrics: Check current CPU/Memory
    Metrics-->>HPA: CPU 85%, Memory 90%
    
    Note over HPA: Calculate: Need 10 more pods
    HPA->>Analyzer: Scale from 3 to 13 pods
    
    Note over Analyzer: Pods pending (insufficient nodes)
    Analyzer->>Cluster: Request node capacity
    Cluster->>Cluster: Add 2 new c5.4xlarge nodes
    
    Note over Cluster: Nodes ready in ~3 min
    Cluster-->>Analyzer: Pods scheduled
    
    Analyzer->>Kafka: Process messages at higher rate
    Note over Kafka: Queue depth returns to normal
    
    KEDA->>HPA: Lag now 200 messages
    HPA->>Analyzer: Scale down to 5 pods (gradual)
    
    Note over Cluster: Wait 10 min (scale-down delay)
    Cluster->>Cluster: Remove underutilized nodes
```

---

## Diagram 5: Data Flow - Observation to Analysis

```mermaid
flowchart TD
    START([Twitter API]) -->|#hate-speech hashtag| C1[Observer Pod]
    
    C1 -->|Raw JSON| KAFKA[Kafka Topic:<br/>incoming-posts]
    
    KAFKA -->|Consumer Group| A1[Analyzer Text Pod 1]
    KAFKA -->|Consumer Group| A2[Analyzer Text Pod 2]
    KAFKA -->|Consumer Group| A3[Analyzer Text Pod 3]
    
    A1 & A2 & A3 -->|Query: Get full content| PG[(PostgreSQL<br/>Primary)]
    PG -->|Content| A1 & A2 & A3
    
    A1 & A2 & A3 -->|ML Inference| GPU[GPU Processing]
    GPU -->|Hate Score,<br/>Entities,<br/>Categories| A1 & A2 & A3
    
    A1 & A2 & A3 -->|Write analysis results| PG
    
    PG -.->|Streaming<br/>Replication| PG2[(PostgreSQL<br/>Replica 1)]
    PG -.->|Streaming<br/>Replication| PG3[(PostgreSQL<br/>Replica 2)]
    
    PG -->|Extract metadata<br/>SHA-256, timestamps,<br/>scores, entities| SYNC[Metadata Sync Pod]
    
    SYNC -.->|Encrypted gRPC<br/>over mTLS| GEOGRAPHY["Geography Cluster<br/>(Beijing)<br/>TimescaleDB"]
    
    style C1 fill:#ccffff
    style A1 fill:#ccffcc
    style A2 fill:#ccffcc
    style A3 fill:#ccffcc
    style PG fill:#ffcccc
    style SYNC fill:#ffffcc
```

---

## Diagram 6: Storage Architecture

```mermaid
graph TB
    subgraph "StatefulSet: postgres"
        PRIMARY["postgres-0<br/>(Primary)<br/>ReadWrite Pod"]
        REPLICA1["postgres-1<br/>(Replica)<br/>ReadOnly Pod"]
        REPLICA2["postgres-2<br/>(Replica)<br/>ReadOnly Pod"]
    end
    
    subgraph "Persistent Volumes"
        PVC0["PVC: data-postgres-0<br/>2TB gp3-encrypted"]
        PVC1["PVC: data-postgres-1<br/>2TB gp3-encrypted"]
        PVC2["PVC: data-postgres-2<br/>2TB gp3-encrypted"]
    end
    
    subgraph "AWS EBS Volumes"
        EBS0["EBS Volume<br/>vol-xxx001<br/>AZ: us-west-1a"]
        EBS1["EBS Volume<br/>vol-xxx002<br/>AZ: us-west-1b"]
        EBS2["EBS Volume<br/>vol-xxx003<br/>AZ: us-west-1c"]
    end
    
    subgraph "Backup Sidecar Containers"
        BACKUP0["pg-backup<br/>Daily 2 AM"]
        BACKUP1["pg-backup<br/>Daily 2 AM"]
        BACKUP2["pg-backup<br/>Daily 2 AM"]
    end
    
    PRIMARY --> PVC0 --> EBS0
    REPLICA1 --> PVC1 --> EBS1
    REPLICA2 --> PVC2 --> EBS2
    
    PRIMARY -.->|Streaming<br/>Replication| REPLICA1
    PRIMARY -.->|Streaming<br/>Replication| REPLICA2
    
    PRIMARY --- BACKUP0
    REPLICA1 --- BACKUP1
    REPLICA2 --- BACKUP2
    
    BACKUP0 & BACKUP1 & BACKUP2 -.->|S3 Backup| S3["S3 Bucket<br/>s3://antihate-backups/"]
    
    style PRIMARY fill:#ff9999
    style REPLICA1 fill:#ffcccc
    style REPLICA2 fill:#ffcccc
    style S3 fill:#99ccff
```

---

## Diagram 7: GitOps Deployment Flow

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant Git as GitHub Repo
    participant ArgoCD as ArgoCD Controller
    participant Helm as Helm
    participant K8s as Kubernetes API
    participant Pods as Running Pods
    
    Dev->>Git: Push code change<br/>(new analyzer version)
    Note over Git: CI builds image<br/>antihate/analyzer-text:v2.3.0
    
    Dev->>Git: Update values.yaml<br/>(image tag: v2.3.0)
    
    ArgoCD->>Git: Poll for changes (every 3 min)
    Git-->>ArgoCD: Detected change in<br/>clusters/local/delhi-001/
    
    ArgoCD->>ArgoCD: Compare desired state<br/>vs current state
    Note over ArgoCD: Out of sync detected
    
    ArgoCD->>Helm: Render templates with<br/>new values
    Helm-->>ArgoCD: Generated manifests
    
    ArgoCD->>K8s: Apply manifests<br/>(rolling update strategy)
    
    K8s->>Pods: Create new pods with v2.3.0
    K8s->>Pods: Wait for new pods to be Ready
    
    Note over Pods: New pods pass<br/>readiness probe
    
    K8s->>Pods: Terminate old pods (v2.2.0)
    
    Pods-->>K8s: Graceful shutdown complete
    K8s-->>ArgoCD: Sync successful
    
    ArgoCD->>ArgoCD: Update application status:<br/>Healthy, Synced
    
    Note over ArgoCD: Send notification to Slack
```

---

## Diagram 8: Monitoring & Observability Stack

```mermaid
graph TB
    subgraph "Application Pods"
        POD1[Observer Pods]
        POD2[Analyzer Pods]
        POD3[PostgreSQL Pods]
    end
    
    subgraph "Metrics Collection"
        POD1 -->|/metrics| PROM[Prometheus]
        POD2 -->|/metrics| PROM
        POD3 -->|pg_exporter| PROM
        
        NODE[Node Exporter<br/>DaemonSet] -->|Node metrics| PROM
        KUBE[kube-state-metrics] -->|K8s objects| PROM
    end
    
    subgraph "Visualization"
        PROM -->|PromQL queries| GRAFANA[Grafana Dashboards]
    end
    
    subgraph "Logging"
        POD1 & POD2 & POD3 -->|stdout/stderr| FLUENTD[Fluentd DaemonSet]
        FLUENTD -->|Parse & forward| LOKI[Loki]
        LOKI -->|LogQL queries| GRAFANA
    end
    
    subgraph "Distributed Tracing"
        POD1 & POD2 -->|Span data| JAEGER[Jaeger Collector]
        JAEGER -->|Trace queries| GRAFANA
    end
    
    subgraph "Alerting"
        PROM -->|Alert rules| ALERTMGR[Alertmanager]
        ALERTMGR -->|Notifications| SLACK[Slack/PagerDuty]
    end
    
    style PROM fill:#ff9966
    style GRAFANA fill:#66ccff
    style LOKI fill:#99cc66
    style JAEGER fill:#cc99ff
```

---

## Diagram 9: Security Boundaries

```mermaid
graph TB
    subgraph "External Traffic"
        INTERNET[Internet]
    end
    
    subgraph "Ingress Layer"
        INTERNET -->|HTTPS| LB[Load Balancer]
        LB -->|TLS Termination| IG[Istio Ingress Gateway]
    end
    
    subgraph "Istio Service Mesh (mTLS Zone)"
        IG -.->|mTLS| SVC1[Service: observer-twitter]
        SVC1 -.->|mTLS| SVC2[Service: postgres]
        
        subgraph "Network Policy: Allow Observer -> Storage"
            SVC1
            SVC2
        end
    end
    
    subgraph "Pod Security"
        SVC1 --> POD1["Pod<br/>securityContext:<br/>runAsNonRoot: true<br/>readOnlyRootFilesystem: true"]
        SVC2 --> POD2["Pod<br/>Pod Security Standard:<br/>Restricted"]
    end
    
    subgraph "RBAC"
        POD1 -->|Uses| SA1[ServiceAccount:<br/>observer-sa]
        SA1 -->|Bound to| ROLE1[Role:<br/>read secrets,<br/>write to kafka]
    end
    
    subgraph "Secrets Management"
        POD1 -->|Reads| ES[External Secret]
        ES -.->|Synced from| VAULT[HashiCorp Vault]
    end
    
    subgraph "Runtime Security"
        POD1 & POD2 -->|Monitored by| FALCO[Falco DaemonSet]
        FALCO -->|Alerts on| ANOMALY[Suspicious Activity]
    end
    
    style IG fill:#ffcccc
    style SVC1 fill:#ffffcc
    style SVC2 fill:#ffffcc
    style VAULT fill:#ccffcc
    style FALCO fill:#ffccff
```

---

## Diagram 10: Disaster Recovery Flow

```mermaid
flowchart TD
    START[Disaster Event:<br/>Node Failure] --> DETECT{Kubernetes<br/>Detects Failure}
    
    DETECT -->|StatefulSet Pod Down| RECREATE[Create New Pod<br/>with Same PVC]
    DETECT -->|Deployment Pod Down| SPAWN[Spawn New Pod<br/>on Healthy Node]
    DETECT -->|Entire Node Down| EVICT[Evict All Pods<br/>from Failed Node]
    
    RECREATE --> MOUNT[Mount Existing<br/>Persistent Volume]
    MOUNT --> RECOVER_DB[PostgreSQL<br/>Auto-Recovery]
    RECOVER_DB --> CHECKLAG{Check<br/>Replication Lag}
    
    CHECKLAG -->|Lag < 1 min| PROMOTE_READY[Replica Ready]
    CHECKLAG -->|Lag > 1 min| WAIT[Wait for Sync]
    WAIT --> CHECKLAG
    
    SPAWN --> PULL[Pull Image]
    PULL --> START_POD[Start Container]
    START_POD --> HEALTH[Run Health Checks]
    
    EVICT --> CA[Cluster Autoscaler]
    CA --> NEWNODE[Provision New Node]
    NEWNODE --> SCHEDULE[Schedule Evicted Pods]
    
    SCHEDULE --> START_POD
    PROMOTE_READY --> READY[Service Restored]
    HEALTH --> READY
    
    READY --> MONITOR[Continuous Monitoring]
    
    subgraph "Velero Backup"
        BACKUP_SCHED[Scheduled Backup<br/>Every 6 hours]
        BACKUP_SCHED -.-> S3_BACKUP[S3 Backup Storage]
        
        DISASTER_TOTAL[Complete Cluster Loss] -.-> RESTORE[Velero Restore]
        RESTORE -.-> S3_BACKUP
        RESTORE --> NEW_CLUSTER[New Cluster]
    end
    
    style START fill:#ff6666
    style READY fill:#66ff66
    style S3_BACKUP fill:#6666ff,color:#fff
```

---

## Summary

These diagrams illustrate:

1. **Multi-Cluster Hierarchy**: How clusters are organized from Local to International
2. **Pod Architecture**: The internal structure of a Local node
3. **Service Mesh**: How Istio manages secure communication
4. **Auto-scaling**: The complete auto-scaling workflow
5. **Data Flow**: From observation through analysis to metadata sync
6. **Storage**: PostgreSQL StatefulSet with HA and backups
7. **GitOps**: How code changes are deployed automatically
8. **Observability**: The complete monitoring stack
9. **Security**: Layered security boundaries
10. **Disaster Recovery**: Automated recovery procedures

All diagrams use Mermaid syntax and will render correctly in GitHub, GitLab, and most modern markdown viewers.
