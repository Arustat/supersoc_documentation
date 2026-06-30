---
sidebar_position: 1
title: "Vue d'ensemble"
description: "Stratégie de persistance polyglotte : MariaDB et MongoDB."
---

# Bases de données — Vue d'ensemble

SuperSOC adopte une approche de **persistance polyglotte** : deux bases de données aux rôles distincts, choisies selon la nature des données.

| Base | Type | Usage |
|------|------|-------|
| **MariaDB** | Relationnelle (SQL) | Tickets, utilisateurs, RBAC, audit, assets |
| **MongoDB** | Documentaire (NoSQL) | Alertes Wazuh, enrichissements IoC, logs d'exécution |

## Pourquoi deux bases ?

Le choix n'est pas gratuit, et c'est un point que le jury examinera :

- Les **données métier structurées** (un ticket a toujours un statut, une sévérité, un propriétaire) bénéficient des **contraintes d'intégrité relationnelles** et des transactions de MariaDB.
- Les **données semi-structurées et volumineuses** (une alerte Wazuh est un JSON dont le schéma varie selon la règle déclenchée) s'accommodent mieux de la **flexibilité documentaire** de MongoDB.

Forcer les alertes Wazuh dans un schéma relationnel rigide imposerait des migrations à chaque évolution de format ; à l'inverse, gérer le RBAC et les tickets en NoSQL ferait perdre les garanties d'intégrité.

:::note Persistance polyglotte
« Utiliser la bonne base pour le bon usage » plutôt qu'une base unique pour tout. C'est une décision d'architecture défendable, pas une complexité gratuite.
:::
