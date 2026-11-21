# Architecture réseau

## 1. Objectif
Cette première phase consiste à concevoir une infrastructure réseau virtuelle reproduisant l'organisation d'une petite entreprise.

Elle sera divisée en plusieurs réseaux afin de séparer les postes utilisateurs, les serveurs et les outils de supervision de sécurité.

Un pare-feu OPNsense sera utilisé pour assurer le routage entre ces réseaux et contrôler les communications.

On pourra ainsi mettre en place differents scénarios de cybersécurité, comme la surveillance réseau, la détection d'activités suspectes et les simulations d'attaques.


## 2. Environnement de virtualisation
Le laboratoire sera déployé sur mon ordinateur personnel utilisant VirtualBox.

## 3. Segmentation réseau
L'infrastructure sera organisée en plusieurs zones réseau ayant chacune une fonction spécifique.

USERS : réseau destiné aux postes utilisateurs, notamment les machines Windows 11 intégrées au domaine Active Directory.

SERVERS : réseau hébergeant les serveurs de l'entreprise, notamment le contrôleur de domaine Active Directory et les services DNS.

SOC : réseau réservé aux outils de sécurité, notamment le serveur Wazuh chargé de centraliser et d'analyser les événements.

DMZ : réseau prévu pour héberger des serveurs de test accessibles selon des règles de filtrage spécifiques.

ATTACK : réseau réservé à Kali Linux et aux simulations d'attaques.

Le pare-feu OPNsense assurera la communication contrôlée entre les différents réseaux.

## 4.Plan d'adressage IP
Le plan d'adressage suivant a été défini pour l'infrastructure :
| Réseau | Sous-réseau | Passerelle prévue |
|---|---|---|
| USERS | 10.10.10.0/24 | 10.10.10.1 |
| SERVERS | 10.10.20.0/24 | 10.10.20.1 |
| SOC | 10.10.30.0/24 | 10.10.30.1 |
| DMZ | 10.10.40.0/24 | 10.10.40.1 |
| ATTACK | 10.10.50.0/24 | 10.10.50.1 |


