---
sidebar_position: 3
title: "Modèle de rapport d'investigation"
description: "Gabarit de rapport d'investigation forensique post-incident."
---

# Modèle de rapport d'investigation

Gabarit à remplir au terme de l'investigation. Un bon rapport est compréhensible par un non-technicien (résumé) tout en restant rigoureux (détails techniques, preuves).

---

## Rapport d'investigation — Incident [ID]

| Champ | Valeur |
|-------|--------|
| **Identifiant incident** | INC-AAAA-NN |
| **Date de l'incident** | JJ/MM/AAAA |
| **Date de l'investigation** | JJ/MM/AAAA |
| **Analyste** | Nom |
| **Sévérité** | Critique / Élevée / Moyenne / Faible |
| **Statut** | En cours / Clos |

## 1. Résumé exécutif

Trois à cinq phrases compréhensibles par tous : que s'est-il passé, quel impact, comment cela a été géré. Pas de jargon.

> Exemple : « Le JJ/MM à 14h30, une tentative d'injection SQL a été détectée sur une machine du Cyber Range. La plateforme a automatiquement créé un ticket, identifié l'IP attaquante comme malveillante et déclenché son blocage. L'attaque n'a pas atteint d'autre zone du réseau. Aucune donnée de production n'a été compromise. »

## 2. Chronologie de l'incident (timeline)

| Heure | Événement | Source / Preuve |
|-------|-----------|-----------------|
| HH:MM:SS | Premier événement détecté | |
| HH:MM:SS | ... | |

## 3. Vecteur d'attaque

Comment l'attaquant est entré ou a tenté d'entrer. Vulnérabilité exploitée, point d'entrée.

## 4. Indicateurs de compromission (IoC)

| Type | Valeur | Source |
|------|--------|--------|
| IP source | | Wazuh |
| Réputation | Score AbuseIPDB / VirusTotal | Enrichissement |
| Compte ciblé | | auth.log |
| Hash / fichier | | |

## 5. Périmètre & impact

- Systèmes touchés :
- Données concernées (confidentialité / intégrité / disponibilité) :
- La segmentation a-t-elle contenu l'attaque ? :
- Impact réel évalué :

## 6. Réponse apportée

| Phase | Action | Automatique / Manuelle |
|-------|--------|------------------------|
| Confinement | Blocage IP | Playbook SOAR (auto) |
| Éradication | Correction vulnérabilité | Manuelle |
| Récupération | Restauration / vérification | |

## 7. Preuves collectées (chaîne de custody)

| ID | Preuve | Hash SHA256 | Collectée le |
|----|--------|-------------|--------------|
| P-01 | wazuh_alerts.json | | |
| P-02 | auth.log | | |
| P-03 | extrait journal Audit | | |

## 8. Recommandations

- Correctifs techniques (issus du finding pentest correspondant).
- Améliorations de détection (règle Wazuh, seuil de criticité).
- Améliorations de réponse (playbook, notification).

## 9. Leçons apprises

Ce que l'incident révèle sur la posture de sécurité, et ce qui a bien fonctionné (à valoriser autant que les axes d'amélioration).

:::note Boucle d'amélioration
La dernière phase du cycle de réponse alimente la première : les leçons d'un incident améliorent la préparation au suivant. Mentionner cette boucle montre une compréhension mature de la réponse à incident.
:::
