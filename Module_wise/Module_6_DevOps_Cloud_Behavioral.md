# Module 6: DevOps, Cloud & Behavioral

> **Scope:** Docker, Kubernetes, AWS, CI/CD (GitHub Actions, GitLab CI), Conflict Resolution, Project Ownership, Leadership
> **Questions:** 20 | **Critical:** 5 | **Coverage:** Product & Service-Based Companies | Sorted by interview frequency (descending)

---

## 🔴 CRITICAL / MUST-KNOW (Top 5)

---

### Q1. 🔴 🌐 How do you containerize a Spring Boot application with Docker? What are the best practices for production images?

**A production Docker image for Spring Boot should use multi-stage builds to separate the build from the runtime, run as a non-root user, use a minimal JRE base image (Eclipse Temurin), leverage Spring Boot's layered JAR for cache-efficient builds, and never exceed 200MB in size.**

```dockerfile
# ============= Stage 1: Build =============
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /build

# Cache dependencies (changes less frequently)
COPY pom.xml mvnw ./
COPY .mvn .mvn
RUN ./mvnw dependency:resolve -B

# Build application
COPY src ./src
RUN ./mvnw package -DskipTests -B

# Extract Spring Boot layers for caching
RUN java -Djarmode=layertools -jar target/*.jar extract --destination extracted

# ============= Stage 2: Runtime =============
FROM eclipse-temurin:21-jre-alpine AS runtime

# Security: non-root user
RUN addgroup -g 1001 appgroup && adduser -u 1001 -G appgroup -D appuser

WORKDIR /app

# Copy layers in order of change frequency (least → most)
COPY --from=builder /build/extracted/dependencies/ ./
COPY --from=builder /build/extracted/spring-boot-loader/ ./
COPY --from=builder /build/extracted/snapshot-dependencies/ ./
COPY --from=builder /build/extracted/application/ ./

# Health check
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD wget -qO- http://localhost:8080/actuator/health/liveness || exit 1

USER appuser
EXPOSE 8080

ENTRYPOINT ["java", \
  "-XX:+UseG1GC", \
  "-XX:MaxRAMPercentage=75.0", \
  "-Djava.security.egd=file:/dev/./urandom", \
  "org.springframework.boot.loader.launch.JarLauncher"]
```

**Key Best Practices:**

| Practice | Rationale |
|----------|-----------|
| Multi-stage build | Final image has no build tools (Maven, JDK) — smaller, more secure |
| `eclipse-temurin:*-alpine` | ~100MB smaller than debian-based images |
| Non-root user | Security — container compromise doesn't get root |
| Layered JAR | Dependency layers are cached — only application code layer changes per build |
| `MaxRAMPercentage=75.0` | JVM respects container memory limits |
| `.dockerignore` | Exclude `.git/`, `target/`, `node_modules/` from build context |

```
# .dockerignore
.git
target
*.md
.idea
.vscode
node_modules
```

**⚠️ Pitfalls:**
- **Never use `latest` tag** — it's mutable. Pin exact versions: `eclipse-temurin:21.0.3_9-jre-alpine`.
- **Never put secrets in the image** — use environment variables or mounted secrets.
- **`-XX:MaxRAMPercentage`** must account for non-heap memory (metaspace, thread stacks, NIO buffers). 75% leaves room.
- **Don't run `mvn package` in the final stage** — the final image should only contain the JRE and your JAR.

---

### Q2. 🔴 🏢 How do you deploy and manage applications on Kubernetes? Explain Deployments, Services, health checks, and HPA.

**Kubernetes manages containerized applications through Deployments (desired state + rolling updates), Services (stable networking), health probes (liveness + readiness for self-healing), and HPA (auto-scaling based on metrics) — together providing zero-downtime deployments and automatic recovery.**

