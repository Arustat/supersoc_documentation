---
sidebar_position: 2
title: "Réseau & VLAN"
description: "Segmentation réseau, VLAN et règles de filtrage pfSense."
---

# Réseau & Segmentation VLAN

La segmentation réseau applique le principe de **cloisonnement** : une compromission dans une zone ne doit pas se propager aux autres. pfSense assure le routage inter-VLAN et le filtrage.

## Plan d'adressage

| VLAN | Zone             | Sous-réseau     | Contenu                          |
|------|------------------|-----------------|----------------------------------|
| 10   | Administration   | 10.0.10.0/24    | Proxmox, pfSense UI, Portainer   |
| 20   | SOC Services     | 10.0.20.0/24    | Wazuh, SOAR FastAPI, Vue.js      |
| 30   | Bases de données | 10.0.30.0/24    | Réservé (extension — DB dédiées) |
| 100  | Cyber Range      | 10.0.100.0/24   | DVWA, agents, cibles d'attaque   |

Le bridge interne `vmbr2` est configuré en `vlan_aware` sur Proxmox et sert de trunk vers le LAN pfSense.

## Logique de filtrage

- **Le Cyber Range (VLAN 100) est le plus contraint** : c'est la zone où l'on simule des attaques. Il peut émettre vers Wazuh (collecte de logs) mais ne doit pas atteindre l'administration.
- **L'administration (VLAN 10) est la plus protégée** : accessible uniquement via VPN.
- **Le SOC (VLAN 20)** reçoit les logs des agents et expose les interfaces analyste (Wazuh Dashboard, SOAR).

## Accès distant — WireGuard

L'accès administrateur se fait **exclusivement** par VPN WireGuard, terminé sur pfSense. Aucun service d'administration n'est exposé directement sur Internet. WireGuard est retenu pour sa simplicité de configuration, ses performances (kernel) et sa surface d'attaque réduite.

:::tip À retenir pour l'oral
La segmentation n'a de valeur que si les règles de filtrage l'accompagnent. Un VLAN sans règle pare-feu restrictive n'isole rien. Le couple **VLAN + règles pfSense** matérialise la défense en profondeur.
:::
