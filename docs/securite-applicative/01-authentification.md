---
sidebar_position: 1
title: "Authentification"
description: "Authentification JWT et hachage des mots de passe."
---

# Authentification

L'authentification de l'API SOAR repose sur des **tokens JWT** et un **hachage robuste** des mots de passe.

## JWT — tokens d'accès et de rafraîchissement

L'API utilise deux types de tokens :

| Token | Durée de vie | Rôle |
|-------|--------------|------|
| **Access token** | Courte | Autorise les requêtes API |
| **Refresh token** | Longue | Obtient un nouvel access token sans re-saisie |

Ce schéma équilibre **sécurité** et **confort** : un access token court limite la fenêtre d'exploitation s'il fuite ; le refresh token évite de redemander les identifiants en permanence.

:::note Révocabilité
Un access token JWT est par nature valable jusqu'à son expiration. Le couple access court + refresh permet de **limiter l'impact** d'un token volé : sa durée de vie réduite borne la fenêtre d'attaque.
:::

## Hachage des mots de passe — argon2id

Les mots de passe ne sont **jamais stockés en clair**. Ils sont hachés avec **argon2id**, l'algorithme recommandé par l'OWASP pour le stockage de mots de passe :

- Résistant aux attaques par GPU (coût mémoire élevé).
- Salage intégré.
- Paramétrable (coût mémoire, temps, parallélisme).

argon2id est préféré à des algorithmes plus anciens (MD5, SHA-1, voire bcrypt) car il est conçu spécifiquement contre les attaques matérielles modernes.

## Protection anti-bruteforce

Le compte se **verrouille** après un nombre défini de tentatives échouées (`failed_login_attempts`, `locked_until` dans le modèle utilisateur), ce qui ralentit les attaques par force brute.