```yaml
# deployment.yaml — Spring Boot app on K8s
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
  labels:
    app: order-service
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1       # At most 1 pod down during update
      maxSurge: 1              # At most 1 extra pod during update
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
        prometheus.io/path: "/actuator/prometheus"
    spec:
      serviceAccountName: order-service
      containers:
        - name: order-service
          image: registry.company.com/order-service:1.4.2  # Pinned version
          ports:
            - containerPort: 8080
          
          # Resource limits — CRITICAL for K8s scheduling
          resources:
            requests:
              cpu: "250m"        # 0.25 vCPU guaranteed
              memory: "512Mi"    # 512MB guaranteed
            limits:
              cpu: "1000m"       # Can burst to 1 vCPU
              memory: "1Gi"      # Hard limit — OOMKilled if exceeded
          
          # Health probes — K8s self-healing
          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 8080
            initialDelaySeconds: 30    # Wait for JVM startup
            periodSeconds: 10
            failureThreshold: 3        # 3 failures → restart pod
          
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: 8080
            initialDelaySeconds: 15
            periodSeconds: 5
            failureThreshold: 3        # 3 failures → remove from Service endpoints
          
          startupProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 5
            failureThreshold: 30       # 30 × 5s = 150s max startup time
          
          env:
            - name: SPRING_PROFILES_ACTIVE
              value: "prod"
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: order-db-credentials
                  key: password
          
          envFrom:
            - configMapRef:
                name: order-service-config

---
# service.yaml — Stable network endpoint
apiVersion: v1
kind: Service
metadata:
  name: order-service
spec:
  selector:
    app: order-service
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP

---
# HPA — auto-scaling
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: order-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service
  minReplicas: 3
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70   # Scale up when avg CPU > 70%
    - type: Pods
      pods:
        metric:
          name: http_requests_per_second
        target:
          type: AverageValue
          averageValue: "1000"     # Custom metric from Prometheus
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300  # Wait 5 min before scaling down
```

**Spring Boot Application Properties for K8s:**
```yaml
# application-prod.yml
management:
  health:
    livenessstate:
      enabled: true
    readinessstate:
      enabled: true
  endpoint:
    health:
      probes:
        enabled: true
      group:
        readiness:
          include: db, redis, kafka

spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s  # Graceful shutdown
server:
  shutdown: graceful                  # Finish in-flight requests before stopping
```

**⚠️ Pitfalls:**
- **Liveness ≠ Readiness** — liveness checks if the app is alive (restart if dead). Readiness checks if it can serve traffic (remove from LB if not ready). Don't include DB checks in liveness — a DB outage would restart ALL pods.
- **Missing `startupProbe`** — without it, liveness probe can kill slow-starting JVM apps before they initialize.
- **Memory limits too tight** — JVM has off-heap memory (Metaspace, thread stacks, NIO). Set memory limit 20-30% above `-Xmx`.
- **Graceful shutdown** — without `server.shutdown=graceful`, in-flight requests are dropped during rolling updates.

---

### Q3. 🔴 🌐 Design a CI/CD pipeline for a Spring Boot microservice. What stages should it include?

**A production CI/CD pipeline for Spring Boot should include: build → test (unit + integration) → static analysis → container build → security scan → deploy to staging → integration/smoke tests → production deploy with canary/blue-green strategy — fully automated from commit to production.**

