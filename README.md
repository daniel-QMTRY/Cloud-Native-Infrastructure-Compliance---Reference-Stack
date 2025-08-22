# Cloud-Native Infrastructure & Compliance — Reference Stack

**Goal:** a practical, demo-ready stack that shows **cloud-native** patterns and **compliance evidence** end-to-end: IaC → policy → scans → artifacts → dashboard.

* **Platform:** Ubuntu Server Pro VM (hypervisor host) or any Linux VM  
* **K8s:** containerd • **Cilium** (CNI) • **ingress-nginx** • **Kata Containers** (optional isolation)  
* **Build:** **Podman** / Buildah (no Docker daemon)  
* **Data:** **Couchbase** (local) or **Couchbase Capella** (managed)  
* **App:** Streamlit “Evidence Dashboard”  
* **Policy/Scans:** OPA/Conftest • Checkov • Trivy/Grype • Bandit • Gitleaks • Syft (SBOM)  
* **Observability:** Loki + Grafana (optional)  
* **LLM (optional):** Amazon **Bedrock** / Google **Gemini** → human-readable findings  
* **Datasets:** synthetic (Synthea / Data4Citizen)

> This repo is designed to run fully **local** (zero cloud bill) and optionally on **AWS/K8s** later.

---

## Table of Contents

- [Architecture](#architecture)
- [Repo Layout](#repo-layout)
- [Quick Start (Local)](#quick-start-local)
- [GitHub Actions Workflows](#github-actions-workflows)
- [Kubernetes (Optional)](#kubernetes-optional)
- [Couchbase & Capella](#couchbase--capella)
- [LLM Summaries (Bedrock/Gemini – Optional)](#llm-summaries-bedrockgemini--optional)
- [Compliance Evidence](#compliance-evidence)
- [Make Targets](#make-targets)
- [Secrets](#secrets)
- [Notes & Credits](#notes--credits)

---

## Architecture

```mermaid
flowchart LR
  Dev[Podman/Buildah] --> Img[Container Image]
  IaC[Terraform] -->|plan/apply| Cloud[(LocalStack/AWS)]
  Policy[OPA/Conftest\nCheckov] --> Evidence[(evidence/*.json)]
  SecScan[Trivy/Grype\nBandit/Gitleaks\nSyft SBOM] --> Evidence
  Couch[Couchbase / Capella] --> App[Streamlit Evidence Dashboard]
  Evidence --> App
  App -->|Ingress| K8s[(Kubernetes\ncontainerd + Cilium + ingress-nginx)]
  K8s --> Logs[(Loki/Grafana)]
  Kata[(Kata Containers)] --- K8s
