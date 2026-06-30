---
sidebar_position: 1
title: Vue d'ensemble
description: Architecture générale de la chaîne CI/CD de SuperSOC.
---

# CI/CD — Vue d'ensemble

La chaîne d'intégration et de déploiement continus de SuperSOC est hébergée sur **GitLab CI/CD**, avec un *runner* auto-hébergé (`supersoc-runner`) situé dans le réseau local afin d'atteindre l'hyperviseur Proxmox et les VM cibles.

Elle poursuit trois objectifs :

- **Reproductibilité** — toute l'infrastructure est décrite en code (Terraform, Ansible) et rejouable à l'identique.
- **Sécurité du déploiement** — séparation stricte des étapes, déploiement de production manuel et protégé.
- **DevSecOps** — intégration de contrôles de sécurité automatisés dans la chaîne (voir [Sécurité dans la CI](./04-securite-ci)).

## Deux chaînes complémentaires

Le pipeline se décompose en deux ensembles cohérents, déclenchés dans le même fichier `.gitlab-ci.yml` :

| Chaîne | Rôle | Outil |
|--------|------|-------|
| **Provisioning** | Créer/mettre à jour les VM et le réseau sur Proxmox | Terraform |
| **Configuration & déploiement** | Installer et configurer les services SOC sur les VM | Ansible |

## Flux global

```text
┌─────────────────── PROVISIONING (Terraform) ───────────────────┐
│  validate  →  plan  →  apply [MANUEL]                           │
└────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────── CONFIGURATION (Ansible) ───────────────────────┐
│  ansible-lint  →  deploy-test  →  deploy-staging [MANUEL]       │
│                              →  deploy-production [MANUEL]       │
│                              →  rollback [MANUEL]                │
└────────────────────────────────────────────────────────────────┘
```

## Principes de conception

- **Le déploiement de production est toujours manuel** (`when: manual`) et protégé par un `resource_group` qui sérialise les exécutions pour éviter deux déploiements concurrents.
- **Le `terraform apply` est manuel** : aucune modification d'infrastructure n'est appliquée sans relecture humaine du `plan`.
- **L'état Terraform est centralisé** dans le backend HTTP de GitLab (verrouillage concurrent géré nativement).
- **Les secrets ne sont jamais dans le dépôt** : ils proviennent des variables CI/CD GitLab (masquées et protégées).

:::note Pourquoi un runner auto-hébergé ?
Le déploiement cible des machines du réseau privé (Proxmox, pfSense, VM SOC) non exposées sur Internet. Un runner SaaS GitLab.com n'y aurait pas accès. Le runner local résout cette contrainte tout en gardant les secrets dans le périmètre maîtrisé.
:::
