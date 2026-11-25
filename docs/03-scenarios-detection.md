# Scénarios de détection SOC

## 1. Objectif

Cette phase du homelab consiste à simuler différentes activités sur le poste Windows 11 et à vérifier leur détection dans Wazuh.

L'objectif est de comprendre le fonctionnement de la collecte des événements, des règles de détection et de l'analyse des alertes dans un environnement SOC.

## 2. Scénario 1 — Échec d'authentification Windows

### Objectif

Simuler une tentative d'authentification avec un compte inexistant et vérifier que Wazuh détecte l'échec.

### Simulation

Depuis WIN11-01, une tentative d'authentification SMB locale a été réalisée avec la commande :

`net use \\127.0.0.1\IPC$ /user:WIN11-01\lab-test-inexistant *`

Un mot de passe incorrect a été saisi volontairement.

Windows a refusé la connexion avec l'erreur système `1326`, correspondant à des identifiants incorrects.

### Détection

L'événement Windows Security `4625` a été généré et transmis à Wazuh.

| Champ | Résultat |
|---|---|
| Machine | WIN11-01 |
| Event ID | 4625 |
| Compte ciblé | lab-test-inexistant |
| Logon Type | 3 — Network |
| Protocole | NTLM |
| Adresse source | 127.0.0.1 |
| Sous-code d'échec | 0xC0000064 |
| Règle Wazuh | 60122 |
| Niveau d'alerte | 5 |

La règle Wazuh `60122`, intitulée *Logon Failure - Unknown user or bad password*, a permis d'identifier cet échec d'authentification.

<img width="1080" height="542" alt="05-alertes-wazuh-regle-60122" src="https://github.com/user-attachments/assets/4ae9bbab-896e-4ba8-b4ff-30c680818c1e" />

### Analyse et conclusion

L'analyse de l'alerte a permis d'identifier le compte ciblé, la méthode d'authentification et l'origine de la connexion.

Le sous-code `0xC0000064` indique que le compte utilisateur n'existe pas.

L'activité a été classée comme bénigne, puisqu'elle provenait d'une simulation volontaire.

**Résultat : détection validée.**

## 3. Scénario 2 — Surveillance d'une exécution PowerShell

### Objectif

Vérifier que Sysmon et Wazuh permettent de détecter l'exécution d'un processus PowerShell et d'analyser sa ligne de commande.

### Simulation

La commande suivante a été exécutée sur WIN11-01 :

`powershell.exe -NoProfile -Command "Write-Output 'SOC-LAB-TEST'"`

Cette commande démarre une nouvelle instance PowerShell et affiche un message de test.

### Détection

Sysmon a généré un événement `1` (*Process Create*), qui a été transmis à Wazuh.

| Champ | Résultat |
|---|---|
| Machine | WIN11-01 |
| Source | Sysmon |
| Event ID | 1 |
| Processus | powershell.exe |
| Processus parent | powershell.exe |
| Règle Wazuh | 92027 |
| Niveau d'alerte | 4 |
| MITRE ATT&CK | T1059.001 — PowerShell |

La règle `92027`, intitulée *Powershell process spawned powershell instance*, a détecté la création d'une instance PowerShell depuis un autre processus PowerShell.
<img width="952" height="522" alt="image" src="https://github.com/user-attachments/assets/dc38ea2f-61d0-43b4-b575-cca88743432b" />

L'analyse détaillée de l'événement permet de retrouver la ligne de commande exécutée.
<img width="512" height="557" alt="image" src="https://github.com/user-attachments/assets/a7cb6291-b166-4437-bce4-669a56070b6e" />


### Analyse et conclusion

L'événement Sysmon contient notamment le processus exécuté, son processus parent, son utilisateur et sa ligne de commande complète.

Ces informations sont utiles pour comprendre le contexte d'exécution d'un programme.

L'activité était légitime dans le cadre du laboratoire, mais elle a permis de valider la surveillance des créations de processus.

**Résultat : détection validée.**

## 4. Scénario 3 — Règle Wazuh personnalisée

### Objectif

Créer une règle personnalisée capable de détecter un marqueur spécifique dans les scripts PowerShell.

### Configuration

Une règle locale a été ajoutée au fichier `/var/ossec/etc/rules/local_rules.xml` sur WAZUH01.

La règle utilise l'identifiant `100110`, le niveau `5` et recherche la chaîne `SOC-LAB-CUSTOM-RULE` dans le contenu des blocs de script PowerShell.

La configuration a été vérifiée avec :

`sudo /var/ossec/bin/wazuh-analysisd -t`

Le Wazuh Manager a ensuite été redémarré pour charger la règle.

### Simulation

La commande de test suivante a été exécutée sur WIN11-01 :

`powershell.exe -NoProfile -Command "Write-Output 'SOC-LAB-CUSTOM-RULE'"`

### Détection

| Champ | Résultat |
|---|---|
| Machine | WIN11-01 |
| Source | PowerShell Script Block Logging |
| Event ID | 4104 |
| Règle Wazuh | 100110 |
| Niveau d'alerte | 5 |
| Description | Homelab - Custom PowerShell detection test |
| Marqueur recherché | SOC-LAB-CUSTOM-RULE |
<img width="1037" height="555" alt="02-alertes-wazuh-regle-100110" src="https://github.com/user-attachments/assets/f788748e-b594-40fd-92f1-a202e1c6edfa" />


Le champ `data.win.eventdata.scriptBlockText` contenait bien la chaîne recherchée.
<img width="487" height="315" alt="03-detail-scriptblocktext" src="https://github.com/user-attachments/assets/ee9529b5-78df-4192-9af8-c5f7d8790b65" />

### Analyse et conclusion

Ce scénario a permis de réaliser l'ensemble du processus de création d'une détection : définition d'une condition, configuration d'une règle, génération d'un événement de test et validation de l'alerte.

La commande utilisée est inoffensive. Le marqueur sert uniquement à vérifier le fonctionnement de la règle.

**Résultat : règle personnalisée opérationnelle.**

## 5. Bilan

Les trois scénarios ont permis de valider plusieurs fonctionnalités de l'infrastructure SOC :

- La détection des échecs d'authentification Windows.
- La surveillance des créations de processus avec Sysmon.
- La collecte des événements PowerShell.
- La création et le test de règles Wazuh personnalisées.

Ces premiers tests constituent une base pour réaliser des simulations plus avancées, améliorer les règles de détection et approfondir l'investigation des alertes.
