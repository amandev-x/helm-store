
# Module 1 What is Helm and why does it exists 🙄

## This helm module will be based on hands on project, so you can get familiar with why helm exists and what does it do for us?

## Let's start with the **why** helm exists - because if we understand the pain **helm** solves everything will make sense.

### 🤔 The Problem - Life without **Helm**

#### If your are learning helm then you must be familiar with kubernetes. We know that to deploy an application in kubernetes we have to write yaml files for every kubernetes resources and manage them manually when we make any changes in our application.

#### Now let's say we have our microservice based project which is **HelStore**. Now in this project we have to deploy an **product-service** for the **HelmStore**. So we would need to have yaml files like this:

```bash
product-service/
├── deployment.yaml
├── service.yaml
├── configmap.yaml
├── secret.yaml
├── ingress.yaml
└── hpa.yaml          ← horizontal pod autoscaler
```

#### That's 6 files for one service. Our **HelmStore** project has 3 services. That's already **18+ YAML files**. Now just imagine that:

- You need a **dev environment** and **prod environment**
- In dev: 1 replica, debug logs and no resource limits
- In prod: 3 replicas, info logs and strict resource limits

#### Now without knowing helm one will write manifests files for one environment and copy those files with significant changes on other environment. Now if you forgot to change an value then your **Production breaks**😵😵.

#### This is the real pain for what **Helm** was built to solve. 

### 💡 What Helm Actually Is 

#### **Helm is a package manager for Kubernetes** — like we have ```apt``` for Ubuntu or ```pip``` for python, but for k8s apps.

#### But it's more than just a package manager. Helm does 3 things

**1. 📦 Packaging**

#### Bundles all your K8s YAML files into a single unit called a **Chart**. One chart = one complete application.

**2. 🔧 Templating**

#### Instead of hardcoding values in YAML, you use variables. One chart works for dev AND prod — just pass different values.

```bash
# Without Helm (hardcoded values - bad)
replicas: 3
image: myapp:v1.0.0

# With Helm (templated values - good)
replicas: {{ .Values.replicas }}
image: {{ .Values.image }}
```

**3. 🚀 Release Management**

### Helm tracks every deployment like Git tracks code. It knows:

- What version is currently running
- What changed between versions
- How to **rollback** to a previous version in one command

## 🏗️ Helm's 4 Core Concepts

### These 4 words will come up in **every interview**. Learn them cold:

#### 1. Chart 📦

A Chart is a **collection of files** that describe a K8s application. Think of it like a recipe — it contains all instructions to deploy your app but no actual running instance yet.

``` bash
mychart/
├── Chart.yaml        ← metadata (name, version, description)
├── values.yaml       ← default configuration values
└── templates/        ← your K8s YAML templates
    ├── deployment.yaml
    └── service.yaml
```

#### 2. Repository 🗄️

A place where Charts are stored and shared — like Docker Hub but for Helm charts. The most popular is **Artifact Hub** (artifacthub.io). Companies also host private repos.

#### 3. Release 🚀

When you actually install a chart into your cluster, that running instance is called a Release. You can install the same chart multiple times with different names — each is a separate release.

```bash
# Installing same PostgreSQL chart twice = 2 releases
helm install product-db bitnami/postgresql # release: product-db
helm install cart-db bitnami/postgresql # release: cart-db

Both run in the same cluster, but completely independent.
```

#### 4. Values ⚙️ 

The configuration that customizes a chart for your specific use case. Every chart has default values you can override — replicas, image tags, resource limits, passwords, etc.

## 🔄 How Helm Works — The Full Flow

```bash
You run:
helm install product-service ./charts/product-service --values values.dev.yml

         │
         ▼
┌─────────────────────┐
│   Helm Client       │  ← your CLI (helm binary)
│   reads chart +     │
│   merges values     │
└────────┬────────────┘
         │  renders templates into final YAML
         ▼
┌─────────────────────┐
│   Kubernetes API    │  ← K8s receives the final rendered YAML
│   Server            │
└────────┬────────────┘
         │  creates resources
         ▼
┌─────────────────────┐
│  Pods, Services,    │  ← your app is now running!
│  ConfigMaps etc.    │
└─────────────────────┘

Helm stores release info as a Secret in application namespace
(so it remembers what was deployed)
```

---

## 🆚 Helm vs Raw YAML — Side by Side

