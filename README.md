# CKAD_Study

This repository is a curated collection of notes and YAML examples for the **Certified Kubernetes Application Developer (CKAD)** exam. It is organized by key exam domains and is designed to help reinforce both conceptual understanding and hands-on practice.

## 📁 Directory Structure

```
CKAD_Study/
├── Course_Notes/
│   ├── 2. CoreConcepts/
│   │   ├── Compute-quota.yaml
│   │   ├── namespace-dev.yaml
│   │   ├── pod_definition_example.yaml
│   │   ├── rc-definition.yaml
│   │   └── replicaset-definition.yaml
│   ├── 3. Configuration/
│   │   ├── config-map.yaml
│   │   ├── pod-definition-with-config.yaml
│   │   ├── secret-data.yaml
│   │   ├── webapp-config-map.yaml
│   │   └── cheat-sheet.md
├── .gitignore
├── LICENSE
└── README.md
```

## What's Included

### ✅ 2. Core Concepts
Examples and resources focused on fundamental Kubernetes building blocks:
- Pods
- Namespaces
- Replication controllers
- ReplicaSets
- Resource quotas

### ✅ 3. Configuration
YAML templates and notes on managing application configuration using:
- ConfigMaps
- Secrets
- Environment variables

Includes a `cheat-sheet.md` for rapid review and CLI reference.

## CKAD Exam Focus

This repo is intended to help you practice:
- Fast, CLI-driven YAML creation and modification
- Core Kubernetes object configuration
- Real-world usage of exam topics

## How to Use

- Clone the repo:  
  ```bash
  git clone https://github.com/<your-username>/CKAD_Study.git
  ```
- Apply files using `kubectl apply -f <file.yaml>`
- Use the `cheat-sheet.md` for quick lookup during practice

## License

This project is licensed under the terms of the MIT License.

