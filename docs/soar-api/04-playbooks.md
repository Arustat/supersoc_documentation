---
sidebar_position: 4
title: "Moteur de playbooks"
description: "Automatisation de la réponse : exécution des playbooks SOAR."
---

# Moteur de playbooks

Les playbooks automatisent la **réponse** à un incident. Un playbook est une suite d'actions déclenchées selon le contexte de l'alerte.

## Exemple : blocage d'IP malveillante

1. Une alerte critique référence une IP à la réputation mauvaise (confirmée par l'enrichissement).
2. Le playbook de réponse demande le **blocage de l'IP** au pare-feu pfSense (via son API).
3. L'action et son résultat sont **journalisés** (audit + `playbook_execution_logs`).

## États d'exécution

Une action de réponse peut aboutir à plusieurs états, tous tracés :

| État | Signification |
|------|---------------|
| `applied` | Action réellement appliquée (ex. règle pfSense créée) |
| `simulated` | Action simulée — pfSense indisponible ou mode dégradé |
| `failed` | Échec de l'action |

L'état `simulated` correspond à une **dégradation gracieuse** : si pfSense n'est pas joignable, le SOAR ne plante pas, il enregistre que l'action aurait dû être appliquée. Le système reste fonctionnel même en environnement incomplet.

:::tip Nuance importante pour l'oral
`simulated` ne signifie pas « test avant application ». C'est un **mode dégradé** : l'action n'a pas pu être appliquée (pfSense absent), mais elle est tracée pour ne rien perdre. Cette distinction montre une vraie compréhension de la résilience.
:::

## Traçabilité

Chaque exécution est consignée : quel playbook, déclenché par qui/quoi, sur quel ticket, avec quel résultat. Ces traces alimentent l'audit et l'investigation forensique.
