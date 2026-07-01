---
sidebar_position: 7
title: "Campagne Red Team — Scénario RAT pupy"
description: "Campagne Red Team phishing → RAT pupy → C2 → persistance sur la Cyber Range SuperSOC, mappée MITRE ATT&CK."
---

# Campagne Red Team — Scénario RAT pupy

Cette campagne simule une intrusion réaliste sur la Cyber Range : compromission initiale par phishing, déploiement d'un RAT (pupy), établissement d'un canal de commande et contrôle (C2) chiffré, puis persistance. Elle vise à valider la chaîne de détection Wazuh et à produire des preuves exploitables en investigation.

## 1. Objectifs

- Simuler une chaîne d'attaque complète et réaliste (pas seulement des attaques web isolées).
- Valider les règles de détection personnalisées développées pour le projet (`100100`–`100150`).
- Produire un jeu de preuves pour l'investigation forensic.

## 2. Chaîne d'attaque (MITRE ATT&CK)

| Étape | Technique MITRE | Action |
|-------|-----------------|--------|
| Livraison | T1566 — Phishing | Email piégé via GoPhish (Mailpit interne) |
| Exécution | T1204 — User Execution | La victime ouvre la pièce jointe / le lien |
| Implant | — | Déploiement du RAT pupy |
| C2 | T1071 — Application Layer Protocol | Canal C2 chiffré vers le serveur d'attaque |
| Persistance | T1547 / T1543 — Boot/Logon Autostart, systemd | Drop-in systemd, autostart XDG, rc.local |
| Exfiltration | T1041 — Exfiltration Over C2 | Simulée via le canal C2 |

## 3. Environnement (Cyber Range)

| Composant | Rôle |
|-----------|------|
| GoPhish + Mailpit | Infrastructure de phishing interne |
| Serveur C2 pupy | Commande et contrôle |
| cr-victim-01 / 02 | Postes Linux avec agent Wazuh |
| Wazuh | Détection (règles custom pupy) |

:::caution Isolation
Le VLAN 100 (Cyber Range) applique un `deny-all` sortant : aucun egress Internet, pas de propagation hors du périmètre de test. L'infrastructure de phishing est **interne** (Mailpit), aucun email réel n'est émis.
:::

## 4. Déroulement

### Phase 1 — Livraison (phishing)
Un email piégé est envoyé via GoPhish à la victime. 

> 📷 Capture — Campagne GoPhish

### Phase 2 — Exécution & implant
La victime déclenche la charge, le RAT pupy s'exécute et établit le canal C2.

Artefacts déposés :
```text
/usr/bin/dcrond          (binaire malveillant masqué)
/lib/libxstat.so.1       (shared object masqué)
~/.config/autostart/dbus-update.desktop  (persistance)
```

### Phase 3 — C2 & persistance
Le canal C2 chiffré est établi. La persistance est installée (drop-in systemd / autostart / rc.local).

> 📷 Capture — Session C2 pupy active

## 5. Détection Wazuh — règles déclenchées

Les règles personnalisées du projet détectent les IOC pupy à chaque étape :

| Règle | Niveau | Détecte |
|-------|--------|---------|
| `100100` | 12 | Dépôt d'un binaire malveillant connu (dcrond, libflushplugin.so...) |
| `100101` | 12 | Shared object masqué dans un répertoire système (`lib{RAND}.so.1`) |
| `100102` | 10 | Fichier déposé dans un répertoire utilisateur de masquage |
| `100110` | 12 | Drop-in systemd suspect (persistance) |
| `100111` | 12 | Entrée XDG autostart suspecte |
| `100112` | 12 | Modification d'un script d'init (rc.local) |
| `100120` | 10 | Processus masqué (`compiz`) détecté |
| `100150` | **14** | **INCIDENT — corrélation de plusieurs IOC pupy (RAT actif)** |

La règle `100150` est la clé : elle **corréle** plusieurs IOC de même source dans une fenêtre de 600 s pour élever un incident de niveau 14. C'est la différence entre une alerte isolée et un incident qualifié.

> 📷 Capture — Alerte incident `100150` dans le dashboard Wazuh

## 6. Traces attendues par cible

| Cible | Événements |
|-------|------------|
| cr-victim-01 | Exécution pupy, C2, persistance systemd, IOC fichiers |
| cr-victim-02 | Idem (seconde victime) |
| Wazuh | Règles `100100`–`100112` puis corrélation `100150` |

## 7. Conclusion

Cette campagne reproduit une intrusion complète et moderne (RAT + C2 + persistance), bien plus représentative qu'une simple attaque web. Elle valide directement les règles de détection développées pour le projet et alimente l'investigation forensic (voir le rapport d'investigation associé).
