---
sidebar_position: 2
title: "Autorisation (RBAC)"
description: "Contrôle d'accès basé sur les rôles via les dépendances FastAPI."
---

# Autorisation — RBAC

Une fois l'utilisateur authentifié, le **RBAC** (Role-Based Access Control) détermine ce qu'il a le droit de faire.

## RBAC par dépendances FastAPI

L'autorisation est implémentée via le système de **dépendances** de FastAPI : chaque route déclare la permission ou le rôle requis, et la vérification est effectuée **avant** l'exécution du handler.

Avantages de cette approche :

- **Déclaratif** : la contrainte d'accès est visible directement sur la route.
- **Centralisé** : la logique de vérification est factorisée, pas réécrite à chaque endpoint.
- **Difficile à oublier** : une route protégée déclare explicitement sa dépendance ; on ne risque pas d'oublier un `if user.role == ...` dispersé dans le code.

## Modèle de rôles

Le RBAC s'appuie sur le modèle relationnel (voir Bases de données) : utilisateurs ↔ rôles ↔ permissions. Ajouter un rôle ou ajuster des permissions ne nécessite pas de modifier le code des routes.

:::tip Question probable du jury
« Comment garantissez-vous qu'une route sensible n'est pas accessible sans permission ? » → La permission est une **dépendance de la route** : sans elle, FastAPI rejette la requête avant d'atteindre la logique métier. Le contrôle est structurel, pas optionnel.
:::
