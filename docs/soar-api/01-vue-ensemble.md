---
sidebar_position: 1
title: "Vue d'ensemble"
description: "Le SOAR de SuperSOC : orchestration et automatisation de la réponse."
---

# SOAR & API — Vue d'ensemble

Le **SOAR** (Security Orchestration, Automation and Response) est le cœur applicatif de SuperSOC. C'est une **API asynchrone développée sur-mesure** en Python 3.11 avec **FastAPI**, qui transforme les alertes de détection en réponses coordonnées.

## Rôle

Le SOAR fait le lien entre la **détection** (Wazuh) et la **réponse** (tickets, blocage, notification) :

```text
Wazuh (alerte)
   ↓ webhook
SOAR API (FastAPI)
   ├── crée un ticket (MariaDB)
   ├── enrichit les IoC (VirusTotal / AbuseIPDB → MongoDB)
   ├── exécute un playbook (blocage IP via pfSense)
   └── journalise (audit)
```

## Pourquoi FastAPI ?

- **Asynchrone** : les enrichissements IoC font des appels réseau externes (VirusTotal, AbuseIPDB) ; l'async évite de bloquer l'API pendant ces attentes.
- **Validation native** via Pydantic : les données entrantes sont validées automatiquement.
- **Documentation auto-générée** (OpenAPI / Swagger) : utile pour la démonstration et les tests.

## Organisation du code

L'API est structurée en **routers** par domaine fonctionnel (auth, tickets, alerts, playbooks, ioc, firewall, etc.) et en **services** qui portent la logique métier (ingestion d'alertes, enrichissement, moteur de playbooks, notification, audit).

:::note Développé sur-mesure
Le SOAR n'est pas un outil sur étagère configuré, mais un développement propre à l'équipe. C'est ce qui permet de justifier chaque choix d'architecture à l'oral.
:::
