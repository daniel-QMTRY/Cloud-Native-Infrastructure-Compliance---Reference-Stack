# Cloud-Native Infrastructure & Compliance — Reference Stack

![Built with Podman](https://img.shields.io/badge/built%20with-Podman-6b1d9a)
![Cilium CNI](https://img.shields.io/badge/CNI-Cilium-3b82f6)
![Ingress NGINX](https://img.shields.io/badge/Ingress-nginx-10b981)
![SBOM Syft](https://img.shields.io/badge/SBOM-Syft-64748b)
![Scans Trivy/Grype](https://img.shields.io/badge/Scans-Trivy%2FGrype-ef4444)
![Policy OPA/Checkov](https://img.shields.io/badge/Policy-OPA%2FCheckov-f59e0b)
![Dashboard Streamlit](https://img.shields.io/badge/Dashboard-Streamlit-f43f5e)

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

%% ===== Build & CI =====
subgraph CI[Build & CI]
  Dev[Podman/Buildah] --> Img[Container Image]
  CIpipe[GitHub Actions CI]
  Policy[OPA/Conftest & Checkov] --> Evidence[(evidence/*.json)]
  SecScan[Trivy/Grype\nBandit/Gitleaks\nSyft SBOM] --> Evidence
  Img -. "scan image" .-> SecScan
  CIpipe --> Policy
  CIpipe --> SecScan
end

%% ===== Infrastructure =====
subgraph Infra[Infrastructure]
  IaC[Terraform] -->|plan/apply| Cloud[(LocalStack/AWS)]
end

%% ===== Data =====
subgraph Data
  Couch[(Couchbase / Capella)]
end

%% ===== App Layer =====
subgraph App[App Layer]
  AppUI[Streamlit Evidence Dashboard]
end

%% ===== Flows =====
Evidence --> AppUI
Couch --> AppUI
AppUI -->|Ingress| K8s[(Kubernetes\ncontainerd + Cilium + ingress-nginx)]
Kata[Kata Containers] --- K8s
K8s --> Logs[(Loki/Grafana)]

%% ===== Readable cylinder styles (dark/light friendly) =====
style Evidence fill:#1f2937,stroke:#93c5fd,stroke-width:1.6px,color:#eef2ff
style Cloud    fill:#1f2937,stroke:#93c5fd,stroke-width:1.6px,color:#eef2ff
style Couch    fill:#1f2937,stroke:#93c5fd,stroke-width:1.6px,color:#eef2ff
style K8s      fill:#1f2937,stroke:#93c5fd,stroke-width:1.6px,color:#eef2ff
style Logs     fill:#1f2937,stroke:#93c5fd,stroke-width:1.6px,color:#eef2ff

