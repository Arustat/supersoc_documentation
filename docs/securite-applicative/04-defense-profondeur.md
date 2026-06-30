---
sidebar_position: 4
title: "Défense en profondeur"
description: "Synthèse des couches de sécurité applicative et leurs complémentarités."
---

# Défense en profondeur

La sécurité de SuperSOC ne repose pas sur une mesure unique mais sur un **empilement de couches** complémentaires : si l'une cède, les autres limitent l'impact.

## Synthèse des couches

| Couche | Mesure | Classe de risque couverte |
|--------|--------|---------------------------|
| Réseau | VLAN, pfSense, VPN WireGuard | Propagation latérale, exposition |
| Conteneur | `no-new-privileges`, réseau `internal` | Élévation de privilèges, exposition DB |
| Authentification | JWT access/refresh, argon2id | Vol d'identifiants, force brute |
| Autorisation | RBAC par dépendances | Accès non autorisé |
| Données | Comptes au moindre privilège (DCL) | Impact d'une compromission applicative |
| Traçabilité | Audit append-only | Répudiation, effacement de traces |
| Frontend | Token en mémoire (anti-XSS) | Vol de token via XSS |

## Token en mémoire (anti-XSS)

Côté frontend Vue.js, le token JWT est conservé **en mémoire** plutôt que dans le `localStorage`. Le `localStorage` est lisible par n'importe quel script de la page ; en cas de faille XSS, un token qui y serait stocké serait exfiltrable. Le garder en mémoire réduit cette surface (au prix d'une reconnexion au rechargement, géré par le refresh token).

## Principe directeur

Chaque couche attrape une **classe de problème différente**. Aucune n'est suffisante seule, mais leur combinaison rend une compromission complète nettement plus coûteuse pour un attaquant.

:::tip Phrase de synthèse pour l'oral
« Nous n'avons pas une sécurité, nous avons des sécurités en couches : réseau, conteneur, authentification, autorisation, données et traçabilité. Chacune limite l'impact d'une défaillance des autres. »
:::
