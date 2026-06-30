---
sidebar_position: 5
title: "Sécurité de l'API"
description: "Authentification, autorisation et audit de l'API SOAR."
---

# Sécurité de l'API

La sécurité de l'API est détaillée dans la section dédiée « Sécurité applicative ». En résumé, l'API SOAR applique :

- **Authentification JWT** : tokens d'accès courts + refresh.
- **Hachage argon2id** des mots de passe.
- **RBAC par dépendances FastAPI** : chaque route vérifie les permissions de façon déclarative.
- **Anti-bruteforce** : verrouillage de compte après échecs répétés.
- **Audit centralisé** : journalisation IP + user-agent des actions sensibles.

Pour le détail des mécanismes et leur justification, voir la section **Sécurité applicative**.
