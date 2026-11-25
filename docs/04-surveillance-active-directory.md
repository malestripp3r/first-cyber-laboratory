
# Surveillance Active Directory et corrélation Kerberos

## 1. Objectif et environnement

Cette phase du homelab vise à surveiller les modifications d'Active Directory et les authentifications Kerberos au moyen de **Windows Security** et du SIEM **Wazuh**.


| Élément | Configuration |
|---|---|
| Contrôleur de domaine | DC01 — Windows Server 2025 — 10.10.20.10 |
| Domaine Active Directory | corp.test (CORP) |
| Poste client | WIN11-01 — 10.10.10.130 |
| Serveur SIEM | WAZUH01 — 10.10.30.10 |
| Collecte | Agent Wazuh sur DC01, journal Windows Security |

Une GPO dédiée, `GPO-DC-Auditing`, a été créée et liée à l'OU **Domain Controllers**.

Elle active l'audit **Success and Failure** pour les sous-catégories suivantes :

- User Account Management
- Security Group Management
- Kerberos Authentication Service
- Kerberos Service Ticket Operations

Les paramètres ont été vérifiés avec `auditpol` sur DC01.

## 2. Scénario 4 — Création d'un utilisateur Active Directory

### Objectif

Détecter la création d'un nouveau compte utilisateur dans le domaine.

### Simulation

Un utilisateur de test désactivé, `soc.lab.test`, a été créé depuis DC01 avec la commande PowerShell `New-ADUser`.

Cette action a généré l'événement Windows Security **4720** (*A user account was created*).

### Détection

| Élément | Observation |
|---|---|
| Auteur | CORP\Administrator |
| Compte créé | soc.lab.test |
| Event ID | 4720 |
| Règle Wazuh | 60109 |
| Description | User account enabled or created |
| Niveau | 8 |

<img width="947" height="537" alt="image" src="https://github.com/user-attachments/assets/41d10e55-1194-442d-bbfa-176a52514ae6" />

### Analyse et conclusion

La détection permet d'identifier le compte créé ainsi que l'administrateur à l'origine de l'opération.

Une création de compte inattendue pourrait représenter une activité suspecte nécessitant une investigation.


**Résultat : détection validée.**

## 3. Scénario 5 — Ajout à un groupe de sécurité

### Objectif

Surveiller les modifications d'appartenance à un groupe Active Directory.

### Simulation

Le compte `soc.lab.test` a été ajouté au groupe global de sécurité `GG_IT` :

```powershell
Add-ADGroupMember -Identity "GG_IT" -Members "soc.lab.test"
```

### Détection

| Élément | Observation |
|---|---|
| Groupe | GG_IT |
| Membre ajouté | soc.lab.test |
| Auteur | CORP\Administrator |
| Event ID | 4728 |
| Règle Wazuh | 60141 |
| Niveau | 5 |

<img width="947" height="372" alt="image" src="https://github.com/user-attachments/assets/ceab266e-bd01-4d96-bb21-4b9f0a5e9703" />


### Analyse et conclusion

L'événement permet de retrouver le groupe modifié, le compte ajouté et l'utilisateur ayant effectué l'opération.

Cette surveillance est importante, car l'ajout d'un utilisateur à certains groupes peut lui accorder de nouvelles permissions.

**Résultat : détection validée.**

## 4. Scénario 6 — Retrait d'un groupe de sécurité

### Objectif

Détecter la suppression d'un membre d'un groupe Active Directory.

### Simulation

Le compte de test a été retiré du groupe `GG_IT` :

```powershell
Remove-ADGroupMember -Identity "GG_IT" -Members "soc.lab.test" -Confirm:$false
```

### Détection

| Élément | Observation |
|---|---|
| Groupe | GG_IT |
| Membre retiré | soc.lab.test |
| Auteur | CORP\Administrator |
| Event ID | 4729 |
| Règle Wazuh | 60142 |
| Niveau | 5 |

<img width="952" height="552" alt="image" src="https://github.com/user-attachments/assets/600d56a9-6aac-4a4a-8774-3e3245132492" />

### Analyse et conclusion

L'événement Windows `4729` permet de suivre les modifications des groupes de sécurité, y compris lorsqu'un utilisateur perd son appartenance à un groupe.

**Résultat : détection validée.**

## 5. Scénario 7 — Échec de préauthentification Kerberos

### Objectif

Détecter une tentative d'authentification Kerberos avec un mot de passe incorrect et identifier son origine réseau.

### Simulation

Un compte de test dédié, `soc.kerb.test`, a été créé dans Active Directory.

Depuis WIN11-01, une tentative d'authentification avec un mauvais mot de passe a été réalisée :

```powershell
runas /user:soc.kerb.test@corp.test cmd
```

### Détection

Le contrôleur de domaine a généré l'événement **4771** (*Kerberos pre-authentication failed*).

| Élément | Observation |
|---|---|
| Compte | soc.kerb.test |
| Event ID | 4771 |
| Code d'échec | 0x18 |
| Adresse source | ::ffff:10.10.10.130 |
| Règle Wazuh | 60104 |
| Niveau | 5 |

<img width="936" height="610" alt="image" src="https://github.com/user-attachments/assets/e0783c0c-1214-46db-9df1-694fb0d84bcb" />

<img width="967" height="562" alt="image" src="https://github.com/user-attachments/assets/8cf54947-4263-4f4d-a09d-3dac34f3833f" />


### Analyse et conclusion

Le code `0x18` indique un échec de préauthentification Kerberos lié à des informations d'authentification incorrectes.

L'adresse `::ffff:10.10.10.130` est une représentation IPv4-mappée IPv6 correspondant à WIN11-01.