| Problem | Raw YAML | With Helm |
|--------|----------|-----------|
| Deploy to dev & prod | Duplicate all files | One chart, two value files |
| Update image tag | Edit every deployment.yaml | Change one value |
| Rollback a bad deploy | Manually re-apply old YAML | `helm rollback myapp 1` |
| Share your app setup | Send 20+ files | `helm install myrepo/myapp` |
| Track what's deployed | Check each resource manually | `helm list` |
| Delete everything cleanly | `kubectl delete` each resource | `helm uninstall myapp` |

---

## 🏪 HelmStore Context — Why We Need Helm

**Without Helm, deploying HelmStore means managing:**

``` bash
Raw YAML chaos:
├── product-service/
│   ├── deployment.yaml       ← hardcoded image, replicas
│   ├── service.yaml
│   ├── configmap.yaml        ← hardcoded DB URL
│   ├── secret.yaml           ← hardcoded passwords 💀
│   └── ingress.yaml
├── cart-service/
│   ├── deployment.yaml       ← copy-pasted from above
│   ├── service.yaml
│   ├── configmap.yaml
│   └── secret.yaml
├── frontend/
│   ├── deployment.yaml
│   └── service.yaml
└── dev/                      ← same files again for dev!
    ├── product-service/...
    ├── cart-service/...
    └── frontend/...
```

**With Helm (what we'll build):**

```bash
Clean Helm structure:
├── charts/
│   ├── product-service/      ← one chart, works everywhere
│   ├── cart-service/
│   └── frontend/
└── values/
    ├── dev.yaml              ← just the differences
    └── prod.yaml             ← just the differences
```

## ⚙️ Environment Setup — Let's Get Your Hands Dirty

Let's make sure your environment is ready. Run these one by one:

**Step 1: Install Helm** 

```bash
# On Linux
sudo apt-get install curl gpg apt-transport-https --yes
curl -fsSL https://packages.buildkite.com/helm-linux/helm-debian/gpgkey | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/helm.gpg] https://packages.buildkite.com/helm-linux/helm-debian/any/ any main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

**Step 2: Create kind cluster**

``` bash
# I'm using kind cluster you can use whatever you are familiar with.
kind create cluster --name helm-learning --config helm-cluster.yml

# Verify cluster is running
kubectl get nodes
```

**Step 3: Your first Helm command**

``` bash
# See all helm commands
helm --help 

# List what's currently deployed
helm list or helm list -a

# Expected: NAME  NAMESPACE  REVISION  ...  STATUS  CHART  APP VERSION
```

**Step 4: Add your first chart repository**

```bash
# Add bitnami repo
helm repo add bitnami https://charts.bitnami.com/bitnami

# Update repo index (like apt update)
helm repo update 

# Verify if the repo was added
helm repo list
# Expected:  NAME     URL
             bitnami  https://charts.bitnami.com/bitnami
```

**Step 5: Search for the chart**

```bash
# Search for postgresql chart
helm search repo postgresql

# Search for postgresql from hub
helm search hub postgresql
```

**🎉 You just interacted with Helm for the first time!**

## Module 1 Complete! 🎉

## Answer these questions:

### Q1. What is the difference between a **Chart** and a **Release**?

**Ans:** A Chart is the template/package (like a Docker image). A Release is a running instance of that chart installed in the cluster (like a running container). You can have multiple releases from one chart.

### Q2. You need to deploy the same app in dev (1 replica) and prod (5 replicas). How does Helm help you avoid duplicating YAML files?

**Ans:** You write **one chart** with templated values (```{{ .Values.replicaCount }}```), then create two value files — ```dev.yaml``` (replicaCount: 1) and ```prod.yaml``` (replicaCount: 5). Helm merges them at install time.

### Q3. Where does Helm store information about what's currently deployed in your cluster?

**Ans:** Helm stores release information as **Kubernetes Secrets** in the namespace where the release was installed. That's how it tracks history, versions, and enables rollbacks.

### Q4. What is **Helm** and why would you use it over plain **kubectl apply**?

**Ans:** Helm is Kubernetes' package manager. Instead of managing dozens of raw YAML files per environment, Helm lets you templatize your K8s manifests into reusable charts, override values per environment, and track deployments as versioned releases — which means you get one-command rollbacks. For a multi-service app like a microservices platform, it reduces YAML duplication massively.

### Q5. What's the difference between **helm install** and **helm upgrade**?

**Ans:** ```helm install``` creates a brand new release. ```helm upgrade``` updates an existing release to a new chart version or new values. If the release doesn't exist, you can combine both with ```helm upgrade --install``` which does either depending on whether it exists — this is what CI/CD pipelines usually use.