```yaml
# GitHub Actions: .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # ════════════ Stage 1: Build & Test ════════════
  build-and-test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_DB: testdb
          POSTGRES_PASSWORD: testpass
        ports: ['5432:5432']
        options: --health-cmd pg_isready --health-interval 10s --health-timeout 5s --health-retries 5
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
          cache: 'maven'
      
      - name: Run unit tests
        run: ./mvnw test -B
      
      - name: Run integration tests
        run: ./mvnw verify -P integration-tests -B
        env:
          SPRING_DATASOURCE_URL: jdbc:postgresql://localhost:5432/testdb
      
      - name: Upload test results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: test-results
          path: target/surefire-reports/

  # ════════════ Stage 2: Code Quality ════════════
  code-quality:
    runs-on: ubuntu-latest
    needs: build-and-test
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for SonarQube
      
      - name: SonarQube Analysis
        run: ./mvnw sonar:sonar -Dsonar.host.url=${{ secrets.SONAR_URL }} -B
      
      - name: Check quality gate
        run: |
          STATUS=$(curl -s "${{ secrets.SONAR_URL }}/api/qualitygates/project_status?projectKey=order-service" | jq -r '.projectStatus.status')
          if [ "$STATUS" != "OK" ]; then echo "Quality gate failed!"; exit 1; fi

  # ════════════ Stage 3: Build & Push Container ════════════
  build-image:
    runs-on: ubuntu-latest
    needs: [build-and-test, code-quality]
    if: github.ref == 'refs/heads/main'
    permissions:
      contents: read
      packages: write
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Build Docker image
        run: docker build -t ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} .
      
      - name: Run Trivy security scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
          severity: 'CRITICAL,HIGH'
          exit-code: '1'  # Fail pipeline on critical vulnerabilities
      
      - name: Push to registry
        run: |
          echo ${{ secrets.GITHUB_TOKEN }} | docker login ${{ env.REGISTRY }} -u ${{ github.actor }} --password-stdin
          docker push ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
          docker tag ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
          docker push ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest

  # ════════════ Stage 4: Deploy to Staging ════════════
  deploy-staging:
    runs-on: ubuntu-latest
    needs: build-image
    environment: staging
    steps:
      - name: Deploy to K8s staging
        run: |
          kubectl set image deployment/order-service \
            order-service=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} \
            --namespace=staging
          kubectl rollout status deployment/order-service --namespace=staging --timeout=300s
      
      - name: Run smoke tests
        run: |
          STAGING_URL="https://staging.api.company.com"
          curl -sf "$STAGING_URL/actuator/health" | jq '.status' | grep -q '"UP"'
          curl -sf "$STAGING_URL/api/v1/orders?page=0&size=1" | jq '.content'

  # ════════════ Stage 5: Deploy to Production ════════════
  deploy-production:
    runs-on: ubuntu-latest
    needs: deploy-staging
    environment: production  # Requires manual approval in GitHub
    steps:
      - name: Deploy canary (10% traffic)
        run: |
          kubectl set image deployment/order-service-canary \
            order-service=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} \
            --namespace=production
          # Monitor error rate for 5 minutes
          sleep 300
      
      - name: Check canary health
        run: |
          ERROR_RATE=$(curl -s "http://prometheus:9090/api/v1/query?query=rate(http_server_requests_seconds_count{status=~'5..'}[5m])" | jq '.data.result[0].value[1]')
          if (( $(echo "$ERROR_RATE > 0.01" | bc -l) )); then
            echo "Canary error rate too high: $ERROR_RATE"
            kubectl rollout undo deployment/order-service-canary --namespace=production
            exit 1
          fi
      
      - name: Full rollout
        run: |
          kubectl set image deployment/order-service \
            order-service=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} \
            --namespace=production
          kubectl rollout status deployment/order-service --namespace=production --timeout=600s
```

**GitLab CI equivalent (key differences):**
```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - quality
  - package
  - deploy-staging
  - deploy-production

deploy-production:
  stage: deploy-production
  environment:
    name: production
  when: manual  # GitLab's manual approval
  only:
    - main
```

**⚠️ Pitfalls:**
- **Never deploy directly to production** without staging validation.
- **Pin ALL action/image versions** — `uses: actions/checkout@v4` not `@latest`. Supply chain attacks are real.
- **Store no secrets in code** — use GitHub Secrets, Vault, or AWS Secrets Manager.
- **Canary deployments need automatic rollback** — if error rate spikes, undo without human intervention.

---

### Q4. 🔴 🏢 What AWS services do you use for deploying microservices? Design a production architecture.

**A production Spring Boot microservices stack on AWS typically uses EKS (Kubernetes) or ECS (Fargate) for compute, RDS/Aurora for databases, ElastiCache for Redis, MSK for Kafka, CloudFront for CDN, and ALB for load balancing — with infrastructure as code via Terraform or CDK.**

**Production Architecture:**

```
                    Route 53 (DNS)
                        │
                    CloudFront (CDN)
                        │
                    ALB (Application Load Balancer)
                        │
            ┌───────────┼───────────────┐
            ▼           ▼               ▼
    ┌─────────────┐ ┌──────────┐ ┌────────────┐
    │ EKS Cluster │ │ EKS Node │ │ EKS Node   │
    │ ┌─────────┐ │ │ ┌──────┐ │ │ ┌────────┐ │
    │ │Order Svc│ │ │ │Pay   │ │ │ │Inv Svc │ │
    │ │(3 pods) │ │ │ │Svc   │ │ │ │(3 pods)│ │
    │ └─────────┘ │ │ └──────┘ │ │ └────────┘ │
    └─────────────┘ └──────────┘ └────────────┘
            │               │               │
    ┌───────▼───────────────▼───────────────▼───────┐
    │           AWS PrivateLink / VPC               │
    ├───────────────────────────────────────────────┤
    │  Aurora PostgreSQL    ElastiCache (Redis)     │
    │  (Multi-AZ, auto-    (Cluster mode,          │
    │   scaling replicas)   encryption at rest)     │
    ├───────────────────────────────────────────────┤
    │  Amazon MSK (Kafka)   S3 (file storage)      │
    │  (3 brokers,          SQS (simple queues)    │
    │   multi-AZ)           SNS (pub/sub)          │
    ├───────────────────────────────────────────────┤
    │  Secrets Manager      CloudWatch/X-Ray       │
    │  (DB creds, API keys) (Monitoring, tracing)  │
    └───────────────────────────────────────────────┘
```