L'analyse de l'événement permet donc d'identifier le compte ciblé ainsi que le poste à l'origine de la tentative.

**Résultat : détection validée.**

## 6. Scénario 8 — Authentification Kerberos réussie

### Objectif

Comparer une authentification Kerberos réussie avec les échecs précédemment détectés.

### Simulation

Depuis WIN11-01, la commande suivante a été exécutée avec le bon mot de passe :

```powershell
runas /user:soc.kerb.test@corp.test cmd
```

L'identité du compte utilisé a été vérifiée avec :

```cmd
whoami
```

Résultat :

```text
corp\soc.kerb.test
```

Le contrôleur de domaine a enregistré un événement **4768** correspondant à une demande de ticket Kerberos TGT réussie.

### Détection

L'événement ne générait pas d'alerte visible par défaut dans Threat Hunting.

Une règle Wazuh personnalisée a donc été créée pour surveiller les demandes de tickets réussies concernant notre compte de test.

| Élément | Observation |
|---|---|
| Event ID | 4768 |
| Compte | soc.kerb.test |
| Code de résultat | 0x0 |
| Adresse source | 10.10.10.130 |
| Règle personnalisée | 100120 |
| Niveau | 3 |

<img width="957" height="755" alt="image" src="https://github.com/user-attachments/assets/a1a4b166-0894-45f7-86f6-ec066c23490b" />


### Analyse et conclusion

Ce scénario a permis de comprendre la différence entre un événement journalisé et une alerte générée par Wazuh.

La règle personnalisée permet de rendre visible une authentification réussie pour un compte spécifique.

Le niveau 3 a été choisi pour identifier une activité informative, sans la considérer automatiquement comme suspecte.

**Résultat : règle personnalisée validée.**

## 7. Scénario 9 — Corrélation Kerberos

### Objectif

Détecter plusieurs échecs d'authentification Kerberos rapprochés, suivis d'une authentification réussie.

L'objectif est de dépasser la détection d'événements isolés pour analyser une succession d'activités potentiellement suspectes.

### Simulation

Depuis WIN11-01, plusieurs tentatives d'authentification avec un mauvais mot de passe ont été réalisées sur le compte `soc.kerb.test`.

Une authentification réussie a ensuite été effectuée avec le même compte.

Avant les tests, la politique de verrouillage applicable au compte a été vérifiée afin d'éviter un blocage involontaire.

### Configuration des règles Wazuh

Plusieurs règles personnalisées ont été ajoutées via l'interface Web Wazuh.

| Règle | Fonction | Niveau |
|---|---|---|
| 100130 | Détection d'un échec Kerberos 4771 avec le code 0x18 | 5 |
| 100131 | Détection de trois échecs pour le même compte dans une fenêtre de 120 secondes | 10 |
| 100120 | Détection d'une demande de ticket TGT réussie | 3 |
| 100132 | Détection d'une réussite après une alerte de corrélation précédente, dans une fenêtre de 300 secondes | 11 |

Les règles ont été vérifiées avec :

```bash
sudo /var/ossec/bin/wazuh-analysisd -t
```

Le Wazuh Manager a ensuite été redémarré pour appliquer les modifications.

### Détection des échecs répétés

La règle `100131` utilise les mécanismes de corrélation temporelle de Wazuh, notamment `frequency`, `timeframe` et `same_field`.

Elle permet de détecter plusieurs échecs Kerberos rapprochés concernant le même compte utilisateur.

<img width="950" height="605" alt="image" src="https://github.com/user-attachments/assets/ce995109-ae41-4782-89c5-5eb5e18d9dfe" />

### Détection d'une réussite après les échecs

Une deuxième règle, `100132`, a été créée afin de détecter une authentification réussie après une alerte précédente de corrélation.

Lors du test final, Wazuh a généré les alertes suivantes :

| Règle | Résultat |
|---|---|
| 100130 | Échecs Kerberos individuels |
| 100131 | Plusieurs échecs rapprochés détectés |
| 100132 | Authentification réussie après les échecs |

<img width="957" height="150" alt="image" src="https://github.com/user-attachments/assets/4cb65d01-0a53-4739-b635-177c3b561893" />

Les détails de l'alerte `100132` confirment :

- `eventID : 4768`
- `targetUserName : soc.kerb.test`
- `status : 0x0`
- `ipAddress : ::ffff:10.10.10.130`
<img width="445" height="492" alt="image" src="https://github.com/user-attachments/assets/88a0ef23-1120-4671-babf-37127d314be5" />

<img width="475" height="315" alt="image" src="https://github.com/user-attachments/assets/614566c8-0744-4af5-bbb2-87e9892ed255" />

### Analyse et conclusion

Ce scénario démontre la possibilité de corréler plusieurs événements Windows dans Wazuh afin de détecter un comportement plus complexe qu'une simple authentification échouée.

La règle `100132` s'est déclenchée après plusieurs échecs Kerberos suivis d'une réussite.

Cette séquence pourrait représenter une activité suspecte dans un environnement professionnel.

**Résultat : corrélation validée dans les conditions du laboratoire.**

[Consulter les règles Wazuh Kerberos](../rules/kerberos-correlation.xml)

## 8. Bilan

Ces scénarios ont permis de mettre en place et de valider plusieurs mécanismes de surveillance :

- Détection des créations de comptes Active Directory.
- Surveillance des ajouts et retraits de membres de groupes.
- Détection des échecs d'authentification Kerberos.
- Surveillance des authentifications réussies.
- Création de règles personnalisées Wazuh.
- Corrélation temporelle de plusieurs événements de sécurité.
