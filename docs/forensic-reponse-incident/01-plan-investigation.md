---
sidebar_position: 1
title: "Plan d'investigation forensique"
description: "Cadrage de la démarche d'investigation numérique post-incident sur SuperSOC (méthodologie, principes, cycle de réponse)."
---

# Plan d'investigation forensique — SuperSOC

Ce document cadre la démarche d'investigation numérique menée à la suite d'un incident de sécurité simulé sur la plateforme. Il définit la méthodologie, les principes de préservation de la preuve et le cycle de réponse, **avant** toute investigation réelle.

## 1. Objectif

Démontrer la capacité de la plateforme à **détecter, qualifier et investiguer** un incident de sécurité de bout en bout : de l'alerte initiale à la reconstitution de la chronologie de l'attaque, jusqu'aux recommandations.

L'investigation s'appuie sur un **incident simulé** sur le Cyber Range (typiquement une attaque menée lors de la campagne de pentest), ce qui crée un fil continu entre l'audit offensif et la réponse défensive.

:::note Articulation pentest ↔ forensic
Le pentest produit l'attaque ; le forensic l'investigue. Enquêter sur une attaque dont on connaît déjà le déroulé permet de **valider** que la plateforme reconstitue correctement les faits — c'est la preuve que la chaîne de détection et de traçabilité fonctionne.
:::

## 2. Méthodologie : le cycle de réponse à incident (NIST / SANS)

L'investigation suit le cycle de réponse à incident en 6 phases, modèle de référence (NIST SP 800-61, SANS Incident Handling).

| Phase | Objectif | Outils SuperSOC mobilisés |
|-------|----------|---------------------------|
| 1. Préparation | Disposer des moyens de détecter et tracer | Wazuh, audit append-only, SOAR |
| 2. Identification | Détecter et qualifier l'incident | Alertes Wazuh, ticket SOAR |
| 3. Confinement | Limiter la propagation | Playbook (blocage IP pfSense), segmentation VLAN |
| 4. Éradication | Supprimer la cause | Correction de la vulnérabilité exploitée |
| 5. Récupération | Restaurer un état sain | Sauvegardes, redéploiement IaC |
| 6. Leçons apprises | Capitaliser | Rapport, ajustement des règles de détection |

## 3. Principes forensiques

L'investigation respecte les principes fondamentaux de la preuve numérique.

### Préservation de la preuve

Une preuve altérée n'a aucune valeur. On travaille **sur des copies**, jamais sur les données originales, et on vérifie l'intégrité par empreinte (hash).

```bash
# Calcul d'empreinte pour garantir l'intégrité d'une preuve collectée
sha256sum evidence.log > evidence.log.sha256
# Toute modification ultérieure changera le hash → preuve d'intégrité
```

### Ordre de volatilité (RFC 3227)

Collecter du plus volatile au moins volatile :

1. Mémoire vive, état des processus, connexions réseau actives
2. Sessions, tables de routage, ARP
3. Fichiers temporaires
4. Disque (systèmes de fichiers)
5. Logs distants, configuration, archives

### Chaîne de custody (traçabilité de la preuve)

Documenter **qui** a collecté **quoi**, **quand**, et **comment** la preuve a été manipulée. Sans chaîne de custody, une preuve est contestable.

:::tip Atout structurel de SuperSOC
Le journal d'audit **append-only** de la plateforme est une chaîne de custody applicative native : il enregistre chaque action (acteur, IP, horodatage) sans possibilité de modification. C'est un argument fort pour la non-répudiation.
:::

## 4. Sources de preuves disponibles

La plateforme centralise plusieurs sources exploitables en investigation :

| Source | Contenu | Localisation |
|--------|---------|--------------|
| **Alertes Wazuh** | Détections horodatées, règles déclenchées | MongoDB (`wazuh_alerts`) + indexer |
| **Journal d'audit** | Actions sur le SOAR (acteur, IP, user-agent) | MariaDB (`Audit`, append-only) |
| **Logs d'exécution playbooks** | Réponses automatiques déclenchées | MongoDB (`playbook_execution_logs`) |
| **Enrichissements IoC** | Réputation des indicateurs | MongoDB (`ioc_enrichments`) |
| **Historique des tickets** | Cycle de vie de l'incident | MariaDB (`Ticket`, `Historique`) |
| **Logs système / agents** | Événements OS sur les machines | Agents Wazuh (Linux/Windows) |

## 5. Démarche d'investigation type

Face à une alerte, l'analyste suit une progression structurée :

1. **Point de départ** : une alerte Wazuh (ou un ticket SOAR créé automatiquement).
2. **Qualification** : l'incident est-il réel ? Quelle sévérité ? (réduction des faux positifs)
3. **Collecte** : rassembler les preuves liées (alertes corrélées, audit, logs agents), avec horodatage.
4. **Reconstitution de la timeline** : ordonner les événements pour comprendre le déroulé de l'attaque.
5. **Identification** : vecteur d'entrée, actions de l'attaquant, IoC (IP, hash, comptes).
6. **Évaluation de l'impact** : quelles données/systèmes touchés (confidentialité, intégrité, disponibilité).
7. **Réponse** : confinement, éradication, récupération.
8. **Capitalisation** : rapport et amélioration des règles de détection.

## 6. Scénario d'incident retenu

Scénario simulé sur le Cyber Range (à confirmer selon les tests réalisés) :

> Une machine du Cyber Range subit une **attaque web (injection SQL)** suivie d'une **tentative de brute force SSH**. L'objectif de l'investigation est de reconstituer la chronologie complète à partir des seules traces de la plateforme.

Questions auxquelles l'investigation doit répondre :

- Quand l'attaque a-t-elle commencé ? (premier événement détecté)
- Quel est le vecteur d'entrée ?
- Quelle adresse IP source ? Quelle réputation (enrichissement) ?
- La plateforme a-t-elle réagi automatiquement (ticket, playbook) ?
- Quel est l'impact réel ?

## 7. Livrables

- Ce plan d'investigation (cadrage).
- Une **timeline reconstituée** de l'incident (chronologie horodatée).
- Un **rapport d'investigation** (résumé, déroulé, IoC, impact, réponse, recommandations).
- Le **registre de preuves** (chaîne de custody : preuves collectées, hash, horodatage).

:::note Pour l'oral
La force de la démonstration : « Je n'ai pas eu besoin d'aller fouiller dix outils différents. Toute la chronologie de l'attaque était reconstituable depuis ma plateforme — alerte Wazuh, ticket SOAR, enrichissement, journal d'audit — car la traçabilité est intégrée à la conception. »
:::