**Key AWS Services for Microservices:**

| Service | Purpose | Alternative |
|---------|---------|-------------|
| **EKS** | Kubernetes orchestration | ECS Fargate (serverless containers) |
| **Aurora PostgreSQL** | Relational DB (auto-scaling, multi-AZ) | RDS PostgreSQL |
| **ElastiCache Redis** | Caching, session store | DAX (DynamoDB cache) |
| **MSK** | Kafka-managed service | SQS + SNS (simpler) |
| **ALB** | Layer 7 load balancing | NLB (Layer 4, gRPC) |
| **ECR** | Container registry | Docker Hub |
| **CloudWatch** | Logging, metrics | Datadog, New Relic |
| **Secrets Manager** | Secret rotation | SSM Parameter Store |
| **CloudFront** | CDN, edge caching | — |

```hcl
# Terraform example — EKS cluster
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 20.0"

  cluster_name    = "microservices-prod"
  cluster_version = "1.30"

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  eks_managed_node_groups = {
    general = {
      instance_types = ["m6i.xlarge"]
      min_size       = 3
      max_size       = 20
      desired_size   = 5
    }
  }
}

# Aurora PostgreSQL
module "aurora" {
  source  = "terraform-aws-modules/rds-aurora/aws"
  name    = "orders-db"
  engine  = "aurora-postgresql"
  engine_version = "16.1"
  
  instances = {
    writer  = { instance_class = "db.r6g.xlarge" }
    reader1 = { instance_class = "db.r6g.large" }
    reader2 = { instance_class = "db.r6g.large" }
  }
  
  storage_encrypted = true
  deletion_protection = true
}
```

**⚠️ Pitfalls:**
- **EKS vs. ECS Fargate** — EKS gives full Kubernetes control but requires K8s expertise. ECS Fargate is simpler for smaller teams.
- **Multi-AZ is NOT optional** for production databases — single-AZ means downtime during maintenance.
- **VPC endpoints** for S3, ECR, and other AWS services — avoid NAT Gateway costs for inter-service AWS API calls.
- **Cost** — always use Reserved Instances/Savings Plans for predictable workloads (60-70% savings).

---

### Q5. 🔴 🌐 Tell me about a project where you led the architecture or drove a major technical decision. What was the outcome?

**This is the most important behavioral question. Structure your answer using the STAR framework (Situation, Task, Action, Result) with specific metrics, showing technical depth AND leadership.**

**Example Answer (Adapt to your experience):**

> **Situation:** At my previous company, our monolithic order management system was experiencing 30+ second response times during peak traffic (Black Friday). The system handled 500 requests/second but needed to support 5000+ rps.
>
> **Task:** I was tasked with architecting the decomposition strategy and leading a team of 6 engineers to migrate critical services to microservices within 4 months.
>
> **Action:**
> 1. **Research & Decision:** I evaluated three approaches — full microservices, modular monolith, and a hybrid strangler fig. I presented a technical RFC to the architecture board recommending the strangler fig pattern, starting with the order and payment services.
> 2. **Architecture Design:** Designed the target architecture using Spring Boot 3, Kafka for event-driven communication, and Redis for caching. I introduced CQRS for the order service where read and write patterns differed significantly.
> 3. **Risk Mitigation:** Implemented a dual-write period where both the monolith and new services processed orders, with automated comparison of results. This caught 3 critical bugs before cutover.
> 4. **Team Leadership:** Ran weekly architecture reviews, paired with junior engineers on complex integrations, and created internal documentation that became the team's microservices playbook.
>
> **Result:**
> - Reduced P99 latency from 30s to 200ms.
> - Scaled to 8000 rps with auto-scaling during peak events.
> - Zero downtime during migration — no customer-facing incidents.
> - The architecture playbook was adopted by 3 other teams.

