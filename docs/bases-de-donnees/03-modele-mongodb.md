---
sidebar_position: 3
title: "Modèle documentaire (MongoDB)"
description: "Collections MongoDB, validation de schéma et index."
---

# Modèle documentaire — MongoDB

MongoDB stocke les données semi-structurées et volumineuses du SOC.

## Collections

| Collection | Contenu |
|------------|---------|
| `wazuh_alerts` | Alertes brutes émises par Wazuh (JSON variable) |
| `ioc_enrichments` | Résultats d'enrichissement IoC (VirusTotal, AbuseIPDB) |
| `playbook_execution_logs` | Traces d'exécution des playbooks SOAR |

## Validation de schéma

Bien que NoSQL, les collections ne sont pas un « fourre-tout » : chaque collection est protégée par un **validateur `$jsonSchema`** qui impose les champs obligatoires et leurs types. On obtient la flexibilité documentaire **sans renoncer** à un minimum d'intégrité.

:::note Démonstration concrète
Un document mal formé est **rejeté** par le validateur à l'insertion. Ce rejet n'est pas un bug : c'est la preuve que le contrôle d'intégrité fonctionne, même en NoSQL.
:::

## Index

- **Index classiques** sur les champs de recherche fréquents (IP, hash, identifiant d'alerte).
- **Index TTL** (Time-To-Live) pour la **purge automatique** des documents au-delà d'une durée de rétention — utile pour les logs volumineux qui n'ont pas vocation à être conservés indéfiniment.

## Flexibilité démontrée

Un champ optionnel `vulnerability` (CVE / score CVSS) peut être ajouté à un enrichissement **sans migration** ni modification de schéma rigide. C'est l'illustration directe de l'intérêt du documentaire pour ce type de données évolutives.
