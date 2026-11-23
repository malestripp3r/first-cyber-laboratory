# Infrastructure Windows et supervision de sécurité

## 1.Objectif
Cette phase du projet consiste à mettre en place un environnement Windows centralisé autour d'Active Directory et à déployer une solution SIEM permettant de collecter et d'analyser les événements de sécurité.

L'objectif est de reproduire une infrastructure d'entreprise et de se familiariser avec l'administration des identités, la journalisation et la supervision des systèmes.

## 2. Active Directory
Un contrôleur de domaine Windows Server 2025, nommé DC01, a été déployé avec les services Active Directory Domain Services (AD DS) et DNS.

Le domaine corp.test a été créé afin de centraliser la gestion des utilisateurs, des groupes et des ordinateurs.

Plusieurs unités d'organisation (OU) ont été configurées pour structurer l'annuaire, notamment pour les utilisateurs, les postes de travail, les serveurs et les groupes de sécurité.

La machine WIN11-01 a ensuite été intégrée au domaine afin de tester l'authentification des utilisateurs et l'application des stratégies de groupe.
<img width="935" height="657" alt="image" src="https://github.com/user-attachments/assets/08605ef6-723d-4b8a-9297-be14f9667dd1" />


## 3.Sécurité et journalisation Windows
Des stratégies de groupe (GPO) ont été configurées pour renforcer la sécurité et améliorer la traçabilité des activités sur les postes Windows.

L'outil Sysmon a également été installé sur WIN11-01 afin de collecter des informations supplémentaires sur les activités du système, notamment l'exécution de processus et certaines connexions réseau.

## 4.Supervision avec Wazuh
Un serveur WAZUH01 a été déployé sous Ubuntu 24.04 LTS dans le réseau SOC.

Wazuh a été installé avec ses trois composants principaux : Manager, Indexer et Dashboard.

Un agent Wazuh a été configuré sur WIN11-01 pour transmettre les journaux Windows Security, Sysmon et PowerShell vers le SIEM.

Le Dashboard Wazuh permet de centraliser les événements, de consulter les alertes de sécurité et d'effectuer des recherches sur les activités des machines surveillées.
<img width="1006" height="426" alt="image" src="https://github.com/user-attachments/assets/e5b9e623-fae6-4400-bd1e-945bf2d06f0e" />


## 5.Tests et résultats
Plusieurs vérifications ont été réalisées afin de valider le fonctionnement de l'infrastructure.

L'intégration de WIN11-01 au domaine Active Directory, la résolution DNS ainsi que l'application des GPO ont été testées avec succès.

La communication entre l'agent Windows et le serveur Wazuh a également été validée. Les événements Windows Security, Sysmon et PowerShell sont visibles depuis l'interface Threat Hunting.

Ces résultats confirment le fonctionnement de la centralisation des journaux et constituent une première base pour la détection et l'analyse d'activités suspectes.
<img width="997" height="602" alt="image" src="https://github.com/user-attachments/assets/2dc08ec1-9c92-4b76-9153-b15ce55d42bf" />

<img width="997" height="542" alt="image" src="https://github.com/user-attachments/assets/5eb11368-14b8-43e2-94bd-ba05fa50db4b" />