**Key Principles for Behavioral Answers:**
1. **Be specific** — mention technologies, team size, timeline, metrics.
2. **Show trade-offs** — "We chose Kafka over RabbitMQ because..."
3. **Acknowledge failures** — "We initially underestimated... and learned..."
4. **Quantify impact** — latency reduced by X%, cost saved by Y%.
5. **Show leadership** — mentoring, documentation, cross-team collaboration.

---

## 🟡 HIGH FREQUENCY (Questions 6–12)

---

### Q6. 🏢 How do you manage Kubernetes configurations for multiple environments (dev, staging, prod)?

**Use Kustomize (built into kubectl) or Helm charts to manage environment-specific K8s configurations — with a base set of manifests and per-environment overlays/values that modify replicas, resource limits, environment variables, and image tags.**

```
# Kustomize structure
k8s/
├── base/
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   └── configmap.yaml
├── overlays/
│   ├── dev/
│   │   ├── kustomization.yaml      # replicas: 1, limits: low
│   │   └── patch-deployment.yaml
│   ├── staging/
│   │   ├── kustomization.yaml      # replicas: 2, limits: medium
│   │   └── patch-deployment.yaml
│   └── prod/
│       ├── kustomization.yaml      # replicas: 3, limits: high, HPA
│       └── patch-deployment.yaml
```

```yaml
# k8s/overlays/prod/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ../../base
patches:
  - path: patch-deployment.yaml
images:
  - name: order-service
    newName: registry.company.com/order-service
    newTag: "1.4.2"
```

```bash
# Deploy
kubectl apply -k k8s/overlays/prod/
```

**⚠️ Pitfall:** Never use `kubectl apply -f` directly for production. Use Kustomize or Helm through GitOps (ArgoCD/Flux) for version-controlled, auditable deployments.

---

### Q7. 🏢 How do you implement Infrastructure as Code (IaC) with Terraform?

**Terraform defines cloud infrastructure declaratively in `.tf` files, applies changes incrementally via a plan-apply workflow, and tracks state to manage the full lifecycle of resources — it's the standard for multi-cloud IaC with modules for reusability and workspaces for environment isolation.**

```hcl
# main.tf — RDS instance for Spring Boot service
resource "aws_db_instance" "orders" {
  identifier           = "orders-${var.environment}"
  engine              = "postgres"
  engine_version      = "16.3"
  instance_class      = var.db_instance_class
  allocated_storage   = 100
  storage_encrypted   = true
  
  db_name             = "orders"
  username            = "orders_admin"
  password            = data.aws_secretsmanager_secret_version.db_password.secret_string
  
  multi_az            = var.environment == "prod"
  deletion_protection = var.environment == "prod"
  
  vpc_security_group_ids = [aws_security_group.db.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name
  
  backup_retention_period = var.environment == "prod" ? 30 : 7
  
  tags = {
    Environment = var.environment
    ManagedBy   = "terraform"
  }
}

# variables.tf
variable "environment" {
  type    = string
  default = "dev"
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}
```

**⚠️ Pitfalls:**
- **Store state remotely** (S3 + DynamoDB locking) — local state files cause conflicts in teams.
- **Never hardcode secrets** — use `data` sources to read from Secrets Manager.
- **`terraform plan` before every `apply`** — review what will change before executing.

---

### Q8. 🏢 How do you handle secrets management in a microservices environment?

**Secrets should never exist in code, configs, or container images — use a dedicated secrets manager (AWS Secrets Manager, HashiCorp Vault) with automatic rotation, and inject secrets at runtime via environment variables or mounted volumes.**

```yaml
# Kubernetes: External Secrets Operator + AWS Secrets Manager
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: order-db-credentials
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: ClusterSecretStore
  target:
    name: order-db-credentials
  data:
    - secretKey: password
      remoteRef:
        key: prod/order-service/db
        property: password
    - secretKey: username
      remoteRef:
        key: prod/order-service/db
        property: username
```

```java
// Spring Boot — read from environment (injected by K8s)
spring:
  datasource:
    url: jdbc:postgresql://${DB_HOST}:5432/orders
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}

// Or use Spring Cloud Vault for direct integration
spring:
  cloud:
    vault:
      uri: https://vault.company.com
      authentication: kubernetes
      kubernetes:
        role: order-service
```

