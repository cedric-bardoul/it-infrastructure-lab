# Donner accès à Internet à deux VM avec une passerelle, du NAT et du DNS

Séance du 16 novembre 2026. Manipulation réelle sous Hyper-V et Ubuntu.

## Problème et objectif

Les VM communiquaient sur leur réseau local. La table de routage de VM1 montrait uniquement la route vers `192.168.10.0/24`. La recherche de route vers `192.168.20.10` renvoyait `Network is unreachable`.

L’objectif était de fournir une sortie IPv4 par Windows avec NAT, puis de résoudre les noms.

## Architecture obtenue

| Équipement ou service | Configuration du lab |
|---|---|
| Commutateur Hyper-V | Type interne, nom affiché LAB-PRIVE |
| Carte Windows côté lab | vEthernet (LAB-PRIVE), ifIndex 33 |
| Adresse Windows côté lab | `192.168.10.1/24` |
| NAT Windows | NAT-LAB, réseau interne `192.168.10.0/24`, Active True |
| VM1 | `192.168.10.20/24` |
| VM2 | `192.168.10.22/24` |
| Passerelle des VM | `192.168.10.1` |
| DNS renseigné sur les VM | `1.1.1.1` |

Les VM envoient leurs communications externes à Windows par le commutateur interne. Windows réalise le NAT vers son réseau physique, puis la box fournit la sortie vers Internet. Le réseau physique et celui du lab ont été contrôlés comme distincts.

## Préparation Windows, réalisée avec aide

Le commutateur a été changé du type privé au type interne. Le nom LAB-PRIVE est resté affiché.

Contrôles de la configuration, des cartes et des NAT

```powershell
ipconfig
Get-NetNat
Get-NetAdapter
Get-NetIPAddress -InterfaceIndex 33 -AddressFamily IPv4
```

Le premier contrôle NAT ne retournait aucun objet. La carte 33 possédait initialement une adresse automatique en `169.254.x.x/16`.

Ouverture d’une fenêtre PowerShell administrateur, suivie de la validation de la demande Windows

```powershell
Start-Process powershell -Verb RunAs
```

Attribution de l’adresse côté lab, contrôle et création du NAT

```powershell
New-NetIPAddress -InterfaceIndex 33 -IPAddress 192.168.10.1 -PrefixLength 24
Get-NetIPAddress -InterfaceIndex 33 -AddressFamily IPv4
New-NetNat -Name NAT-LAB -InternalIPInterfaceAddressPrefix 192.168.10.0/24
```

Le numéro 33 est propre à cette configuration. Il a été identifié avant modification. L’état de l’adresse est passé de `Tentative` à `Preferred`. Le NAT a été affiché avec `Active: True`.

## Passerelle et tests IPv4

Dans les profils IPv4 Ubuntu, les adresses statiques ont été conservées et la passerelle `192.168.10.1` a été ajoutée. Après application, la connexion Filaire a été désactivée puis réactivée.

VM1 a été configurée avec aide. J’ai reproduit la configuration sur VM2 avec les indications précédentes disponibles et vérifié sa route avant le test.

```bash
ip route show
ping -4 -c 4 1.1.1.1
```

| Contrôle | Résultat |
|---|---|
| Route par défaut VM1 | Présence confirmée pendant la manipulation |
| Route par défaut VM2 | `default via 192.168.10.1 dev eth0 proto static metric 20100` |
| Ping de VM1 vers 1.1.1.1 | 4 envoyés, 4 reçus, aucune perte, résultat relevé pendant la manipulation |
| Ping de VM2 vers 1.1.1.1 | 4 envoyés, 4 reçus, 0 % de perte, résultat confirmé sur capture |

Ces tests ont établi que les deux VM recevaient des réponses de cette adresse Internet.

## Incident DNS et correction

Sur VM2, cette commande ne retournait initialement aucune adresse malgré le ping IPv4 réussi.

```bash
getent ahostsv4 example.com
```

Dans le profil IPv4, le DNS était en automatique, sans serveur saisi manuellement. Aucun DHCP n’avait été installé sur le réseau du lab.

J’ai renseigné `1.1.1.1` comme serveur DNS, puis appliqué et réactivé la connexion. L’interrupteur Automatique est resté actif dans l’interface. Le contrôle de la configuration effective a permis de vérifier le résultat.

```bash
resolvectl status
getent ahostsv4 example.com
```

Sur VM2, `Current DNS Server` et `DNS Servers` affichaient tous deux `1.1.1.1`. La résolution a retourné `172.66.147.243` et `104.20.23.154` pendant la séance. Ces réponses peuvent changer.

J’ai ensuite reproduit le réglage DNS sur VM1 et obtenu des adresses IPv4 avec getent. La configuration saisie et le résultat de résolution sont confirmés sur capture. Le contrôle resolvectl a été effectué sur VM2.

## Rôles à distinguer

| Élément | Rôle dans la manipulation |
|---|---|
| Adresse de la VM | Identifier l’interface de la machine sur le réseau |
| Passerelle 192.168.10.1 | Atteindre les destinations externes au réseau local via Windows |
| NAT Windows | Traduire les communications du réseau interne vers le réseau côté hôte |
| DNS 1.1.1.1 | Retrouver les adresses associées à un nom |
| Ping vers 1.1.1.1 | Tester une destination IP sans avoir à résoudre un nom |

## Bilan de pratique

- Configuration du réseau interne, de l’IP Windows et du NAT réalisée avec aide.
- Passerelle de VM1 configurée avec aide, puis reproduction et contrôle sur VM2 avec les notes disponibles.
- Diagnostic DNS guidé sur VM2, puis reproduction du réglage et test sur VM1.
- Vérification après redémarrage encore à effectuer.
- Le retour de quatre réponses à un ping et la résolution d’un nom sont les contrôles réalisés dans cette séance.

[Retour à la présentation](../README.md)
