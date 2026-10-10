# Wazuh - Installation du serveur

## Rôle

La VM `WAZUH01` héberge les composants centraux Wazuh pour le laboratoire :

- Wazuh Manager ;
- Wazuh Indexer ;
- Wazuh Dashboard.

## Configuration de la VM

| Paramètre | Valeur |
|---|---|
| Hostname | `wazuh01` |
| OS | Ubuntu Server 24.04 LTS |
| vCPU | 4 |
| RAM | 8 Go |
| Disque | 70 Go dynamique |
| Réseau VirtualBox | `LAB-SERVERS` |
| Adresse IPv4 | `10.10.30.30/24` |
| Passerelle | `10.10.30.1` |
| DNS | `10.10.30.10` |

## Préparation système

La VM a été configurée avec une adresse statique, le DNS Active Directory et le fuseau horaire adapté au laboratoire.

Vérifications réseau principales :

```bash
ip -br addr
ip route
resolvectl status
ping -c 4 10.10.30.1
ping -c 4 10.10.30.10
ping -c 4 8.8.8.8
ping -c 4 google.com
```

Le système a ensuite été mis à jour et les outils nécessaires installés.

## Installation Wazuh

Le déploiement choisi est un déploiement all-in-one, adapté à un homelab.

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

Les services principaux ont ensuite été vérifiés :

```bash
sudo systemctl status wazuh-manager --no-pager
sudo systemctl status wazuh-indexer --no-pager
sudo systemctl status wazuh-dashboard --no-pager
```

## Pare-feu local

UFW est utilisé pour limiter les accès entrants. Les flux autorisés concernent principalement :

- SSH depuis le réseau d'administration utilisé dans le lab ;
- HTTPS vers le dashboard ;
- communication et enrôlement des agents Wazuh.

Les ports internes tels que celui de l'Indexer ne sont pas exposés inutilement aux postes utilisateurs.

## DNS

Une entrée DNS `wazuh01.corp.test` pointe vers `10.10.30.30`.

## Validation

Le dashboard est accessible depuis `WIN11-01` via HTTPS et les composants centraux sont opérationnels.

## Prochaine étape

Déployer le Wazuh Agent sur `WIN11-01`, puis centraliser :

- Windows Security ;
- PowerShell Operational ;
- Sysmon Operational ;
- Microsoft Defender Operational.

> Les mots de passe, certificats privés et secrets générés par Wazuh ne sont jamais versionnés dans ce dépôt.
