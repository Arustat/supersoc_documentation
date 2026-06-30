---
sidebar_position: 4
title: "Contrôle d'accès (DCL)"
description: "Comptes de service au moindre privilège sur MariaDB et MongoDB."
---

# Contrôle d'accès aux bases (DCL)

Le contrôle d'accès applique le **principe de moindre privilège** : chaque composant ne dispose que des droits strictement nécessaires.

## MariaDB — comptes de service séparés

Plutôt qu'un unique compte root partagé, le déploiement crée **plusieurs comptes applicatifs** aux droits distincts :

| Compte | Droits | Usage |
|--------|--------|-------|
| `soar_app` | CRUD sur les tables métier | Application SOAR en fonctionnement |
| `soar_migration` | DDL (création/modification de tables) | Migrations de schéma uniquement |
| `soar_readonly` | SELECT | Lecture (reporting, dashboard) |
| `soar_audit` | INSERT/SELECT sur l'audit | Journalisation append-only |

Cette séparation limite l'impact d'une compromission : un vol des identifiants `soar_readonly` ne permet aucune écriture ; le compte applicatif ne peut pas altérer la structure des tables.

## MongoDB — utilisateurs à rôle limité

| Utilisateur | Rôle |
|-------------|------|
| `soar_app` | `readWrite` sur la base SOAR |
| `soar_readonly` | `read` seul |

## Bonnes pratiques appliquées

- Aucun compte root utilisé par l'application.
- Mots de passe injectés via variables d'environnement (`.env` en `0600`), jamais en dur.
- Bases non exposées réseau (`internal: true`) : accès limité aux services du réseau Docker interne.

:::tip Question probable du jury
« Que se passe-t-il si l'API SOAR est compromise ? » → L'attaquant hérite des droits de `soar_app` (CRUD métier), mais **pas** des droits de migration (DDL) ni d'un accès root. Le journal d'audit, alimenté par un compte distinct, conserve la trace des actions.
:::
