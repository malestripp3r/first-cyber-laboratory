# First Cyber Laboratory

Homelab de cybersécurité construit progressivement pour pratiquer l'administration système et réseau, Active Directory, la supervision SOC, la détection, l'investigation et les tests offensifs dans un environnement isolé.

## Objectifs

Ce laboratoire vise à reproduire une petite infrastructure d'entreprise afin de pratiquer :

- segmentation réseau et pare-feu avec OPNsense ;
- Windows Server et Active Directory ;
- DNS, utilisateurs, groupes, OU et GPO ;
- durcissement Windows et journalisation ;
- Sysmon et collecte de télémétrie ;
- SIEM/XDR avec Wazuh ;
- Linux ;
- tests offensifs depuis un réseau isolé ;
- détection, investigation et réponse à incident ;
- automatisation avec PowerShell, Bash, Python et Ansible.

## Architecture actuelle

```text
                         Internet
                            |
                       VirtualBox NAT
                            |
                         OPNsense
                            |
             +--------------+--------------+
             |                             |
           USERS                         SERVERS
      10.10.20.0/24                  10.10.30.0/24
             |                             |
         WIN11-01                        DC01
                                     10.10.30.10
                                          |
                                     corp.test

                              WAZUH01
                               10.10.30.30
```

## Plan d'adressage

| Hôte / interface | Adresse | Rôle |
|---|---:|---|
| OPNsense WAN | DHCP VirtualBox | Accès Internet |
| OPNsense USERS | 10.10.20.1/24 | Passerelle postes utilisateurs |
| OPNsense SERVERS | 10.10.30.1/24 | Passerelle serveurs |
| DC01 | 10.10.30.10/24 | AD DS + DNS |
| WIN11-01 | DHCP 10.10.20.0/24 | Poste utilisateur du domaine |
| WAZUH01 | 10.10.30.30/24 | SIEM Wazuh |

## Active Directory

- Domaine : `corp.test`
- Contrôleur de domaine : `DC01`
- DNS AD : `10.10.30.10`
- Poste joint au domaine : `WIN11-01`
- OU principales :
  - `CORP-Users`
  - `CORP-Computers`
  - `CORP-Groups`
  - `CORP-ServiceAccounts`

## Sécurité Windows déjà mise en place

- GPO `GPO-Workstations-Security-Baseline`
- verrouillage de session après inactivité ;
- audit avancé des connexions et créations de processus ;
- journalisation PowerShell ;
- Microsoft Defender actif ;
- Windows Firewall actif ;
- Sysmon installé et journalisation des événements principaux.

## État du projet

- [x] VirtualBox
- [x] OPNsense
- [x] Réseau USERS
- [x] Réseau SERVERS
- [x] Windows Server / DC01
- [x] Active Directory `corp.test`
- [x] DNS Active Directory
- [x] OU, utilisateurs et groupes
- [x] WIN11-01 joint au domaine
- [x] GPO de sécurité
- [x] Audit Windows
- [x] Sysmon
- [x] Wazuh
- [ ] Agent Wazuh sur Windows
- [ ] Centralisation Security / PowerShell / Sysmon
- [ ] Kali Linux
- [ ] Réseau ATTACK
- [ ] DMZ
- [ ] Machines et applications vulnérables
- [ ] Règles de détection personnalisées
- [ ] Exercices d'incident response
- [ ] Automatisation

## Structure du dépôt

```text
first-cyber-laboratory/
├── README.md
├── architecture/
├── opnsense/
├── active-directory/
├── windows/
├── wazuh/
├── detection/
├── attacks/
└── incident-response/
```

> Aucun mot de passe, secret, clé privée ou credential réel ne doit être ajouté au dépôt.
