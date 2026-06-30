---
sidebar_position: 5
title: "Sauvegardes & restauration"
description: "Stratégie de sauvegarde des bases MariaDB et MongoDB."
---

# Sauvegardes & restauration

La stratégie de sauvegarde garantit la **récupération des données** en cas d'incident, et alimente directement le plan de reprise (PRA).

## Périmètre

Quatre scripts assurent sauvegarde et restauration des deux bases :

| Script | Rôle |
|--------|------|
| `backup_mariadb.sh` | Dump MariaDB |
| `backup_mongodb.sh` | Dump MongoDB |
| `restore_mariadb.sh` | Restauration MariaDB |
| `restore_mongodb.sh` | Restauration MongoDB |

## Principes appliqués

- **Stratégie 3-2-1 simplifiée** : sauvegardes régulières, conservées avec une rétention définie.
- **Rétention 7 jours** : rotation automatique des dumps au-delà.
- **Dumps cohérents** : `--single-transaction` côté MariaDB pour une sauvegarde sans verrou bloquant ni incohérence.
- **Robustesse des scripts** : `set -euo pipefail` (arrêt à la première erreur), rotation effectuée **après** un dump réussi uniquement.
- **Permissions restrictives** sur les fichiers de sauvegarde et secrets.

## Lien avec le PRA

Les sauvegardes définissent le **RPO** (perte de données maximale acceptable) : avec une sauvegarde quotidienne, le RPO est de 24 h au pire cas. Couplées à l'infrastructure reproductible (Terraform + Ansible), elles permettent de reconstruire un service et d'y réinjecter les données.

:::note Piste d'amélioration assumée
Les dumps sont compressés et protégés par des permissions restrictives, mais **pas chiffrés au repos** dans la version prototype. Le chiffrement des sauvegardes et une copie hors-site constituent la suite logique (perspective d'évolution).
:::
