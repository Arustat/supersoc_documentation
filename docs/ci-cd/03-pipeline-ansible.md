---
sidebar_position: 3
title: Pipeline Ansible
description: Étapes de configuration et de déploiement des services SOC (lint, dry-run, staging, production, rollback).
---

# Pipeline Ansible — Configuration & déploiement

Une fois l'infrastructure provisionnée par Terraform, Ansible installe et configure la stack SOC sur les VM (Wazuh, SOAR, bases de données, Portainer). Le pipeline applique une **montée en risque progressive** : on ne déploie en production qu'après avoir validé la syntaxe, la connectivité, puis un *dry-run* détaillé.

## Étapes

### 1. `ansible-lint` — automatique

Vérifie la qualité et la syntaxe des playbooks/rôles à chaque commit. Bloque tôt les erreurs courantes (modules dépréciés, idempotence douteuse, etc.).

```yaml
ansible-lint:
  script:
    - pip install --break-system-packages ansible-lint
    - ansible-galaxy collection install -r ansible/requirements.yml
    - ansible-lint ansible/playbooks/deploy_services.yml
```

### 2. `ansible-deploy-test` — automatique (dry-run)

Valide que le playbook **pourrait** s'exécuter, sans rien modifier : `--syntax-check`, test de connectivité SSH (`ping`), puis exécution en mode `--check` sur la cible SOC.

```yaml
ansible-playbook deploy_services.yml --limit soc --check -vvv
```

La connectivité au Cyber Range est testée en **non bloquant** (la VM peut être éteinte sans faire échouer le pipeline).

### 3. `ansible-deploy-staging` — manuel (dry-run détaillé)

Exécution en mode `--check --diff` : affiche **précisément** les changements qui seraient appliqués, secrets injectés à la volée depuis les variables CI. Le log est archivé en artefact pour relecture.

```yaml
ansible-playbook deploy_services.yml --check --diff -vvv \
  -e mariadb_root_password="$MARIADB_ROOT_PASSWORD" \
  -e jwt_secret_key="$JWT_SECRET_KEY" \
  | tee ansible/logs/staging-run-${CI_COMMIT_SHORT_SHA}.log
```

### 4. `ansible-deploy-production` — manuel + protégé

Le **seul** job qui applique réellement les changements. Trois garde-fous :

- `when: manual` — déclenchement humain explicite ;
- `resource_group: production-deployment` — **sérialise** les déploiements (jamais deux en parallèle) ;
- **health checks** post-déploiement automatiques (`docker ps` sur toutes les cibles).

```yaml
ansible-deploy-production:
  resource_group: production-deployment
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
      when: manual
```

### 5. `ansible-rollback` — manuel (urgence)

Permet d'arrêter proprement le déploiement courant (`docker compose down`) pour revenir à l'état précédent en cas d'incident. Également protégé par `resource_group`.

## Tableau récapitulatif

| Stage | Déclenchement | Modifie la prod ? | Rôle |
|-------|---------------|-------------------|------|
| `ansible-lint` | Auto | Non | Qualité/syntaxe |
| `ansible-deploy-test` | Auto | Non | Dry-run + SSH |
| `ansible-deploy-staging` | Manuel | Non | Dry-run détaillé (diff) |
| `ansible-deploy-production` | Manuel | **Oui** | Déploiement réel |
| `ansible-rollback` | Manuel | Oui | Retour arrière |

## Secrets requis

| Variable | Rôle |
|----------|------|
| `ANSIBLE_DEPLOY_KEY` | Clé SSH privée (base64) pour atteindre les VM |
| `MARIADB_ROOT_PASSWORD` / `MONGO_ROOT_PASSWORD` | Mots de passe des bases |
| `WAZUH_API_PASSWORD` / `WAZUH_INDEXER_PASSWORD` | Identifiants Wazuh |
| `JWT_SECRET_KEY` | Secret de signature JWT du SOAR (≥ 32 caractères) |

:::note Pourquoi staging puis production ?
Le mode `--check --diff` (staging) montre exactement ce qui va changer **avant** de l'appliquer. C'est l'équivalent d'un `terraform plan` pour Ansible : on relit le diff, puis on déclenche la production en connaissance de cause.
:::