**⚠️ Pitfalls:**
- **Rotate secrets regularly** — AWS Secrets Manager supports automatic rotation with Lambda.
- **Never log secrets** — mask sensitive values in log output.
- **Least privilege** — each service should only access its own secrets, not all secrets in the namespace.

---

### Q9. 🏢 How do you monitor microservices in production? What metrics and alerts do you set up?

**Production monitoring follows the RED method for services (Rate, Errors, Duration) and USE method for infrastructure (Utilization, Saturation, Errors) — with dashboards in Grafana, metrics in Prometheus, and alerts in PagerDuty for critical conditions.**

**RED Method (Services):**

| Metric | What | Alert Threshold |
|--------|------|----------------|
| **R**ate | Requests per second | Spike: >200% of baseline |
| **E**rrors | Error rate (5xx) | >1% of total requests |
| **D**uration | P99 latency | >500ms for APIs |

**Key Alerts to Configure:**

```yaml
# Prometheus alert rules
groups:
  - name: order-service-alerts
    rules:
      - alert: HighErrorRate
        expr: rate(http_server_requests_seconds_count{status=~"5.."}[5m]) 
              / rate(http_server_requests_seconds_count[5m]) > 0.01
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Order service error rate > 1%"
      
      - alert: HighLatency
        expr: histogram_quantile(0.99, rate(http_server_requests_seconds_bucket[5m])) > 0.5
        for: 5m
        labels:
          severity: warning
      
      - alert: PodRestarting
        expr: increase(kube_pod_container_status_restarts_total[1h]) > 3
        labels:
          severity: critical
      
      - alert: DatabaseConnectionPoolExhausted
        expr: hikaricp_connections_active / hikaricp_connections_max > 0.9
        for: 2m
        labels:
          severity: critical
```

**⚠️ Pitfall:** Alert on symptoms (high latency, errors), not causes (high CPU). CPU can be high during legitimate load. Set alerts with `for` duration to avoid flapping.

---

### Q10. 🌐 Describe a conflict you had with a team member about a technical decision. How did you resolve it?

**Use the STAR framework. Show emotional intelligence, data-driven decision-making, and prioritization of team outcomes over personal ego.**

**Example Answer:**

> **Situation:** A senior colleague strongly advocated for using MongoDB for our new product catalog service, while I believed PostgreSQL with JSONB columns was the better choice given our team's expertise and query patterns.
>
> **Task:** Reach a technical consensus without damaging the working relationship.
>
> **Action:**
> 1. **Data first** — I created a comparison document with benchmarks: write throughput, query patterns, operational costs, and team expertise matrix.
> 2. **Listened actively** — invited them to present their perspective first. They had valid points about flexible schemas and horizontal scaling.
> 3. **Identified shared ground** — we both agreed that schema flexibility was important. I proposed PostgreSQL with JSONB as a middle ground — relational + flexible queries.
> 4. **Prototype** — built two small prototypes over a weekend. The PostgreSQL JSONB version actually performed better for our specific read-heavy, join-heavy queries.
> 5. **Let data decide** — presented both prototypes to the team. The data made the decision clear.
>
> **Result:** Team unanimously chose PostgreSQL. My colleague and I actually became closer — they appreciated the structured approach. The catalog service has been running for 2 years with zero data model issues.

**Key Behavioral Principles:**
- **Never make it personal** — "I think Option A is better because..." not "Your idea won't work."
- **Propose experiments** — prototypes and benchmarks defuse arguments.
- **Give credit** — "Maria's concern about schema flexibility led us to the JSONB approach."
- **Disagree and commit** — if overruled, support the decision fully.

---

### Q11. 🏢 How do you handle zero-downtime deployments?

**Zero-downtime deployments are achieved through rolling updates (K8s default), blue-green (two parallel environments), or canary (gradual traffic shift) — combined with graceful shutdown, readiness probes, and database backward compatibility.**

**Requirements for Zero-Downtime:**
1. **Rolling update or blue-green** — old and new versions run simultaneously during transition.
2. **Graceful shutdown** — drain in-flight requests before terminating.
3. **Backward-compatible DB migrations** — old code must work with new schema.
4. **Readiness probes** — new pods only receive traffic when fully initialized.
5. **Health checks** — automatic rollback if new version is unhealthy.

