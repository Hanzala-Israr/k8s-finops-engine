# 🚀 Automated GitOps & Infrastructure Cost-Optimization Engine

> **A multi-tenant Kubernetes framework leveraging ArgoCD, native Kubernetes Resource Governance, and the CNCF Kube-green Operator to enforce infrastructure budget guardrails and reduce non-production compute costs by 40–50%.**

<p align="center">
  <video src="https://github.com/user-attachments/assets/154ec8d1-5133-4d20-9eda-38ccb536aba9" width="100%" controls autoplay loop muted></video>
</p>

---

# 📖 Overview

Cloud environments often run development and testing workloads continuously, even when no engineers are actively using them. These idle workloads consume unnecessary compute resources, increasing operational costs without delivering business value.

This project demonstrates a **GitOps-driven FinOps platform** that automatically optimizes Kubernetes infrastructure by combining:

* **ArgoCD** for declarative GitOps deployment
* **Kubernetes Resource Governance** using ResourceQuotas and LimitRanges
* **CNCF Kube-green** for automated workload scheduling

Development environments are automatically suspended outside working hours, while production workloads continue operating uninterrupted.

---

# ✨ Features

* Multi-tenant Kubernetes architecture
* GitOps deployment with ArgoCD
* Automated workload scheduling
* Kubernetes ResourceQuotas
* LimitRanges for default resource allocation
* Automatic scaling to zero during off-hours
* Production environment isolation
* Infrastructure cost optimization
* Declarative Infrastructure-as-Code
* Native Kubernetes implementation

---

# 🛠 Technology Stack

| Category               | Technology                 |
| ---------------------- | -------------------------- |
| Container Platform     | Kubernetes                 |
| Local Cluster          | KinD / Minikube            |
| GitOps                 | ArgoCD                     |
| FinOps Scheduler       | CNCF Kube-green            |
| Certificate Management | Cert-Manager               |
| Resource Governance    | ResourceQuota & LimitRange |
| Configuration          | YAML                       |

---

# 🏗 Architecture

```
                        GitHub Repository
                               │
                               ▼
                          ArgoCD GitOps
                               │
             ┌─────────────────┴─────────────────┐
             │                                   │
             ▼                                   ▼
     team-alpha-dev                     team-beta-prod
      (Development)                      (Production)

 ResourceQuota + LimitRange           No Scheduled Sleep

      Kube-green
  Night/Weekend Sleep
             │
             ▼
      Scale Deployments
         Down to Zero
```

<!-- <img width="1600" height="863" alt="argo-khan" src="https://github.com/user-attachments/assets/d7a596f2-bb5b-40a3-9f54-298d3ba36e62" /> -->
<img width="1600" height="941" alt="WhatsApp Image 2026-07-01 at 10 42 04 AM" src="https://github.com/user-attachments/assets/311e354d-fe94-4a24-b4e7-74cfeb63e5db" />

---

# 📂 Repository Structure

```text
k8s-finops-engine/
│
├── argocd-applications.yaml
│
└── tenants/
    ├── team-alpha-dev/
    │   ├── namespace.yaml
    │   ├── quota.yaml
    │   ├── limit-range.yaml
    │   ├── sleep-schedule.yaml
    │   └── deployment.yaml
    │
    └── team-beta-prod/
        ├── namespace.yaml
        └── deployment.yaml
```

---

# 🏢 Multi-Tenant Design

## Development Namespace (`team-alpha-dev`)

Designed for non-production workloads.

Features:

* ResourceQuota enforcement
* Default container sizing using LimitRanges
* Automatic sleep schedule
* GitOps managed deployments
* Budget guardrails

---

## Production Namespace (`team-beta-prod`)

Designed for mission-critical workloads.

Features:

* Always running
* No scheduled shutdown
* GitOps managed deployments
* Production isolation

---

# ⚙ Resource Governance

Development namespaces enforce native Kubernetes resource policies.

### ResourceQuota

Limits:

* CPU usage
* Memory usage
* Number of Pods
* Number of Services

### LimitRange

Provides default:

* CPU Requests
* CPU Limits
* Memory Requests
* Memory Limits

This prevents oversized containers from being deployed accidentally.

---

# 🌙 Automated Off-Hours Scheduling

The **CNCF Kube-green Operator** continuously monitors configured schedules.

During:

* Nights
* Weekends
* Holidays (configurable)

it automatically scales development workloads to:

```text
replicas = 0
```

Production deployments remain unaffected.

---

# 💰 FinOps Cost Optimization

There are **168 hours** in a calendar week.

Typical development environments are only used for approximately:

```text
8 hours/day
5 days/week

= 40 hours
```

Idle time:

```text
168 - 40 = 128 hours
```

Infrastructure idle percentage:

```text
128 / 168 × 100
≈ 76.2%
```

Although nodes cannot always be reduced by the full workload percentage, Kubernetes Cluster Autoscaler or Karpenter can reclaim a significant portion of unused compute capacity.

**Estimated infrastructure savings:**

* **40–50% reduction in non-production compute costs**

---

# 🔄 GitOps Synchronization Challenge

## The Problem

Kube-green modifies live Deployments by changing:

```yaml
spec:
  replicas: 0
```

ArgoCD compares the live cluster against Git.

Git contains:

```yaml
replicas: 2
```

ArgoCD therefore detects drift and immediately restores the deployment.

Result:

```
Kube-green
      ↓
Scale to 0

      ↓

ArgoCD
      ↓
Scale back to 2

      ↓

Infinite reconciliation loop
```

---

## The Solution

ArgoCD is configured to ignore replica count differences.

```yaml
ignoreDifferences:
  - group: apps
    kind: Deployment
    jsonPointers:
      - /spec/replicas
```

This allows:

* ArgoCD to continue managing application configuration
* Kube-green to control runtime replica counts

without conflicting with each other.

---

# 🔍 Verification

Monitor both namespaces simultaneously:

```bash
watch -n 1 "
echo '=== SYSTEM UTC TIME ==='
date -u

echo
echo '=== TEAM ALPHA (DEV) ==='
kubectl get pods -n team-alpha-dev

echo
echo '=== TEAM BETA (PROD) ==='
kubectl get pods -n team-beta-prod
"
```

---

# 🛠 Troubleshooting

## View Kube-green Logs

```bash
kubectl logs -n kube-green \
-l control-plane=controller-manager \
--tail=50
```

---

## Reset Scheduler State

If scheduling has already executed and you need to force a new evaluation:

```bash
kubectl delete secret sleepinfo-off-hours-savings \
-n team-alpha-dev
```

---

👨‍💻 Author

Hanzala Israr


