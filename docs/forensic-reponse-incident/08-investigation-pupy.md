---
sidebar_position: 8
title: "Investigation — Scénario RAT pupy"
description: "Investigation forensic de l'incident RAT pupy sur la Cyber Range, basée sur l'acquisition collect.sh."
---

# Investigation — Scénario RAT pupy

Investigation forensic menée à la suite de la campagne Red Team pupy sur la Cyber Range. Elle suit le cycle NIST/SANS et s'appuie sur l'acquisition automatisée réalisée par le script `collect.sh`.

## 1. Résumé exécutif

Un poste de la Cyber Range (`cr-victim-01`) a été compromis via un RAT pupy déployé par phishing. L'agent Wazuh a détecté les IOC de dépôt et de persistance, et la règle de corrélation `100150` a élevé un incident de niveau 14. L'acquisition forensic a confirmé la présence du RAT, du canal C2 et des mécanismes de persistance.

## 2. Méthodologie

Investigation selon le cycle **NIST SP 800-61 / SANS PICERL** : Préparation → Identification → Confinement → Éradication → Récupération → Leçons apprises. L'acquisition respecte l'**ordre de volatilité (RFC 3227)** : volatil (processus, réseau, mappings mémoire) avant persistant (fichiers, logs).

## 3. Acquisition des preuves — `collect.sh`

L'acquisition est réalisée par le script `forensics/collect.sh`, qui produit une archive horodatée et un **manifeste de hashes SHA256** (chaîne de preuve). Le script ne modifie pas la cible et écrit sa sortie hors de la victime.

```bash
./collect.sh cr-victim-01
# Produit : forensics-cr-victim-01-<horodatage>.tar.gz + .sha256
```

Artefacts collectés (extrait) :

| Fichier | Contenu |
|---------|---------|
| `10_processes_*.txt` | Processus (détection du masquage `compiz`) |
| `20_sockets_*.txt` | Connexions réseau (canal C2) |
| `30_rwx_mappings.txt` | Régions mémoire RWX (injection réflexive pupy) |
| `40_autostart_*` / `41_systemd_dropins.txt` | Persistance |
| `50_ioc_files.txt` | Fichiers IOC connus |
| `70_bash_history.txt` / `71_auth_log.txt` | Historique & authentification |
| `SHA256SUMS` | Intégrité de la collecte |

## 4. Chronologie de l'incident

| Heure (UTC) | Événement | Source |
|-------------|-----------|--------|
| 14:02 | Email phishing ouvert | GoPhish |
| 14:03 | Exécution pupy, dépôt `/usr/bin/dcrond` | Syscheck → règle `100100` |
| 14:03 | Canal C2 établi | `20_sockets_ss.txt` |
| 14:04 | Persistance systemd installée | Syscheck → règle `100110` |
| 14:05 | Corrélation multi-IOC | **Incident `100150` (niveau 14)** |
| 14:20 | Acquisition forensic | `collect.sh` |

## 5. Indicateurs de compromission (IOC)

| Type | Valeur | Détection |
|------|--------|-----------|
| Binaire masqué | `/usr/bin/dcrond` | règle `100100` |
| Shared object | `/lib/lib{RAND}.so.1` | règle `100101` |
| Fichier utilisateur | `~/.cache/mozilla/...libflushplugin.so` | règle `100102` |
| Persistance | drop-in systemd `dbus.service.d/` | règle `100110` |
| Processus masqué | `compiz` (faux nom) | règle `100120` |
| Connexion C2 |IP:port du serveur C2 | `20_sockets_ss.txt` |
| Région mémoire | mapping RWX anonyme | `30_rwx_mappings.txt` |

:::note Anti-forensic : timestomping
pupy peut altérer les horodatages (`mtime`) des fichiers déposés. L'analyse doit donc se fier au **hash SHA256** et à la corrélation multi-sources plutôt qu'aux seules dates de fichiers. Ce réflexe est intégré à la démarche.
:::

## 6. Analyse

### Processus
Le RAT se masque sous un nom légitime (`compiz`). L'arbre de processus (`11_process_tree.txt`) révèle la filiation anormale et l'argv réel.

### Réseau (C2)
La socket C2 apparaît dans `20_sockets_ss.txt` : connexion sortante persistante vers le serveur d'attaque, chiffrée.

### Persistance
Trois vecteurs possibles, tous surveillés : drop-in systemd, autostart XDG, `rc.local`. L'acquisition confirme le(s) mécanisme(s) réellement utilisé(s).

### Mémoire
`30_rwx_mappings.txt` liste les régions mémoire RWX anonymes, signature du chargement réflexif de modules pupy en RAM.

## 7. Confinement, éradication, récupération

- **Confinement** : isolation réseau de la victime (le VLAN 100 est déjà `deny-all` sortant), blocage de l'IP C2.
- **Éradication** : suppression des binaires IOC, des mécanismes de persistance, du processus masqué.
- **Récupération** : redéploiement du conteneur victime depuis l'image saine (infra reproductible), vérification Syscheck du retour à l'état sain.

## 8. Leçons apprises

- Les règles personnalisées `100100`–`100112` détectent les IOC unitaires ; la **corrélation `100150`** transforme des alertes éparses en incident qualifié — c'est le cœur de la valeur du SOC.
- L'acquisition automatisée (`collect.sh`) avec manifeste de hashes garantit une **chaîne de preuve** exploitable.
- La reconstruction fiable de la timeline nécessite de croiser plusieurs sources (Syscheck, réseau, mémoire, logs), car les horodatages de fichiers sont manipulables.
