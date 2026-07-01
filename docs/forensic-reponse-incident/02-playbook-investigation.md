---
sidebar_position: 2
title: "Playbook d'investigation"
description: "Démarche concrète de collecte et d'analyse des preuves, étape par étape."
---

# Playbook d'investigation

Démarche concrète à dérouler face à un incident, organisée selon le cycle de réponse. À utiliser après avoir déclenché un incident sur le Cyber Range.

:::caution Règles forensiques
- Travailler sur des **copies**, jamais sur les preuves originales.
- **Hasher** chaque preuve dès sa collecte (intégrité).
- **Horodater** et **documenter** chaque action (chaîne de custody).
- Respecter l'ordre de volatilité : volatile d'abord, disque ensuite.
:::

## Préparation de l'investigation

```bash
# Espace de travail isolé pour l'enquête
mkdir -p ~/forensic_supersoc/{evidences,timeline,analysis}
cd ~/forensic_supersoc

# Registre de preuves (chaîne de custody)
cat > registre_preuves.md << 'EOF'
# Registre de preuves — Incident YYYY-MM-DD
| ID | Preuve | Source | Collectée le | Hash SHA256 | Analyste |
|----|--------|--------|--------------|-------------|----------|
EOF
```

## Phase 1 — Identification (point de départ)

L'incident commence par une alerte. On récupère l'événement déclencheur.

```bash
# Noter l'horodatage de référence de l'incident
echo "Incident identifié — $(date)" >> timeline/timeline.md
```

Côté plateforme :

1. Ouvrir le **dashboard Wazuh** → localiser l'alerte (règle déclenchée, niveau, horodatage).
2. Ouvrir le **SOAR** → le ticket associé (créé automatiquement si critique).
3. Relever les premiers indicateurs : IP source, machine ciblée, type d'attaque.

## Phase 2 — Collecte des preuves

### Depuis la plateforme (sources centralisées)

```bash
# Export des alertes Wazuh liées à l'incident (via l'API Wazuh)
curl -k -u "$WAZUH_USER:$WAZUH_PASS" \
  "https://WAZUH_IP:55000/security/events?q=srcip=ATTACKER_IP" \
  -o evidences/wazuh_alerts.json

# Hash immédiat de la preuve
sha256sum evidences/wazuh_alerts.json | tee evidences/wazuh_alerts.json.sha256
```

```sql
-- Journal d'audit lié à l'incident (depuis MariaDB, compte lecture seule)
SELECT action, resource_type, resource_id, ip_address, user_agent, timestamp
FROM Audit
WHERE timestamp BETWEEN 'DEBUT_INCIDENT' AND 'FIN_INCIDENT'
ORDER BY timestamp;
```

```javascript
// Logs d'exécution des playbooks (depuis MongoDB)
db.playbook_execution_logs.find({
  "started_at": { $gte: ISODate("DEBUT"), $lte: ISODate("FIN") }
}).sort({ started_at: 1 })

// Enrichissements de l'IP attaquante
db.ioc_enrichments.find({ "indicator": "ATTACKER_IP" })
```

### Depuis la machine compromise (Cyber Range)

En respectant l'ordre de volatilité :

```bash
# 1. Volatile : connexions réseau actives, processus
ss -tunap        > evidences/connexions.txt
ps auxf          > evidences/processus.txt
who              > evidences/sessions.txt

# 2. Logs d'authentification (tentatives SSH, brute force)
cp /var/log/auth.log evidences/auth.log
grep "Failed password" evidences/auth.log > analysis/ssh_bruteforce.txt

# 3. Hash de toutes les preuves collectées
cd evidences && sha256sum * > ../evidences_hashes.txt && cd ..
```

## Phase 3 — Reconstitution de la timeline

Le cœur de l'investigation : ordonner les événements de toutes les sources.

```bash
# Fusionner et trier les événements horodatés de toutes les sources
# (objectif : une ligne de temps unique et cohérente)
```

| Heure | Source | Événement | Preuve |
|-------|--------|-----------|--------|
| 14:30:12 | Wazuh | Alerte SQLi détectée (règle 31103) | wazuh_alerts.json |
| 14:30:13 | SOAR | Ticket #42 créé automatiquement | Audit |
| 14:30:14 | SOAR | IP enrichie (AbuseIPDB score 100) | ioc_enrichments |
| 14:30:15 | SOAR | Playbook blocage IP déclenché | playbook_execution_logs |
| 14:33:00 | Wazuh | Brute force SSH détecté | auth.log |

*(exemple de structure — à remplir avec les valeurs réelles)*

## Phase 4 — Analyse & identification

À partir de la timeline, répondre aux questions d'investigation :

- **Vecteur d'entrée** : par où l'attaque est-elle passée ? (ex. formulaire web vulnérable)
- **IoC** : IP source, hash de fichiers, comptes utilisés, user-agents.
- **Actions de l'attaquant** : qu'a-t-il tenté/réussi ?
- **Portée** : la segmentation a-t-elle contenu l'attaque ? A-t-il atteint d'autres VLAN ?

## Phase 5 — Confinement, éradication, récupération

```bash
# Confinement : vérifier que le playbook a bien bloqué l'IP (ou bloquer manuellement)
# Éradication : corriger la vulnérabilité exploitée (cf. rapport pentest)
# Récupération : restaurer depuis sauvegarde si données altérées
bash restore_mariadb.sh   # si nécessaire
```

## Phase 6 — Capitalisation

- Rédiger le **rapport d'investigation** (voir modèle).
- Identifier les **améliorations** : règle Wazuh à affiner, seuil à ajuster, playbook à enrichir.
- Mettre à jour le registre de preuves (chaîne de custody complète).

:::tip Démonstration de bout en bout
Si la timeline montre « détection → ticket → enrichissement → blocage » en quelques secondes, et que l'investigation reconstitue tout depuis la plateforme, tu démontres simultanément la détection (Wazuh), l'automatisation (SOAR) et l'investigation (forensic). Trois compétences en une seule narration.
:::
