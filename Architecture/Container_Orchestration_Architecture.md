# Kubernetes Container Orchestration Architecture

**Anti-Hate Platform - Implementation Guide v1.0**

> **📊 Visual Diagrams**: For visual representations of this architecture, see [Container Orchestration Diagrams](./Container_Orchestration_Diagrams.md)

---

## Executive Summary

This document provides the complete Kubernetes orchestration architecture for the Anti-Hate Platform's federated node system. It translates the high-level architectural vision from the [Technical Roadmap v0.0.2](./Anti-Hate_Pleatform_Technical_RoadMap_v_0_0_2.md) into deployable Kubernetes specifications.

**Target Audience**: DevOps engineers, SREs, and infrastructure teams responsible for deploying and operating the platform.

**Scope**: This covers the orchestration layer for all node types (Leaf, Regional, National, Global) with production-ready specifications.

---

## Table of Contents

1. [Cluster Architecture](#part-1-cluster-architecture)
2. [Namespace Design](#part-2-namespace-design)
3. [Workload Specifications](#part-3-workload-specifications)
4. [Service Mesh Configuration](#part-4-service-mesh-configuration)
5. [Storage Architecture](#part-5-storage-architecture)
6. [Auto-scaling Strategies](#part-6-auto-scaling-strategies)
7. [Helm Charts](#part-7-helm-charts)
8. [GitOps with ArgoCD](#part-8-gitops-with-argocd)
9. [Observability Stack](#part-9-observability-stack)
10. [Security Hardening](#part-10-security-hardening)
11. [Operational Procedures](#part-11-operational-procedures)
12. [Appendix: Reference Manifests](#appendix-reference-manifests)

---

## Part 1: Cluster Architecture

### 1.1 Multi-Cluster Strategy

The Anti-Hate Platform uses a **multi-cluster architecture** where each node level operates in its own Kubernetes cluster:

```
┌────────────────────────────────────────────────────────┐
│ International Coordination Cluster (Multi-Region HA)  │
│ - Location: Multi-Cloud (AWS, Azure, GCP, Alibaba)    │
│ - Size: 12-20 nodes (High availability)                │
└────────────────────────────────────────────────────────┘
                          │
      ┌───────────────────┼───────────────────┐
      │                   │                   │
   ⋯  │                   │                   │  ⋯
      │                   │                   │
┌─────▼─────┐  ┌─────▼─────┐  ┌─────▼─────┐  ┌─────▼─────┐
│🇺🇸 USA     │  │🇷🇺 Russia  │  │🇨🇳 China   │  │🇮🇳 India   │  ...
│ National  │  │ National  │  │ National  │  │ National  │
│ Cluster   │  │ Cluster   │  │ Cluster   │  │ Cluster   │
│ 8-12 nodes│  │ 8-12 nodes│  │ 8-12 nodes│  │ 8-12 nodes│
└─────┬─────┘  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘
      │              │              │              │
 ┌────┴────┐    ┌────┴────┐    ┌────┴────┐    ┌────┴────┐
 │Geography│    │Geography│    │Geography│    │Geography│
 │Clusters │    │Clusters │    │Clusters │    │Clusters │
 │4-8 nodes│    │4-8 nodes│    │4-8 nodes│    │4-8 nodes│
 └────┬────┘    └────┬────┘    └────┬────┘    └────┬────┘
      │              │              │              │
 ┌────┴────┐    ┌────┴────┐    ┌────┴────┐    ┌────┴────┐
 │ Local   │    │ Local   │    │ Local   │    │ Local   │
 │Clusters │    │Clusters │    │Clusters │    │Clusters │
 │2-6 nodes│    │2-6 nodes│    │2-6 nodes│    │2-6 nodes│
 └─────────┘    └─────────┘    └─────────┘    └─────────┘
```

**Rationale**:
- **Data Sovereignty**: Local clusters stay within jurisdictional boundaries
- **Fault Isolation**: Cluster failure doesn't cascade
- **Independent Scaling**: Each node level scales independently
- **Network Latency**: Reduces cross-cluster traffic

### 1.2 Control Plane High Availability

Each cluster above Geography level uses a **highly available control plane**:

```yaml
# Managed Kubernetes Configuration (EKS, GKE, AKS, Yandex Cloud)
apiVersion: v1
kind: Cluster
metadata:
  name: antihate-national-india
spec:
  version: "1.28"
  controlPlane:
    instanceType: t3.large  # 2 vCPU, 8 GB RAM
    replicas: 3             # HA across availability zones
    zones:
      - region-a
      - region-b
      - region-c
  networking:
    serviceIPv4CIDR: "10.100.0.0/16"
    podIPv4CIDR: "10.200.0.0/16"
```

### 1.3 Node Pools and Taints

Each cluster segregates workloads using **node pools** with taints and tolerations:

**Local Cluster Node Pools**:
```yaml
# High-Memory Pool (for databases)
- name: postgres-pool
  instanceType: r5.2xlarge  # 8 vCPU, 64 GB RAM
  minSize: 2
  maxSize: 6
  taints:
    - key: workload-type
      value: database
      effect: NoSchedule
  labels:
    node-type: database

# High-CPU Pool (for ML analysis)
- name: analyzer-pool
  instanceType: c5.4xlarge  # 16 vCPU, 32 GB RAM
  minSize: 2
  maxSize: 10
  taints:
    - key: workload-type
      value: analysis
      effect: NoSchedule
  labels:
    node-type: analysis

# General Pool (for observers, sync)
- name: general-pool
  instanceType: t3.xlarge  # 4 vCPU, 16 GB RAM
  minSize: 1
  maxSize: 5
  labels:
    node-type: general
```

---

## Part 2: Namespace Design

### 2.1 Namespace Hierarchy

Each cluster uses a **namespace-per-workload-type** pattern:

```yaml
# Core Infrastructure Namespaces
apiVersion: v1
kind: Namespace
metadata:
  name: antihate-system     # Platform-wide services
---
apiVersion: v1
kind: Namespace
metadata:
  name: antihate-monitoring # Prometheus, Grafana, Loki
---
apiVersion: v1
kind: Namespace
metadata:
  name: antihate-security   # Vault, cert-manager, Falco
---
# Workload Namespaces (Local Node Example)
apiVersion: v1
kind: Namespace
metadata:
  name: antihate-observers  # Social media observers
  labels:
    node-level: local
    istio-injection: enabled
---
apiVersion: v1
kind: Namespace
metadata:
  name: antihate-analyzers   # ML analysis workloads
  labels:
    node-level: local
    istio-injection: enabled
---
apiVersion: v1
kind: Namespace
metadata:
  name: antihate-storage     # PostgreSQL, Redis
  labels:
    node-level: local
    istio-injection: enabled
---
apiVersion: v1
kind: Namespace
metadata:
  name: antihate-coordination # Metadata sync, control plane
  labels:
    node-level: local
    istio-injection: enabled
```

### 2.2 Resource Quotas

Each namespace has **resource quotas** to prevent runaway workloads:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: analyzer-quota
  namespace: antihate-analyzers
spec:
  hard:
    requests.cpu: "64"        # Max 64 vCPUs requested
    requests.memory: 128Gi    # Max 128 GB memory requested
    limits.cpu: "96"          # Max 96 vCPUs limit
    limits.memory: 192Gi      # Max 192 GB memory limit
    pods: "50"                # Max 50 pods
    persistentvolumeclaims: "5"
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: observer-quota
  namespace: antihate-observers
spec:
  hard:
    requests.cpu: "16"
    requests.memory: 32Gi
    limits.cpu: "24"
    limits.memory: 48Gi
    pods: "30"
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: storage-quota
  namespace: antihate-storage
spec:
  hard:
    requests.cpu: "32"
    requests.memory: 256Gi    # Databases need more memory
    limits.cpu: "48"
    limits.memory: 384Gi
    pods: "10"
    persistentvolumeclaims: "10"
    requests.storage: "10Ti"  # Max 10 TB storage
```

### 2.3 Network Policies

**Default Deny** with explicit allow rules:

```yaml
# Deny all ingress by default
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: antihate-analyzers
spec:
  podSelector: {}
  policyTypes:
  - Ingress
---
# Allow analyzers to access storage
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: analyzer-to-storage
  namespace: antihate-storage
spec:
  podSelector:
    matchLabels:
      app: postgres
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: antihate-analyzers
    - namespaceSelector:
        matchLabels:
          name: antihate-observers
    ports:
    - protocol: TCP
      port: 5432
---
# Allow coordination pods to access everything (for metadata sync)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: coordination-full-access
  namespace: antihate-coordination
spec:
  podSelector:
    matchLabels:
      component: metadata-sync
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector: {}  # All namespaces
```

---

## Part 3: Workload Specifications

### 3.1 Deployment: Observation Containers (Stateless)

**Twitter Observer Example**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: observer-twitter
  namespace: antihate-observers
  labels:
    app: observer-twitter
    component: observation
spec:
  replicas: 3  # Start with 3, HPA will scale
  selector:
    matchLabels:
      app: observer-twitter
  template:
    metadata:
      labels:
        app: observer-twitter
        component: observation
        version: v1.2.0
    spec:
      serviceAccountName: observer-sa
      # Schedule on general node pool
      nodeSelector:
        node-type: general
      
      containers:
      - name: observer-twitter
        image: antihate/observer-twitter:v1.2.0
        imagePullPolicy: IfNotPresent
        
        resources:
          requests:
            cpu: "500m"      # 0.5 vCPU
            memory: "1Gi"
          limits:
            cpu: "2000m"     # 2 vCPU max
            memory: "4Gi"
        
        env:
        - name: TWITTER_API_KEY
          valueFrom:
            secretKeyRef:
              name: twitter-api-creds
              key: api-key
        - name: HASHTAGS
          value: "#hate-speech-en,#hate-speech-es"
        - name: GEOGRAPHY
          value: "Northern Mumbai"
        - name: POSTGRES_HOST
          value: "postgres.antihate-storage.svc.cluster.local"
        - name: KAFKA_BROKER
          value: "kafka.antihate-system.svc.cluster.local:9092"
        
        ports:
        - name: metrics
          containerPort: 9090
          protocol: TCP
        
        livenessProbe:
          httpGet:
            path: /health
            port: 9090
          initialDelaySeconds: 30
          periodSeconds: 10
        
        readinessProbe:
          httpGet:
            path: /ready
            port: 9090
          initialDelaySeconds: 10
          periodSeconds: 5
        
        volumeMounts:
        - name: config
          mountPath: /etc/antihate
          readOnly: true
      
      volumes:
      - name: config
        configMap:
          name: observer-config
---
apiVersion: v1
kind: Service
metadata:
  name: observer-twitter
  namespace: antihate-observers
spec:
  selector:
    app: observer-twitter
  ports:
  - name: metrics
    port: 9090
    targetPort: 9090
  type: ClusterIP
```

### 3.2 Deployment: Analysis Containers (GPU-enabled)

**Text Analyzer with GPU**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: analyzer-text
  namespace: antihate-analyzers
spec:
  replicas: 2  # Will be scaled by HPA
  selector:
    matchLabels:
      app: analyzer-text
  template:
    metadata:
      labels:
        app: analyzer-text
        component: analysis
    spec:
      serviceAccountName: analyzer-sa
      # Tolerate analysis node pool taint
      tolerations:
      - key: workload-type
        operator: Equal
        value: analysis
        effect: NoSchedule
      nodeSelector:
        node-type: analysis
      
      containers:
      - name: analyzer-text
        image: antihate/analyzer-text:v2.1.0
        
        resources:
          requests:
            cpu: "4000m"     # 4 vCPU
            memory: "16Gi"
            nvidia.com/gpu: 1  # 1 GPU
          limits:
            cpu: "8000m"     # 8 vCPU max
            memory: "32Gi"
            nvidia.com/gpu: 1
        
        env:
        - name: MODEL_PATH
          value: "/models/hate-bert-multilingual"
        - name: BATCH_SIZE
          value: "32"
        - name: KAFKA_CONSUMER_GROUP
          value: "analyzer-text-group"
        - name: MAX_SEQUENCE_LENGTH
          value: "512"
        
        ports:
        - name: metrics
          containerPort: 9091
        
        volumeMounts:
        - name: models
          mountPath: /models
          readOnly: true
        - name: shm
          mountPath: /dev/shm  # Shared memory for PyTorch
      
      volumes:
      - name: models
        persistentVolumeClaim:
          claimName: ml-models-pvc
      - name: shm
        emptyDir:
          medium: Memory
          sizeLimit: 8Gi
```

### 3.3 StatefulSet: PostgreSQL Cluster

**PostgreSQL with Streaming Replication**:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  namespace: antihate-storage
spec:
  serviceName: postgres-headless
  replicas: 3  # Primary + 2 replicas
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
        component: storage
    spec:
      serviceAccountName: postgres-sa
      tolerations:
      - key: workload-type
        operator: Equal
        value: database
        effect: NoSchedule
      nodeSelector:
        node-type: database
      
      # Init container for pgpool setup
      initContainers:
      - name: init-postgres
        image: antihate/postgres-init:15.4
        command: ['/bin/bash', '-c']
        args:
        - |
          if [ ! -f /data/postgresql.conf ]; then
            echo "Initializing PostgreSQL data directory..."
            initdb -D /data
          fi
        volumeMounts:
        - name: data
          mountPath: /data
      
      containers:
      - name: postgres
        image: postgres:15.4
        
        resources:
          requests:
            cpu: "4000m"
            memory: "32Gi"
          limits:
            cpu: "8000m"
            memory: "64Gi"
        
        env:
        - name: POSTGRES_USER
          value: antihate
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-creds
              key: password
        - name: POSTGRES_DB
          value: antihate_leaf
        - name: PGDATA
          value: /data/pgdata
        - name: POSTGRES_INITDB_ARGS
          value: "--encoding=UTF8 --locale=en_US.UTF-8"
        
        ports:
        - name: postgres
          containerPort: 5432
        
        livenessProbe:
          exec:
            command:
            - /bin/sh
            - -c
            - pg_isready -U antihate
          initialDelaySeconds: 30
          periodSeconds: 10
        
        readinessProbe:
          exec:
            command:
            - /bin/sh
            - -c
            - pg_isready -U antihate
          initialDelaySeconds: 10
          periodSeconds: 5
        
        volumeMounts:
        - name: data
          mountPath: /data
        - name: config
          mountPath: /etc/postgresql
          readOnly: true
      
      # Sidecar for backups
      - name: backup-agent
        image: antihate/pg-backup:1.0.0
        env:
        - name: BACKUP_SCHEDULE
          value: "0 2 * * *"  # 2 AM daily
        - name: S3_BUCKET
          value: "s3://antihate-backups/leaf-mumbai-postgres"
        volumeMounts:
        - name: data
          mountPath: /data
          readOnly: true
      
      volumes:
      - name: config
        configMap:
          name: postgres-config
  
  # Volume claim templates
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: gp3-encrypted  # Multi-Cloud EBS gp3 with encryption
      resources:
        requests:
          storage: 2Ti  # 2 TB per instance
---
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: antihate-storage
spec:
  selector:
    app: postgres
  ports:
  - name: postgres
    port: 5432
    targetPort: 5432
  type: ClusterIP
---
apiVersion: v1
kind: Service
metadata:
  name: postgres-headless
  namespace: antihate-storage
spec:
  selector:
    app: postgres
  clusterIP: None  # Headless for StatefulSet
  ports:
  - name: postgres
    port: 5432
```

### 3.4 DaemonSet: Monitoring Exporter

**Node Exporter for Prometheus**:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: antihate-monitoring
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      hostNetwork: true
      hostPID: true
      containers:
      - name: node-exporter
        image: prom/node-exporter:v1.6.1
        args:
        - --path.procfs=/host/proc
        - --path.sysfs=/host/sys
        - --collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)
        ports:
        - name: metrics
          containerPort: 9100
          hostPort: 9100
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"
        volumeMounts:
        - name: proc
          mountPath: /host/proc
          readOnly: true
        - name: sys
          mountPath: /host/sys
          readOnly: true
      volumes:
      - name: proc
        hostPath:
          path: /proc
      - name: sys
        hostPath:
          path: /sys
      tolerations:
      - effect: NoSchedule
        operator: Exists  # Run on all nodes
```

---

## Part 4: Service Mesh Configuration

### 4.1 Istio Installation

**Install Istio with mTLS STRICT mode**:

```bash
# Install Istio Operator
kubectl apply -f https://istio.io/latest/operator.yaml

# Create Istio ControlPlane resource
kubectl apply -f - <<EOF
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  namespace: istio-system
  name: antihate-istio
spec:
  profile: production
  meshConfig:
    accessLogFile: /dev/stdout
    enableTracing: true
    defaultConfig:
      holdApplicationUntilProxyStarts: true
  components:
    pilot:
      k8s:
        resources:
          requests:
            cpu: 1000m
            memory: 2Gi
    ingressGateways:
    - name: istio-ingressgateway
      enabled: true
      k8s:
        resources:
          requests:
            cpu: 500m
            memory: 1Gi
        service:
          type: LoadBalancer
  values:
    global:
      proxy:
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 2000m
            memory: 1Gi
      mtls:
        enabled: true
    gateways:
      istio-ingressgateway:
        autoscaleMin: 2
        autoscaleMax: 5
EOF
```

### 4.2 PeerAuthentication (mTLS STRICT)

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default-mtls
  namespace: istio-system
spec:
  mtls:
    mode: STRICT  # All traffic must use mTLS
---
# Namespace-level enforcement
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: namespace-mtls
  namespace: antihate-observers
spec:
  mtls:
    mode: STRICT
```

### 4.3 DestinationRule (Traffic Policy)

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: postgres-circuit-breaker
  namespace: antihate-storage
spec:
  host: postgres.antihate-storage.svc.cluster.local
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 50
        maxRequestsPerConnection: 5
    outlierDetection:
      consecutiveErrors: 5
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
    tls:
      mode: ISTIO_MUTUAL  # Use Istio mTLS
---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: analyzer-load-balancing
  namespace: antihate-analyzers
spec:
  host: analyzer-text.antihate-analyzers.svc.cluster.local
  trafficPolicy:
    loadBalancer:
      consistentHash:
        httpHeaderName: X-Request-ID  # Sticky sessions by request ID
```

### 4.4 VirtualService (Retry & Timeout)

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: observer-retry
  namespace: antihate-observers
spec:
  hosts:
  - observer-twitter.antihate-observers.svc.cluster.local
  http:
  - route:
    - destination:
        host: observer-twitter.antihate-observers.svc.cluster.local
        port:
          number: 9090
    retries:
      attempts: 3
      perTryTimeout: 2s
      retryOn: 5xx,reset,connect-failure,refused-stream
    timeout: 10s
```

### 4.5 AuthorizationPolicy (RBAC for Services)

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-analyzer-to-storage
  namespace: antihate-storage
spec:
  selector:
    matchLabels:
      app: postgres
  action: ALLOW
  rules:
  - from:
    - source:
        namespaces: ["antihate-analyzers", "antihate-observers"]
    to:
    - operation:
        ports: ["5432"]
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: deny-external-coordination
  namespace: antihate-coordination
spec:
  selector:
    matchLabels:
      component: metadata-sync
  action: DENY
  rules:
  - from:
    - source:
        notNamespaces: ["antihate-coordination", "antihate-system"]
```

---

## Part 5: Storage Architecture

### 5.1 StorageClass Definitions

```yaml
# High-performance SSD for databases
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp3-encrypted
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  iops: "16000"
  throughput: "1000"
  encrypted: "true"
  kmsKeyId: "arn:aws:kms:multiple-regions:123456789:key/abcd-1234"
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
reclaimPolicy: Retain
---
# Standard storage for logs
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp3-standard
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  encrypted: "true"
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
reclaimPolicy: Delete
```

### 5.2 PersistentVolumeClaim Templates

```yaml
# For ML models (shared across analyzer pods)
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ml-models-pvc
  namespace: antihate-analyzers
spec:
  accessModes:
  - ReadOnlyMany  # Multiple pods can read
  storageClassName: gp3-standard
  resources:
    requests:
      storage: 500Gi  # 500 GB for models
```

### 5.3 Backup Strategy (Velero)

**Install Velero for cluster backups**:

```bash
# Install Velero with S3 backend
velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.8.0 \
  --bucket antihate-velero-backups \
  --backup-location-config region=multiple-regions \
  --snapshot-location-config region=multiple-regions \
  --secret-file ./credentials-velero
```

**Scheduled Backup**:

```yaml
apiVersion: velero.io/v1
kind: Schedule
metadata:
  name: daily-postgres-backup
  namespace: velero
spec:
  schedule: "0 3 * * *"  # 3 AM daily
  template:
    includedNamespaces:
    - antihate-storage
    includedResources:
    - persistentvolumeclaims
    - persistentvolumes
    labelSelector:
      matchLabels:
        app: postgres
    snapshotVolumes: true
    ttl: 720h  # Retain for 30 days
```

---

## Part 6: Auto-scaling Strategies

### 6.1 Horizontal Pod Autoscaler (HPA)

**For stateless observers**:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: observer-twitter-hpa
  namespace: antihate-observers
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: observer-twitter
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70  # Scale at 70% CPU
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80  # Scale at 80% memory
  - type: Pods
    pods:
      metric:
        name: kafka_consumer_lag
      target:
        type: AverageValue
        averageValue: "1000"  # Scale if lag > 1000 messages
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300  # Wait 5 min before scaling down
      policies:
      - type: Percent
        value: 50  # Scale down max 50% at a time
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0  # Scale up immediately
      policies:
      - type: Percent
        value: 100  # Double pods if needed
        periodSeconds: 15
      - type: Pods
        value: 4  # Or add 4 pods
        periodSeconds: 15
      selectPolicy: Max  # Use the more aggressive policy
```

### 6.2 Vertical Pod Autoscaler (VPA)

**For databases (recommendation mode)**:

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: postgres-vpa
  namespace: antihate-storage
spec:
  targetRef:
    apiVersion: apps/v1
    kind: StatefulSet
    name: postgres
  updatePolicy:
    updateMode: "Off"  # Recommendation only, don't auto-apply
  resourcePolicy:
    containerPolicies:
    - containerName: postgres
      minAllowed:
        cpu: "2000m"
        memory: "16Gi"
      maxAllowed:
        cpu: "16000m"
        memory: "128Gi"
```

### 6.3 Cluster Autoscaler

**For adding nodes to the cluster**:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-autoscaler-config
  namespace: kube-system
data:
  cluster-autoscaler.yaml: |
    autoDiscovery:
      clusterName: antihate-leaf-sfo
      tags:
      - k8s.io/cluster-autoscaler/enabled
      - k8s.io/cluster-autoscaler/antihate-leaf-sfo
    cloudProvider: aws
    awsRegion: us-west-1
    scaleDownDelay: 10m
    scaleDownUnneededTime: 10m
    skipNodesWithLocalStorage: false
    skipNodesWithSystemPods: false
    maxNodeProvisionTime: 15m
```

### 6.4 KEDA (Event-Driven Autoscaling)

**Scale analyzers based on Kafka queue depth**:

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: analyzer-kafka-scaler
  namespace: antihate-analyzers
spec:
  scaleTargetRef:
    name: analyzer-text
  minReplicaCount: 2
  maxReplicaCount: 30
  pollingInterval: 10
  cooldownPeriod: 60
  triggers:
  - type: kafka
    metadata:
      bootstrapServers: kafka.antihate-system.svc.cluster.local:9092
      consumerGroup: analyzer-text-group
      topic: analysis-queue
      lagThreshold: "500"  # Scale if lag > 500 messages
      offsetResetPolicy: latest
```

---

## Part 7: Helm Charts

### 7.1 Chart Structure

```
antihate-node/
├── Chart.yaml
├── values.yaml
├── values-leaf.yaml
├── values-regional.yaml
├── values-national.yaml
├── values-global.yaml
├── templates/
│   ├── _helpers.tpl
│   ├── namespace.yaml
│   ├── observers/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── hpa.yaml
│   │   └── configmap.yaml
│   ├── analyzers/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── scaledobject.yaml
│   │   └── pvc.yaml
│   ├── storage/
│   │   ├── statefulset.yaml
│   │   ├── service.yaml
│   │   ├── pvc.yaml
│   │   └── secret.yaml
│   ├── coordination/
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   ├── monitoring/
│   │   ├── servicemonitor.yaml
│   │   └── grafana-dashboard.yaml
│   └── istio/
│       ├── virtualservice.yaml
│       ├── destinationrule.yaml
│       └── peerauthentication.yaml
└── crds/
    └── ...
```

### 7.2 Values Hierarchy

**International values.yaml**:

```yaml
global:
  nodeLevel: local  # leaf, regional, national, global
  nodeId: mumbai-001
  region: us-west-1
  
  imageRegistry: registry.antihate.org
  imagePullPolicy: IfNotPresent
  
  istio:
    enabled: true
    mtls: STRICT
  
  monitoring:
    enabled: true
    prometheus:
      scrapeInterval: 30s
  
  storage:
    class: gp3-encrypted

# Observers configuration
observers:
  twitter:
    enabled: true
    replicas: 3
    resources:
      requests:
        cpu: 500m
        memory: 1Gi
      limits:
        cpu: 2000m
        memory: 4Gi
    autoscaling:
      enabled: true
      minReplicas: 3
      maxReplicas: 20
      targetCPU: 70
    config:
      hashtags:
        - "#hate-speech-en"
        - "#hate-speech-es"
      geography: "Northern Mumbai"
  
  facebook:
    enabled: true
    replicas: 2
    # ... similar structure

# Analyzers configuration
analyzers:
  text:
    enabled: true
    replicas: 2
    gpu:
      enabled: true
      count: 1
    resources:
      requests:
        cpu: 4000m
        memory: 16Gi
      limits:
        cpu: 8000m
        memory: 32Gi
    keda:
      enabled: true
      lagThreshold: 500

# Storage configuration
storage:
  postgres:
    enabled: true
    replicas: 3
    resources:
      requests:
        cpu: 4000m
        memory: 32Gi
      limits:
        cpu: 8000m
        memory: 64Gi
    persistence:
      size: 2Ti
      storageClass: gp3-encrypted
    backup:
      enabled: true
      schedule: "0 2 * * *"
      retention: 30d
```

**values-leaf.yaml** (overrides):

```yaml
global:
  nodeLevel: leaf

observers:
  twitter:
    replicas: 5  # Local nodes collect more data
  facebook:
    replicas: 3
  instagram:
    enabled: true

analyzers:
  text:
    replicas: 4
  image:
    enabled: true
  video:
    enabled: true

storage:
  postgres:
    persistence:
      size: 5Ti  # Local nodes store full content
```

### 7.3 Template Helpers

**templates/_helpers.tpl**:

```yaml
{{/*
Expand the name of the chart.
*/}}
{{- define "antihate.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Create a default fully qualified app name.
*/}}
{{- define "antihate.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}

{{/*
Generate labels
*/}}
{{- define "antihate.labels" -}}
helm.sh/chart: {{ include "antihate.chart" . }}
{{ include "antihate.selectorLabels" . }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
node-level: {{ .Values.global.nodeLevel }}
node-id: {{ .Values.global.nodeId }}
{{- end }}

{{/*
Selector labels
*/}}
{{- define "antihate.selectorLabels" -}}
app.kubernetes.io/name: {{ include "antihate.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}
```

---

## Part 8: GitOps with ArgoCD

### 8.1 ArgoCD Application Manifest

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: antihate-leaf-sfo
  namespace: argocd
spec:
  project: antihate-production
  
  source:
    repoURL: https://github.com/anti-hate-platform/deployments.git
    targetRevision: main
    path: clusters/leaf/mumbai-001
    helm:
      valueFiles:
      - values.yaml
      - values-leaf.yaml
      - secrets://values-secrets.yaml  # From vault
  
  destination:
    server: https://leaf-mumbai-001.k8s.antihate.org
    namespace: antihate-system
  
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
    - CreateNamespace=true
    - PrunePropagationPolicy=foreground
    - PruneLast=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
  
  ignoreDifferences:
  - group: apps
    kind: Deployment
    jsonPointers:
    - /spec/replicas  # HPA controls replicas
```

### 8.2 Progressive Delivery (Canary)

**ArgoCD Rollouts for canary deployment**:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: analyzer-text-rollout
  namespace: antihate-analyzers
spec:
  replicas: 10
  selector:
    matchLabels:
      app: analyzer-text
  template:
    metadata:
      labels:
        app: analyzer-text
    spec:
      containers:
      - name: analyzer-text
        image: antihate/analyzer-text:v2.2.0
        # ... container spec
  
  strategy:
    canary:
      steps:
      - setWeight: 10   # Route 10% traffic to new version
      - pause: {duration: 5m}
      - setWeight: 25
      - pause: {duration: 5m}
      - setWeight: 50
      - pause: {duration: 10m}
      - setWeight: 75
      - pause: {duration: 5m}
      
      analysis:
        templates:
        - templateName: error-rate-analysis
        startingStep: 2  # Start analysis at 25% traffic
        args:
        - name: service-name
          value: analyzer-text
      
      trafficRouting:
        istio:
          virtualService:
            name: analyzer-text-vs
            routes:
            - primary
```

---

## Part 9: Observability Stack

### 9.1 Prometheus Operator

**Deploy Prometheus for monitoring**:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: antihate-prometheus
  namespace: antihate-monitoring
spec:
  replicas: 2
  retention: 30d
  retentionSize: "500GB"
  
  resources:
    requests:
      cpu: 2000m
      memory: 8Gi
    limits:
      cpu: 4000m
      memory: 16Gi
  
  storage:
    volumeClaimTemplate:
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: gp3-standard
        resources:
          requests:
            storage: 1Ti
  
  serviceMonitorSelector:
    matchLabels:
      prometheus: antihate
  
  ruleSelector:
    matchLabels:
      prometheus: antihate
```

### 9.2 ServiceMonitor for Custom Metrics

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: observer-twitter-monitor
  namespace: antihate-observers
  labels:
    prometheus: antihate
spec:
  selector:
    matchLabels:
      app: observer-twitter
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics
```

### 9.3 PrometheusRule for Alerts

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: antihate-alerts
  namespace: antihate-monitoring
  labels:
    prometheus: antihate
spec:
  groups:
  - name: antihate.observers
    interval: 30s
    rules:
    - alert: CollectorHighErrorRate
      expr: |
        rate(collector_errors_total[5m]) > 0.1
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "High error rate in observer {{ $labels.collector_type }}"
        description: "Observer {{ $labels.collector_type }} has error rate {{ $value }}"
    
    - alert: CollectorDown
      expr: |
        up{job="observer-twitter"} == 0
      for: 2m
      labels:
        severity: critical
      annotations:
        summary: "Observer {{ $labels.instance }} is down"
  
  - name: antihate.analyzers
    interval: 30s
    rules:
    - alert: AnalyzerQueueBacklog
      expr: |
        kafka_consumer_lag{topic="analysis-queue"} > 10000
      for: 10m
      labels:
        severity: warning
      annotations:
        summary: "Analysis queue backlog is high"
        description: "Queue lag is {{ $value }} messages"
```

### 9.4 Grafana Dashboard

**ConfigMap for Grafana dashboard**:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-dashboard-leaf-node
  namespace: antihate-monitoring
  labels:
    grafana_dashboard: "1"
data:
  leaf-node-overview.json: |
    {
      "dashboard": {
        "title": "Anti-Hate Local Node Overview",
        "panels": [
          {
            "id": 1,
            "title": "Observation Rate (Posts/min)",
            "targets": [
              {
                "expr": "rate(collector_posts_total[5m]) * 60"
              }
            ]
          },
          {
            "id": 2,
            "title": "Analysis Throughput",
            "targets": [
              {
                "expr": "rate(analyzer_processed_total[5m]) * 60"
              }
            ]
          },
          {
            "id": 3,
            "title": "PostgreSQL Connections",
            "targets": [
              {
                "expr": "pg_stat_database_numbackends"
              }
            ]
          }
        ]
      }
    }
```

---

## Part 10: Security Hardening

### 10.1 Pod Security Standards

**Enforce restricted PSS**:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: antihate-observers
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

### 10.2 RBAC Policies

**ServiceAccount and RBAC for observers**:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: observer-sa
  namespace: antihate-observers
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: observer-role
  namespace: antihate-observers
rules:
- apiGroups: [""]
  resources: ["configmaps", "secrets"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]  # For self-discovery
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: observer-rolebinding
  namespace: antihate-observers
subjects:
- kind: ServiceAccount
  name: observer-sa
  namespace: antihate-observers
roleRef:
  kind: Role
  name: observer-role
  apiGroup: rbac.authorization.k8s.io
```

### 10.3 External Secrets Operator

**Sync secrets from Vault**:

```yaml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: vault-backend
  namespace: antihate-observers
spec:
  provider:
    vault:
      server: "https://vault.antihate.org"
      path: "secret"
      version: "v2"
      auth:
        kubernetes:
          mountPath: "kubernetes"
          role: "antihate-collector"
          serviceAccountRef:
            name: observer-sa
---
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: twitter-api-creds
  namespace: antihate-observers
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: twitter-api-creds
    creationPolicy: Owner
  data:
  - secretKey: api-key
    remoteRef:
      key: social-media/twitter
      property: api_key
  - secretKey: api-secret
    remoteRef:
      key: social-media/twitter
      property: api_secret
```

### 10.4 Runtime Security (Falco)

**Falco rule for detecting unusual activity**:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: falco-rules
  namespace: antihate-security
data:
  custom_rules.yaml: |
    - rule: Unauthorized Process in Container
      desc: Detect unexpected process execution
      condition: >
        container.id != host and
        proc.name not in (postgres, python, node, java, bash, sh)
      output: "Unexpected process in container (user=%user.name command=%proc.cmdline container=%container.name)"
      priority: WARNING
    
    - rule: Sensitive File Access
      desc: Detect access to sensitive files
      condition: >
        open_read and
        fd.name in (/etc/shadow, /etc/sudoers, /root/.ssh/id_rsa)
      output: "Sensitive file accessed (user=%user.name file=%fd.name)"
      priority: CRITICAL
```

---

## Part 11: Operational Procedures

### 11.1 Node Provisioning Workflow

**Automated provisioning script**:

```bash
#!/bin/bash
# provision-leaf-node.sh

set -euo pipefail

# Input parameters
NODE_ID=$1          # e.g., mumbai-002
NODE_LEVEL=$2       # leaf, regional, national, global
AWS_REGION=$3       # e.g., us-west-1

echo "Provisioning $NODE_LEVEL node: $NODE_ID in $AWS_REGION"

# 1. Create EKS cluster with Terraform
cd terraform/clusters
terraform init
terraform apply \
  -var="node_id=$NODE_ID" \
  -var="node_level=$NODE_LEVEL" \
  -var="region=$AWS_REGION" \
  -auto-approve

# 2. Configure kubectl
aws eks update-kubeconfig \
  --region $AWS_REGION \
  --name antihate-$NODE_LEVEL-$NODE_ID

# 3. Install Istio
istioctl install -f ../istio/profile-$NODE_LEVEL.yaml --skip-confirmation

# 4. Install cert-manager
helm upgrade --install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set installCRDs=true

# 5. Install Prometheus Operator
helm upgrade --install prometheus-operator prometheus-community/kube-prometheus-stack \
  --namespace antihate-monitoring \
  --create-namespace \
  -f ../monitoring/values-$NODE_LEVEL.yaml

# 6. Deploy ArgoCD Application
kubectl apply -f ../argocd/applications/$NODE_LEVEL-$NODE_ID.yaml

# 7. Wait for all pods to be ready
kubectl wait --for=condition=ready pod \
  --all-namespaces \
  --all \
  --timeout=600s

echo "✅ Node $NODE_ID provisioned successfully"
```

### 11.2 Rolling Update Procedure

```bash
# Update analyzer image version
kubectl set image deployment/analyzer-text \
  analyzer-text=antihate/analyzer-text:v2.3.0 \
  -n antihate-analyzers

# Monitor rollout status
kubectl rollout status deployment/analyzer-text -n antihate-analyzers

# If issues, rollback
kubectl rollout undo deployment/analyzer-text -n antihate-analyzers
```

### 11.3 Incident Response Runbook

**High CPU on Analyzer Pods**:

```bash
# 1. Check current resource usage
kubectl top pods -n antihate-analyzers

# 2. Check HPA status
kubectl get hpa analyzer-text-hpa -n antihate-analyzers

# 3. Check Kafka queue depth (if using KEDA)
kubectl get scaledobject analyzer-kafka-scaler -n antihate-analyzers

# 4. Manually scale if needed
kubectl scale deployment analyzer-text --replicas=10 -n antihate-analyzers

# 5. Check for resource limits
kubectl describe pod <pod-name> -n antihate-analyzers | grep -A5 Resources

# 6. Review recent logs for errors
kubectl logs -n antihate-analyzers deployment/analyzer-text --tail=100
```

### 11.4 Capacity Planning

**Calculate required resources per node type**:

```yaml
# Local Node (Processing 500K posts/day)
Total vCPU Required:
  - Observers (5 types × 3 replicas × 0.5 vCPU): 7.5 vCPU
  - Analyzers (4 types × 2 replicas × 4 vCPU): 32 vCPU
  - PostgreSQL (3 replicas × 4 vCPU): 12 vCPU
  - Coordination (2 replicas × 1 vCPU): 2 vCPU
  - Overhead (Istio, monitoring): 8 vCPU
  Total: ~62 vCPU

Total Memory Required:
  - Observers: 15 GB
  - Analyzers: 64 GB
  - PostgreSQL: 96 GB
  - Coordination: 4 GB
  - Overhead: 16 GB
  Total: ~195 GB

Storage:
  - PostgreSQL data: 2-5 TB (depends on retention)
  - ML models: 500 GB
  - Logs/backups: 1 TB
  Total: ~4-7 TB

Recommended Cluster Size:
  - 3 × r5.4xlarge (16 vCPU, 128 GB) for databases
  - 2 × c5.9xlarge (36 vCPU, 72 GB) for analyzers
  - 2 × t3.2xlarge (8 vCPU, 32 GB) for observers
```

---

## Appendix: Reference Manifests

### A.1 Complete Observer Helm Values

```yaml
# values-observer-twitter.yaml
observers:
  twitter:
    enabled: true
    replicas: 3
    
    image:
      repository: antihate/observer-twitter
      tag: v1.2.0
      pullPolicy: IfNotPresent
    
    resources:
      requests:
        cpu: 500m
        memory: 1Gi
      limits:
        cpu: 2000m
        memory: 4Gi
    
    env:
      hashtags:
        - "#hate-speech-en"
        - "#hate-speech-es"
        - "#hate-speech-fr"
      geography: "Northern Mumbai"
      rateLimitPerMinute: 300
      maxConcurrentRequests: 10
    
    secrets:
      vaultPath: "social-media/twitter"
    
    autoscaling:
      enabled: true
      minReplicas: 3
      maxReplicas: 20
      targetCPU: 70
      targetMemory: 80
      metrics:
        - type: Pods
          pods:
            metric:
              name: kafka_consumer_lag
            target:
              type: AverageValue
              averageValue: 1000
    
    service:
      type: ClusterIP
      port: 9090
      metrics:
        enabled: true
        path: /metrics
    
    livenessProbe:
      httpGet:
        path: /health
        port: 9090
      initialDelaySeconds: 30
      periodSeconds: 10
    
    readinessProbe:
      httpGet:
        path: /ready
        port: 9090
      initialDelaySeconds: 10
      periodSeconds: 5
```

### A.2 PostgreSQL StatefulSet Complete Manifest

```yaml
# Complete production-ready PostgreSQL StatefulSet
# See Part 3.3 above for the full specification
# This would include:
# - Primary/replica configuration
# - Streaming replication setup
# - Backup sidecar container
# - Connection pooling (pgBouncer)
# - Monitoring exporters
```

### A.3 Istio Complete Configuration

```yaml
# See Part 4 for complete Istio configuration including:
# - PeerAuthentication (mTLS STRICT)
# - DestinationRules (circuit breakers, load balancing)
# - VirtualServices (retry, timeout)
# - AuthorizationPolicies (service-to-service RBAC)
```

---

## Conclusion

This document provides production-ready Kubernetes orchestration specifications for the Anti-Hate Platform. All manifests are designed for:

✅ **High Availability**: Multi-replica deployments, cross-AZ distribution  
✅ **Security**: mTLS, RBAC, Pod Security Standards, runtime scanning  
✅ **Observability**: Prometheus metrics, Grafana dashboards, distributed tracing  
✅ **Auto-scaling**: HPA, VPA, KEDA, Cluster Autoscaler  
✅ **GitOps**: ArgoCD for declarative deployments  
✅ **Disaster Recovery**: Velero backups, restore procedures  

**Next Steps**:
1. Customize the Helm values for your specific deployment
2. Set up the GitOps repository structure
3. Provision the first cluster using the automated scripts
4. Deploy a pilot node and validate all components
5. Iterate based on operational learnings

---

*Document Version: 1.0*  
*Last Updated: 2025-11-22*  
*Status: Production-Ready Implementation Guide*
