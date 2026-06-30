---
sidebar_position: 4
title: "Durcissement (Hardening)"
description: "Mesures de durcissement OS, Docker et réseau appliquées à l'infrastructure."
---

# Durcissement de l'infrastructure

Le durcissement applique le principe de **réduction de la surface d'attaque** à chaque couche : système, conteneurs et réseau.

## Niveau réseau

- **Segmentation VLAN** : isolation des zones (admin / SOC / cyber range).
- **Aucun port d'administration exposé** sur Internet ; accès via VPN WireGuard uniquement.
- **Filtrage pfSense** entre VLAN selon le principe du moindre accès.

## Niveau conteneurs (Docker)

Les `docker-compose` appliquent plusieurs mesures de durcissement :

- **`no-new-privileges:true`** : empêche un processus d'élever ses privilèges dans le conteneur.
- **Réseau `internal: true`** pour les bases de données : aucune exposition de port vers l'extérieur, communication interne uniquement.
- **Healthchecks** : détection automatique d'un service défaillant.
- **Volumes nommés** : persistance maîtrisée des données.
- **Secrets via `.env`** en mode `0600`, jamais en clair dans les images.

```yaml
services:
  mariadb:
    security_opt:
      - no-new-privileges:true
    networks:
      - internal
networks:
  internal:
    internal: true   # aucun accès réseau externe
```

## Niveau système

- Initialisation via cloud-init (pas de configuration manuelle non tracée).
- Accès SSH par clé uniquement (clé injectée par Terraform/cloud-init).
- Droits de fichiers restrictifs sur les secrets et configurations.

:::tip Question probable du jury
« Vos bases de données sont-elles accessibles depuis l'extérieur ? » → Non : le réseau Docker est `internal: true`, MariaDB et MongoDB ne publient aucun port. Seuls les services applicatifs du même réseau Docker peuvent les joindre.
:::
