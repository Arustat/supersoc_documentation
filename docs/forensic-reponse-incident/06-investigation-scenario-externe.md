---
sidebar_position: 6
title: "Investigation — Scénario externe"
description: "Analyse des traces générées par la campagne de test d'intrusion black-box sur SuperSOC."
---

# Investigation — Scénario externe

Ce document analyse, du point de vue de la détection, la campagne de test d'intrusion black-box menée sur le Docker-Host `10.0.20.10`. Il relie les actions offensives effectuées aux traces qu'elles génèrent et à leur détection côté SOC.

## 1. Contexte

Une campagne offensive black-box a été menée depuis une position réseau interne (VPN), sans identifiant. Elle visait la surface exposée par le Docker-Host et les mécanismes de sécurité de l'API SOAR.

## 2. Chronologie des actions offensives

Chronologie reconstituée à partir du journal de campagne et des sorties d'outils.

| Heure (EDT) | Action offensive | Trace générée | Détection SOC |
|-------------|------------------|---------------|---------------|
| 11:22 | `nmap -sn` (découverte réseau) | Flux ICMP/ARP sur le VLAN | Règle de détection de scan |
| 11:26 | `nmap -p` (scan ports web) | Connexions TCP multiples | Règle de détection de scan |
| 11:30 | `nmap -sV -sC` (versions) | Sondes services | Règle de détection de scan |
| 11:32 | Énumération headers/API (`curl`) | Requêtes HTTP anormales | Logs web |
| 11:40 | `gobuster` (énum chemins) | Rafale de requêtes 404/301 | Règle de scan web |
| 11:45 | Brute force login (`/auth/login`) | Échecs d'authentification répétés | Règle bruteforce |
| 11:56 | `nikto` (scan vulnérabilités) | Signature User-Agent nikto | Règle de scan web |
| 12:01 | `sqlmap` (injection login) | Payloads SQL dans les requêtes | Règle d'injection |

## 3. Traces observées (côté attaquant)

Ces éléments sont constatés et documentés dans le rapport de pentest.

### Reconnaissance réseau
Le scan de ports a confirmé que seul `10.0.20.10` expose des services (80, 443, 8000, 8080, 55000), les autres adresses étant filtrées par pfSense.

### Brute force d'authentification
La séquence de tentatives sur `/api/v1/auth/login` a produit une réponse observable et significative :

```text
tentatives 1-4  : HTTP 401  {"detail":"Identifiants invalides"}
tentative 5     : HTTP 423  {"detail":"Compte temporairement verrouillé..."}
tentatives 6+   : HTTP 423  (verrouillage maintenu)
```

Cette bascule 401 → 423 est la trace applicative de l'attaque, exploitable en corrélation avec les logs de l'API.

## 4. Indicateurs de compromission (IOC) de la campagne

| Type | Valeur | Source |
|------|--------|--------|
| IP source attaquant | IP VPN de la machine d'attaque | journal |
| User-Agent | `gobuster/3.8`, `nikto`, `sqlmap` | requêtes HTTP |
| Endpoint ciblé | `/api/v1/auth/login` | logs API |
| Pattern | Rafale de 401 puis 423 | logs API |

## 5. Détection en conditions réelles

Avec une collecte Wazuh active, les actions ci-dessus déclenchent :

- **Scan de ports** : règles de détection de scan (nombreuses connexions depuis une même source).
- **Énumération web** (gobuster/nikto) : pic de requêtes 404, User-Agents d'outils connus.
- **Brute force** : règle sur les échecs d'authentification répétés — corrélable avec le verrouillage applicatif (423).

## 6. Enseignements

- Les **mécanismes applicatifs** (verrouillage, 401 systématiques) offrent une détection même en l'absence de SIEM : l'API elle-même trace et réagit.
- La **défense en profondeur** joue : segmentation pfSense (surface réduite) + contrôle d'accès applicatif + anti-bruteforce.
- La **continuité de la collecte** est une exigence opérationnelle : la persistance des index et la sauvegarde de la configuration Wazuh garantissent la mémoire des incidents.
