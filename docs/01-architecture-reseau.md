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


## 5. Machines virtuelles
| Machine | Système d'exploitation | Adresse IP | Rôle |
|---|---|---|---|
| FW01 | OPNsense 26.7 | 10.10.10.1 / 10.10.20.1 / 10.10.30.1 | Pare-feu et routage inter-réseaux |
| DC01 | Windows Server 2025 | 10.10.20.10 | Contrôleur de domaine AD DS et DNS |
| WIN11-01 | Windows 11 | 10.10.10.130 | Poste utilisateur du domaine |
| WAZUH01 | Ubuntu 24.04 LTS | 10.10.30.10 | SIEM Wazuh : Manager, Indexer et Dashboard |

## 6.Communication et filtrage
Le pare-feu OPNsense assure le routage entre les différents réseaux du laboratoire. Les communications sont limitées à l'aide de règles de filtrage afin d'éviter qu'une machine puisse accéder librement à toutes les zones de l'infrastructure.

## 7.Tests et résultats
Plusieurs tests ont été réalisés afin de vérifier le bon fonctionnement de l'infrastructure réseau.

Les tests de connectivité ont permis de confirmer la communication entre les machines et leurs passerelles. Les règles du pare-feu OPNsense ont également été vérifiées afin de s'assurer que les communications autorisées fonctionnent et que les accès non autorisés sont bloqués.

La résolution DNS, l'intégration de Windows 11 au domaine Active Directory ainsi que l'accès au Dashboard Wazuh ont été validés.


