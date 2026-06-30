---
sidebar_position: 2
title: "Modèle relationnel (MariaDB)"
description: "Schéma MariaDB : 14 tables, RBAC et intégrité référentielle."
---

# Modèle relationnel — MariaDB

MariaDB stocke le cœur métier du SOC : tickets, utilisateurs, rôles, permissions, assets, et journal d'audit.

## Entités principales

| Table | Rôle |
|-------|------|
| `Utilisateur` | Comptes analystes (hash, MFA, état du compte) |
| `Role` / `Permission` | RBAC : rôles et permissions |
| `Posseder` / `Accorder` | Associations user↔rôle et rôle↔permission |
| `Ticket` | Incidents (titre, sévérité, statut, source) |
| `Categorie` | Classement des tickets |
| `Commentaire` | Échanges sur un ticket |
| `Historique` | Changements de statut horodatés |
| `PieceJointe` | Fichiers liés à un ticket |
| `Actif` (asset) | Machines surveillées |
| `Playbook` / `Execution` | Définitions SOAR et exécutions |
| `Audit` | Journal des actions sensibles |

## RBAC — contrôle d'accès par rôle

Le modèle dissocie **utilisateurs**, **rôles** et **permissions** via des tables d'association. Un utilisateur possède un ou plusieurs rôles ; un rôle accorde un ensemble de permissions. Cette granularité permet d'ajouter un rôle (ex. « analyste junior ») sans toucher au code.

## Journal d'audit

La table `Audit` enregistre chaque action sensible (acteur, type de ressource, IP, user-agent, horodatage). Elle est conçue pour être **append-only** : on n'y modifie ni n'y supprime de ligne, ce qui garantit la **non-répudiation** et fournit la base d'une investigation forensique.

:::tip Atout du modèle
Le champ `source_alert_id` du ticket fait le **pont avec MongoDB** : un ticket créé automatiquement par le SOAR référence l'alerte Wazuh d'origine, stockée côté documentaire. Les deux mondes sont reliés sans être fusionnés.
:::
