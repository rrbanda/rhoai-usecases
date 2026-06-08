# RHOAI Use Cases

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![RHOAI](https://img.shields.io/badge/RHOAI-3.4-red)](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.4)

AI use case deployments for **Red Hat OpenShift AI 3.4** -- models, services, and training workloads managed via ArgoCD (GitOps).

This repo is the **use-case companion** to [rhoai-deploy-gitops](https://github.com/rrbanda/rhoai-deploy-gitops), which handles the platform deployment (operators, instances, DSC). This repo handles what runs **on top of** the platform.

## Prerequisites

- RHOAI 3.4 platform deployed via [rhoai-deploy-gitops](https://github.com/rrbanda/rhoai-deploy-gitops)
- ArgoCD (OpenShift GitOps) running with cluster-admin
- GPU nodes available for model serving

## Structure

```
rhoai-usecases/
├── argocd/                          # ArgoCD ApplicationSets and project
│   ├── apps/
│   │   ├── cluster-models-appset.yaml
│   │   ├── cluster-services-appset.yaml
│   │   └── training-workloads-app.yaml
│   └── projects/
│       └── usecases-project.yaml
└── usecases/
    ├── models/                      # Model serving (per-model GitOps)
    │   ├── gpt-oss-120b/
    │   ├── orchestrator-8b/
    │   └── qwen-math-7b/
    └── services/                    # Supporting services
        ├── genai-toolbox/
        ├── llamastack/
        ├── rhokp/
        └── toolorchestra-app/
```

## Deployment

### Option A: GitOps (ArgoCD)

```bash
oc apply -k argocd/apps/
```

ArgoCD auto-discovers models and services via Git directory generators. To deploy a model or service, remove its `exclude` entry from the relevant ApplicationSet and push.

### Option B: Manual

```bash
oc apply -k usecases/models/gpt-oss-120b/profiles/tier1-minimal/
oc apply -k usecases/services/llamastack/profiles/tier1-minimal/
oc apply -k usecases/services/genai-toolbox/profiles/tier1-minimal/
oc apply -k usecases/services/rhokp/profiles/tier1-minimal/
```

## Models and Services

| Category | Name | Default | Description |
|----------|------|:---:|-------------|
| Model | **gpt-oss-120b** | Excluded | OpenAI GPT-OSS 120B MoE (MXFP4, 4x L40S tensor-parallel) |
| Model | **orchestrator-8b** | Excluded | NVIDIA Nemotron-Orchestrator-8B for multi-tool coordination |
| Model | **qwen-math-7b** | Excluded | Qwen2.5-Math-7B-Instruct math specialist |
| Service | **llamastack** | Yes | Meta LlamaStack Distribution with agents, RAG, and tool use |
| Service | **genai-toolbox** | Yes | MCP Toolbox for Databases (PostgreSQL) |
| Service | **rhokp** | Yes | Red Hat OKP MCP Server for RHEL docs, CVEs, errata |
| Service | **toolorchestra-app** | Excluded | ToolOrchestra UI for multi-model orchestration |

## Secrets

Secret YAML files use `CHANGE_ME` / `fake` placeholder values. After deployment, patch with real values:

```bash
oc patch secret <name> -n <namespace> -p '{"stringData":{"key":"real-value"}}'
```

Pre-commit hooks (gitleaks) prevent real credentials from being committed. See `.pre-commit-config.yaml`.

## Setup

```bash
pip install pre-commit
pre-commit install
git config core.hooksPath .githooks
```
