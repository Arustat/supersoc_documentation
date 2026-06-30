---
sidebar_position: 3
title: "IaC — Terraform & Ansible"
description: "Provisioning Terraform et configuration Ansible de l'infrastructure."
---

# Infrastructure as Code

L'infrastructure est entièrement décrite en code, séparée en deux responsabilités complémentaires :

- **Terraform** provisionne les ressources (VM, réseau, VLAN) sur Proxmox.
- **Ansible** configure les VM et déploie les services (stack Docker).

## Terraform — Provisioning

Les fichiers Terraform décrivent le réseau et les machines virtuelles :

| Fichier | Rôle |
|---------|------|
| `network.tf` | Bridge `vmbr2` VLAN-aware, trunk SOC |
| `pfsense.tf` | VM pare-feu pfSense |
| `docker_host.tf` | VM Docker-Host (stack SOC) |
| `cyber_range.tf` | VM Cyber Range (cibles) |
| `cloud_init/` | Initialisation automatique des VM (clé SSH, réseau) |
| `variables.tf` | Paramètres (IP, stockage, node Proxmox) |

Les VM sont initialisées via **cloud-init** : injection de la clé SSH publique, configuration réseau statique (ex. Docker-Host en `10.0.20.10`, Cyber Range en `10.0.100.10`).

```hcl
resource "proxmox_network_linux_bridge" "vmbr2" {
  node_name  = var.proxmox_node
  name       = "vmbr2"
  address    = var.vmbr2_cidr
  comment    = "Internal bridge — SOC VLANs trunk (pfSense LAN)"
  vlan_aware = true
}
```

:::note Déploiement pfSense conditionnel
La variable `deploy_pfsense` (défaut `false`) permet de créer la VM pfSense séparément, via un job CI dédié, pour contourner un bug du provider lors de l'apply global. C'est un compromis assumé, documenté dans le code.
:::

## Ansible — Configuration

Le rôle `docker_host` prépare la VM et déploie les services. Points de sécurité notables :

- Arborescence dédiée `/opt/supersoc/` avec droits maîtrisés.
- Fichier `.env` généré depuis un template Jinja2, en **mode `0600`** (lecture propriétaire uniquement).
- Déploiement des stacks via `community.docker.docker_compose_v2`.

```yaml
- name: Create .env file from template
  ansible.builtin.template:
    src: supersoc.env.j2
    dest: /opt/supersoc/.env
    owner: ubuntu
    group: ubuntu
    mode: "0600"   # secrets lisibles par le seul propriétaire
```

Les rôles Ansible :

| Rôle | Rôle fonctionnel |
|------|------------------|
| `docker_host` | Installe Docker, déploie les stacks SOC |
| `docker` | Installation et durcissement du moteur Docker |
| `cyber_range` | Prépare la VM de simulation d'attaques |

## Chaîne complète

Le provisioning et la configuration sont orchestrés par la CI/CD (voir la section CI/CD & DevSecOps) : Terraform crée les VM, puis Ansible les configure et déploie les conteneurs.
