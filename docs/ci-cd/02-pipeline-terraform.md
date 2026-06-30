---
sidebar_position: 2
title: Pipeline Terraform
description: "Étapes de provisioning de l'infrastructure (validate, plan, apply)."
---

# Pipeline Terraform — Provisioning

Le pipeline Terraform provisionne l'infrastructure sur Proxmox : réseau, VLAN, VM pfSense, Docker-Host et Cyber Range. Il suit le triptyque classique **validate → plan → apply**, avec une application toujours manuelle.

## Étapes

### `validate` — automatique

Exécutée sur **toutes les branches** et les *merge requests*. Elle garantit que le code est correctement formaté et syntaxiquement valide avant toute autre étape.

```yaml
validate:
  stage: validate
  script:
    - terraform fmt -check -recursive   # le formatage est-il conforme ?
    - terraform validate                # la configuration est-elle valide ?
```

### `plan` — automatique

Génère le plan d'exécution (les changements que Terraform appliquerait) et le stocke en *artefact* pour qu'il soit réutilisé tel quel par l'étape `apply`. Le `-parallelism=1` évite les conflits sur l'API Proxmox.

```yaml
plan:
  stage: plan
  script:
    - terraform plan -parallelism=1 -out=plan.tfplan
  artifacts:
    paths:
      - ${TF_ROOT}/plan.tfplan
    expire_in: 1 week
```

### `apply` — manuel

Applique **exactement** le plan validé à l'étape précédente (`needs: plan`). Le déclenchement manuel (`when: manual`) impose une relecture humaine : aucune modification d'infrastructure n'est appliquée automatiquement.

```yaml
apply:
  stage: apply
  script:
    - terraform apply -parallelism=1 plan.tfplan
  needs:
    - job: plan
      artifacts: true
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
      when: manual
```

:::tip Job dédié pfSense
Un job `apply_pfsense` permet de créer **uniquement** la VM pfSense via `-target`, utile pour itérer sur le firewall sans retoucher le reste de l'infrastructure.
:::

## Gestion de l'état (state)

L'état Terraform est stocké dans le **backend HTTP de GitLab**, ce qui apporte :

- un **état centralisé** partagé par toute l'équipe (pas de `terraform.tfstate` local divergent) ;
- un **verrouillage** (`lock`/`unlock`) qui empêche deux `apply` simultanés de corrompre l'état ;
- une **authentification** par `CI_JOB_TOKEN`, donc sans secret supplémentaire à gérer.

```yaml
TF_HTTP_ADDRESS: "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/terraform/state/${TF_STATE_NAME}"
TF_HTTP_LOCK_METHOD: "POST"
TF_HTTP_UNLOCK_METHOD: "DELETE"
```

## Secrets requis

Définis dans **GitLab → Settings → CI/CD → Variables** (jamais dans le dépôt) :

| Variable | Rôle | Attributs |
|----------|------|-----------|
| `TF_VAR_proxmox_api_token` | Authentification API Proxmox | Masked, Protected |
| `PROXMOX_VE_SSH_PRIVATE_KEY` | Clé SSH pour les opérations CLI du provider | Masked, Protected |
| `TF_VAR_ssh_public_key` | Clé publique injectée dans les VM | Protected |
| `TF_VAR_pfsense_iso_id` | ISO pfSense à déployer | — |
