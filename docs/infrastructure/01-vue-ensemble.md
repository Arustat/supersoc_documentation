---
sidebar_position: 1
title: "Vue d'ensemble"
description: "Architecture matérielle et virtualisée de SuperSOC."
---

# Infrastructure — Vue d'ensemble

SuperSOC est déployé sur un **serveur bare metal** sous **Proxmox VE**, qui héberge l'ensemble des machines virtuelles du SOC. L'isolation réseau repose sur un pare-feu **pfSense** et un découpage en **VLAN**, avec un accès distant exclusivement par **VPN WireGuard**.

## Topologie

```text
Proxmox VE (bare metal)
├── VM 100 — pfSense           Firewall, routage VLAN, VPN WireGuard
├── VM 200 — Docker-Host       Stack SOC complète (Docker Compose)
│   ├── wazuh-manager          SIEM/XDR — collecte, règles, alertes
│   ├── wazuh-indexer          OpenSearch (stockage des alertes)
│   ├── wazuh-dashboard        Interface Wazuh
│   ├── soar-api               Backend FastAPI (Python 3.11)
│   ├── soar-frontend          Dashboard Vue.js 3 (Nginx)
│   ├── mariadb                Tickets, RBAC, assets
│   ├── mongodb                Logs bruts, enrichissements IoC
│   └── portainer              Gestion UI des conteneurs
└── VM 300 — Cyber Range       DVWA, agents Wazuh, simulation d'attaques
```

## Principes directeurs

- **Tout est auto-hébergé** : aucune dépendance à un service cloud externe pour le cœur du SOC.
- **Infrastructure as Code** : le provisioning (Terraform) et la configuration (Ansible) sont versionnés et reproductibles.
- **Segmentation réseau** : chaque zone (admin, SOC, données, cyber range) est isolée dans un VLAN dédié.
- **Aucun port exposé sur Internet** : l'accès administrateur passe uniquement par le VPN WireGuard.

:::note Choix du bare metal
Wazuh (et son indexer OpenSearch) est gourmand en RAM et en I/O. Un déploiement bare metal via Proxmox offre de meilleures performances et un contrôle total du réseau, indispensable pour la segmentation VLAN.
:::
