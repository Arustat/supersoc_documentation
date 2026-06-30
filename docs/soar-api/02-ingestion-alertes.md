---
sidebar_position: 2
title: "Ingestion des alertes"
description: "De l'alerte Wazuh au ticket : le service d'ingestion."
---

# Ingestion des alertes

Le service d'ingestion est le point d'entrée du SOAR : il reçoit les alertes de Wazuh et décide de la suite.

## Flux

1. **Réception** : Wazuh émet une alerte (webhook) vers l'API.
2. **Transformation** : l'alerte brute (JSON Wazuh) est normalisée et stockée dans MongoDB (`wazuh_alerts`).
3. **Décision** : selon la **criticité** de l'alerte, le service crée — ou non — un ticket dans MariaDB.
4. **Liaison** : le ticket conserve la référence à l'alerte d'origine (`source_alert_id`).

## Création automatique selon un seuil

Toutes les alertes ne créent pas un ticket : un **seuil de criticité** filtre le bruit. Au-dessus du seuil, le ticket est créé automatiquement ; en dessous, l'alerte est conservée pour consultation mais ne génère pas de ticket.

Ce mécanisme répond directement au problème de l'**alert fatigue** : on évite de noyer l'analyste sous des tickets non pertinents.

:::tip Cohérence à vérifier
Le seuil de création doit être **identique** dans le code, la documentation et le diaporama. Une incohérence (ex. seuil 7 vs 10 selon les documents) est exactement le genre de détail qu'un jury relève. Vérifier la valeur réelle dans le code et l'harmoniser partout.
:::
