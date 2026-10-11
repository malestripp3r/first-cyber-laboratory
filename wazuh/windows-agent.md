# Wazuh - Agent Windows et centralisation des journaux

## Hôte supervisé

| Paramètre | Valeur |
|---|---|
| Hôte | `WIN11-01` |
| Domaine | `corp.test` |
| Réseau | `USERS - 10.10.20.0/24` |
| Serveur Wazuh | `10.10.30.30` |

## Agent Wazuh

Le Wazuh Agent est installé sur `WIN11-01` et apparaît comme **Active** dans le Dashboard.

Les communications principales avec le serveur Wazuh utilisent les ports prévus pour l'agent et son enrôlement.

## Journaux centralisés

L'agent collecte les sources utiles au SOC :

- Windows Security ;
- Sysmon Operational ;
- PowerShell Operational ;
- Microsoft Defender Operational.

Exemple de blocs `localfile` utilisés dans `ossec.conf` :

```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>

<localfile>
  <location>Microsoft-Windows-PowerShell/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>

<localfile>
  <location>Microsoft-Windows-Windows Defender/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

Le journal Windows Security est déjà pris en charge par la configuration Windows standard de l'agent et n'est pas dupliqué.

## Validation

Des événements ont été volontairement générés depuis `WIN11-01` :

- création de processus ;
- requêtes DNS ;
- connexions réseau ;
- commandes PowerShell ;
- opérations sur des comptes locaux.

Les événements et alertes correspondants sont visibles dans Wazuh Threat Hunting en filtrant sur l'agent `WIN11-01`.

## Pipeline SOC

```text
WIN11-01
   |
   +-- Windows Security
   +-- Sysmon
   +-- PowerShell
   +-- Defender
   |
Wazuh Agent
   |
WAZUH01
   |
Manager -> Indexer -> Dashboard
```

## Prochaine étape

- réaliser une première investigation SOC guidée ;
- créer une règle de détection personnalisée ;
- documenter l'alerte et les éléments observés ;
- préparer ensuite un réseau ATTACK avec Kali Linux.

> Aucun mot de passe ou secret n'est versionné dans le dépôt.