```yaml
# Spring Boot graceful shutdown
server:
  shutdown: graceful
spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s

# K8s: preStop hook to allow LB drain time
lifecycle:
  preStop:
    exec:
      command: ["sh", "-c", "sleep 10"]  # Wait for LB to deregister
terminationGracePeriodSeconds: 45
```

**⚠️ Pitfall:** Database schema changes must be backward compatible. Column renames must use a 3-step process: add new column → migrate data → deploy new code → drop old column.

---

### Q12. 🏢 How do you implement GitOps for Kubernetes deployments?

**GitOps uses Git as the single source of truth for infrastructure and application state — a GitOps operator (ArgoCD, Flux) continuously reconciles the desired state in Git with the actual state in Kubernetes, enabling declarative, auditable, and version-controlled deployments.**

```yaml
# ArgoCD Application
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: order-service
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/company/k8s-manifests
    targetRevision: main
    path: services/order-service/overlays/prod
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true        # Delete resources removed from Git
      selfHeal: true      # Revert manual kubectl changes
    syncOptions:
      - CreateNamespace=true
```

**GitOps Flow:**
```
Developer → PR to app repo → CI builds image → Updates image tag in config repo 
→ ArgoCD detects change → Syncs to K8s → Automated tests → Rollback if failed
```

**⚠️ Pitfall:** Separate application code repos from configuration repos — CI updates the config repo after building the image, keeping concerns separated.

---

## 🟢 GOOD TO KNOW (Questions 13–20)

---

### Q13. 🏬 What is the difference between Docker Compose and Kubernetes?

**Docker Compose orchestrates multi-container applications on a single host for local development; Kubernetes orchestrates containers across a cluster of nodes for production — Compose for dev/test, Kubernetes for production.**

| Feature | Docker Compose | Kubernetes |
|---------|---------------|-----------|
| Scale | Single host | Multi-node cluster |
| Self-healing | No | Yes (restarts, replaces pods) |
| Auto-scaling | No | HPA, VPA |
| Service discovery | DNS within network | DNS + Services |
| Use case | Local development | Production |

---

### Q14. 🏬 How do you handle database migrations in CI/CD pipelines?

**Database migrations should be decoupled from application deployment — run migrations as a separate CI/CD step (or Kubernetes Job) before deploying the new application version, ensuring the database schema is always forward-compatible with both old and new code.**

```yaml
# K8s Job for Flyway migration (runs before deployment)
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migrate-{{ .Release.Revision }}
  annotations:
    helm.sh/hook: pre-upgrade    # Runs before Helm upgrade
    helm.sh/hook-weight: "-5"
spec:
  template:
    spec:
      containers:
        - name: flyway
          image: flyway/flyway:10
          args: ["migrate"]
          env:
            - name: FLYWAY_URL
              value: "jdbc:postgresql://db:5432/orders"
            - name: FLYWAY_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-credentials
                  key: password
      restartPolicy: Never
  backoffLimit: 3
```

---

### Q15. 🏬 What is container orchestration and why do we need it?

**Container orchestration automates the deployment, scaling, networking, and lifecycle management of containers across a cluster — solving the operational challenges of running hundreds of containers in production: scheduling, self-healing, load balancing, and rolling updates.**

**Problems Solved:**
- **Scheduling** — which node runs which container (resource-based placement).
- **Self-healing** — restart crashed containers, replace unhealthy nodes.
- **Scaling** — add/remove container instances based on load.
- **Networking** — service discovery, load balancing, ingress.
- **Storage** — persistent volumes, dynamic provisioning.

---

### Q16. 🏬 How do you implement log aggregation for microservices?

**Centralized log aggregation collects logs from all microservices into a single searchable platform — using the EFK stack (Elasticsearch + Fluentd/Fluent Bit + Kibana) or Grafana Loki with structured JSON logging and correlation IDs.**

```java
// Structured JSON logging (Spring Boot + Logback)
// logback-spring.xml
<configuration>
    <appender name="JSON" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
            <includeMdcKeyName>traceId</includeMdcKeyName>
            <includeMdcKeyName>spanId</includeMdcKeyName>
        </encoder>
    </appender>
</configuration>

// Output:
// {"timestamp":"2025-07-15T10:30:00Z","level":"INFO","logger":"OrderService",
//  "message":"Order created","traceId":"abc123","orderId":"456","service":"order-service"}
```

