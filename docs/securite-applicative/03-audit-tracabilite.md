---
sidebar_position: 3
title: "Audit & traçabilité"
description: "Journal d'audit append-only et non-répudiation."
---

# Audit & traçabilité

La traçabilité est une exigence de sécurité : pouvoir répondre à « qui a fait quoi, quand, depuis où ».

## Journal d'audit centralisé

Chaque action sensible est journalisée dans la table `Audit` avec :

- l'**acteur** (utilisateur),
- l'**action** et le **type de ressource** concernée,
- l'**adresse IP** et le **user-agent**,
- l'**horodatage**.

## Append-only & non-répudiation

Le journal est conçu pour être **append-only** : on y ajoute des entrées, on n'en modifie ni n'en supprime. Cette propriété garantit la **non-répudiation** — un utilisateur ne peut pas nier une action, et un attaquant ne peut pas effacer ses traces dans le journal applicatif.

Côté base, un compte dédié (`soar_audit`) aux droits limités alimente cette table, séparément du compte applicatif principal.

## Base de l'investigation forensique

En cas d'incident, ce journal constitue la **source primaire** de l'investigation : reconstitution de la chronologie, identification de l'acteur, corrélation avec les alertes Wazuh stockées dans MongoDB.

:::note Cohérence défense en profondeur
L'audit applicatif (qui a fait quoi dans le SOAR) complète les logs Wazuh (que s'est-il passé sur les machines). Les deux ensemble donnent une vue complète exploitable en forensic.
:::
