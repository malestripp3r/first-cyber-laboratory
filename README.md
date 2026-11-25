# Cybersecurity Homelab

Ce projet consiste à construire progressivement un laboratoire de cybersécurité personnel reproduisant
une infrastructure d'entreprise.

L'objectif est de développer mes compétences pratiques
en administration système et réseau, en sécurité défensive, en détection des menaces et en investigation.


Ce dépôt GitHub servira à documenter les différentes étapes du projet, les configurations réalisées, les problèmes rencontrés et les solutions apportées.

## Objectifs

- Concevoir une infrastructure réseau segmentée et sécurisée.

- Déployer un environnement Windows Server et Active Directory.

- Mettre en place un SIEM pour centraliser et analyser les événements de sécurité.

- Simuler des attaques dans un environnement isolé.

- Détecter et analyser des activités suspectes.

- Apprendre à répondre à des incidents de sécurité.

- Automatiser certaines tâches avec des scripts.

## Technologies utilisées

- Virtualisation : VirtualBox
- Pare-feu et segmentation : OPNsense
- Systèmes : Windows Server 2025, Windows 11, Ubuntu 24.04 LTS
- Services : Active Directory Domain Services, DNS, GPO
- Journalisation : Windows Event Logs, Sysmon, PowerShell Script Block Logging
- SIEM : Wazuh (Manager, Indexer, Dashboard et agent Windows)


## Technologies et travaux prévus

- Kali Linux pour les simulations d'attaques
- Suricata pour la détection réseau
- Détection avancée des menaces
- Investigation et réponse aux incidents
- Automatisation avec Python, PowerShell et Bash
<img width="2880" height="1860" alt="architecture-homelab-sans-legende" src="https://github.com/user-attachments/assets/b9859ffe-07ac-4db9-a92f-000cb0f2932c" />


## Documentation

- [Architecture réseau](docs/01-architecture-reseau.md)
- [Infrastructure Windows et supervision](docs/02-infrastructure-et-supervision.md)
- [Scénarios de détection SOC](docs/03-scenarios-detection.md)
- [Surveillance Active Directory et Kerberos](docs/04-surveillance-active-directory.md)