**⚠️ Pitfall:** Log to `stdout` in containers — let the platform (K8s + Fluent Bit) collect and forward. Don't write to files inside containers.

---

### Q17. 🏢 How do you ensure security in a CI/CD pipeline?

**Pipeline security includes dependency scanning (OWASP, Snyk), container image scanning (Trivy), SAST (SonarQube, CodeQL), secret scanning (git-secrets, TruffleHog), signed artifacts, and least-privilege access to deployment environments.**

**Security Gates:**
```
Code Commit → SAST (SonarQube) → Dependency Scan (Snyk) 
→ Build → Container Scan (Trivy) → Sign Image (Cosign)
→ Deploy Staging → DAST (OWASP ZAP) → Deploy Production
```

**⚠️ Pitfall:** Fail the pipeline on CRITICAL vulnerabilities. HIGH vulnerabilities should be tracked with a deadline but not block deployment to avoid alert fatigue.

---

### Q18. 🌐 Tell me about a time you had to make a decision with incomplete information.

**Example Answer:**

> **Situation:** Our payment service was experiencing intermittent failures (0.5% error rate) in production during the holiday season. We had to decide between an immediate hot-fix (risky) or a full investigation (slow) with customers affected.
>
> **Action:** I chose a middle path:
> 1. Immediately enabled the circuit breaker fallback to queue failed payments for retry.
> 2. Added targeted logging to capture the failure pattern.
> 3. Within 2 hours, identified a connection pool exhaustion under load.
> 4. Applied a targeted fix (increased pool size + added connection timeout) and deployed via fast-track pipeline.
>
> **Result:** Error rate dropped to 0.01% within 30 minutes of the fix. Total customer impact was limited to 4 hours with zero lost transactions (all retried successfully).

---

### Q19. 🏢 How do you approach performance testing for microservices?

**Performance testing for microservices includes load testing (expected traffic), stress testing (beyond capacity), soak testing (sustained load for memory leaks), and chaos engineering — using tools like Gatling, k6, or JMeter with realistic data and production-like infrastructure.**

```javascript
// k6 load test script
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '2m', target: 100 },   // Ramp up to 100 users
    { duration: '5m', target: 100 },   // Stay at 100 users
    { duration: '2m', target: 500 },   // Spike to 500 users
    { duration: '5m', target: 500 },   // Sustain spike
    { duration: '2m', target: 0 },     // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(99)<500'],  // 99% of requests under 500ms
    http_req_failed: ['rate<0.01'],    // Less than 1% failure rate
  },
};

export default function () {
  const res = http.get('https://staging.api.company.com/api/v1/orders');
  check(res, { 'status is 200': (r) => r.status === 200 });
  sleep(1);
}
```

**⚠️ Pitfall:** Always test against a staging environment that mirrors production — testing against dev gives misleading results.

---

### Q20. 🌐 How do you handle technical debt in a fast-moving team?

**Technical debt is managed through explicit tracking, continuous small repayments, and strategic investment — the key is making debt visible (logged as tickets), allocating 15-20% of sprint capacity to debt reduction, and prioritizing debt that blocks future features or causes operational incidents.**

**Framework for Prioritizing Tech Debt:**

| Priority | Criteria | Example | Action |
|----------|---------|---------|--------|
| **P0** | Causing production incidents | Memory leaks, deadlocks | Fix immediately |
| **P1** | Blocking feature development | Tightly coupled modules | Next sprint |
| **P2** | Slowing development velocity | Missing tests, poor API docs | Allocate 20% capacity |
| **P3** | Code smell, style issues | Naming, minor refactors | Boy Scout Rule |

**Strategies:**
1. **Boy Scout Rule** — leave code cleaner than you found it (small, opportunistic fixes).
2. **Tech Debt Sprints** — dedicate 1 sprint per quarter to pure debt reduction.
3. **Definition of Done** — include test coverage and documentation as non-negotiable.
4. **ADRs (Architecture Decision Records)** — document WHY shortcuts were taken so future teams understand context.

> **In interviews, emphasize:** "I believe tech debt is a business decision, not a technical one. I communicate the business impact — 'This legacy service adds 2 days to every feature that touches payments' — to get buy-in for investment."
