---
sidebar_position: 3
title: "Enrichissement IoC"
description: "Enrichissement des indicateurs de compromission via sources externes."
---

# Enrichissement des IoC

L'enrichissement donne du **contexte** à une alerte : une IP ou un hash isolé ne dit rien ; enrichi, il devient exploitable par l'analyste.

## Sources

| Source | Apport |
|--------|--------|
| **VirusTotal** | Réputation de fichiers/URLs/IP, détections multi-moteurs |
| **AbuseIPDB** | Score de réputation d'une IP, historique d'abus signalés |

## Fonctionnement

1. Le service extrait les **indicateurs** (IP, hash) de l'alerte.
2. Il interroge les API externes **de façon asynchrone**.
3. Les résultats sont stockés dans MongoDB (`ioc_enrichments`) et rattachés à l'alerte/ticket.

L'asynchronisme est ici essentiel : les appels réseau externes peuvent être lents, et l'API ne doit pas se bloquer pendant ce temps.

## Robustesse

Les sources externes peuvent être indisponibles ou limitées en quota. Le service est conçu pour **dégrader gracieusement** : l'absence d'enrichissement n'interrompt pas le traitement de l'alerte, elle est simplement notée.

:::note Valeur analyste
Un score AbuseIPDB élevé sur l'IP source d'une alerte permet de **prioriser** immédiatement : c'est la différence entre une alerte « à regarder » et un incident « à traiter en urgence ».
:::
