---
sidebar_position: 4
title: "Sécurité dans la CI (DevSecOps)"
description: "Intégration des contrôles de sécurité automatisés dans la chaîne CI/CD — scan d'images, détection de secrets, analyse statique, tests."
---

# Sécurité dans la CI — DevSecOps

L'approche **DevSecOps** consiste à déplacer la sécurité « vers la gauche » (*shift-left*) : au lieu d'auditer le code en fin de cycle, on intègre des contrôles automatisés directement dans la chaîne CI/CD, à chaque commit. Une vulnérabilité détectée à l'écriture coûte beaucoup moins cher qu'une vulnérabilité détectée en production.

Cette page décrit la cible DevSecOps de SuperSOC et fournit les *jobs* GitLab prêts à intégrer.

## État actuel et cible

| Contrôle | Outil | État |
|----------|-------|------|
| Lint IaC (Terraform) | `terraform validate/fmt` | ✅ En place |
| Lint configuration (Ansible) | `ansible-lint` | ✅ En place |
| Scan d'images Docker | Trivy | 🎯 Cible |
| Détection de secrets | Gitleaks | 🎯 Cible |
| Lint code Python (SOAR) | Ruff / Flake8 | 🎯 Cible |
| Tests unitaires | Pytest | 🎯 Cible |
| Analyse statique (SAST) | GitLab SAST | 🎯 Cible |

:::info Posture assumée
La chaîne actuelle sécurise le **déploiement** (état centralisé, prod manuelle et protégée, secrets hors dépôt). L'étape suivante consiste à sécuriser le **code et les artefacts** applicatifs (SOAR FastAPI, images Docker). Les jobs ci-dessous constituent cette extension.
:::

## Stage `security`

On ajoute un *stage* dédié, exécuté tôt dans le pipeline :

```yaml
stages:
  - security        # ← nouveau, en amont
  - validate
  - plan
  # ...
```

## Détection de secrets — Gitleaks

Empêche qu'une clé, un mot de passe ou un token soit commité par erreur. C'est le contrôle le plus rentable : un secret fuité dans l'historique Git est compromis définitivement.

```yaml
secrets-scan:
  stage: security
  image:
    name: zricethezav/gitleaks:latest
    entrypoint: [""]
  before_script: []
  script:
    - gitleaks detect --source . --verbose --redact
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH
  allow_failure: false   # un secret détecté DOIT bloquer le pipeline
```

## Scan d'images Docker — Trivy

Analyse les images de la stack SOC (SOAR, frontend) à la recherche de CVE connues dans les paquets système et les dépendances. On échoue le job sur les vulnérabilités **HIGH** et **CRITICAL**.

```yaml
trivy-scan:
  stage: security
  image:
    name: aquasec/trivy:latest
    entrypoint: [""]
  before_script: []
  script:
    # Scan du Dockerfile du SOAR (analyse de configuration)
    - trivy config --severity HIGH,CRITICAL ./soar
    # Scan de l'image construite (CVE des dépendances)
    - trivy image --severity HIGH,CRITICAL --exit-code 1 soar-api:latest
  allow_failure: true   # informatif au début, à passer en bloquant ensuite
```

:::tip Justification orale
Commencer en `allow_failure: true` permet de **mesurer** la dette de vulnérabilités sans bloquer l'équipe, puis de passer en bloquant une fois la base assainie. C'est une démarche progressive défendable, pas un contournement.
:::

## Lint & tests du SOAR (Python)

Le code du SOAR (FastAPI, Python 3.11) doit être linté et testé comme tout artefact applicatif.

```yaml
python-quality:
  stage: security
  image: python:3.11-slim
  before_script:
    - pip install --no-cache-dir ruff pytest
    - cd soar
  script:
    - ruff check .                 # lint rapide (remplace Flake8 + isort)
    - pytest -q --maxfail=1        # tests unitaires
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH
```

## Analyse statique — SAST GitLab

GitLab fournit un *template* SAST clé en main qui détecte les vulnérabilités courantes (injection, désérialisation, etc.) dans le code.

```yaml
include:
  - template: Jobs/SAST.gitlab-ci.yml
```

## Pipeline cible (vue d'ensemble)

```text
security (secrets-scan, trivy, python-quality, SAST)
    ↓
validate → plan → apply           [Terraform]
    ↓
ansible-lint → deploy-test → staging → production   [Ansible]
```

## Principe de blocage

| Contrôle | Comportement recommandé |
|----------|-------------------------|
| Secrets (Gitleaks) | **Bloquant** dès le départ — risque maximal |
| Lint / tests Python | **Bloquant** — qualité minimale exigée |
| Trivy (CVE images) | Informatif → bloquant après assainissement |
| SAST | Informatif → revue manuelle des findings |

:::note À retenir pour l'oral
La sécurité dans la CI n'est pas un outil mais une **chaîne de défense en profondeur** : on empêche les secrets d'entrer (Gitleaks), on vérifie la qualité (Ruff/Pytest), on traque les CVE des dépendances (Trivy) et les failles de code (SAST). Chaque contrôle attrape une classe de problème différente.
:::
