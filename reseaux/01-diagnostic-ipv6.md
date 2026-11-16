# Diagnostiquer une absence d’adresse IPv6 de lien local

Séance du 16 novembre 2026. Manipulation réelle dans un laboratoire Hyper-V.

## Contexte et objectif

Deux VM Ubuntu sont reliées au même commutateur privé Hyper-V. L’objectif est d’obtenir une adresse IPv6 de lien local sur chaque VM et de tester les échanges dans les deux sens.

## Problème observé

Sur VM1, cette commande revient à l’invite sans afficher d’adresse.

```bash
ip -6 address show scope link
```

La carte `eth0` affiche pourtant `UP` et `LOWER_UP` avec `ip link show`. La carte est active et le lien réseau virtuel est établi.

## Diagnostic et essais

Le diagnostic a été effectué avec accompagnement.

| Vérification ou essai | Résultat |
|---|---|
| `sysctl net.ipv6.conf.eth0.disable_ipv6` | Valeur initiale 1, IPv6 désactivé sur cette interface |
| `sudo sysctl -w net.ipv6.conf.eth0.disable_ipv6=0` | Valeur passée à 0, sans adresse de lien local au contrôle suivant |
| `sysctl net.ipv6.conf.eth0.addr_gen_mode` | Valeur 1, génération automatique de l’adresse de lien local désactivée |
| Tentatives de modification de addr_gen_mode par sysctl | Erreur de chemin introuvable, cause exacte non déterminée |
| `sudo ip link set dev eth0 addrgenmode eui64` | Commande terminée sans erreur, toujours aucune adresse au contrôle suivant |
| Lecture du profil réseau Ubuntu | Méthode IPv6 réglée sur « Désactivé » sur VM1 |

## Correction appliquée

Dans les paramètres réseau Ubuntu de VM1, j’ai choisi « Réseau local seulement » dans l’onglet IPv6, appliqué le réglage, puis désactivé et réactivé la connexion Filaire.

Une adresse de lien local est alors apparue. VM2 a ensuite présenté sa propre adresse, confirmée à l’affichage. Sa méthode IPv6 initiale n’a pas été relevée dans ce compte rendu.

| Machine | Carte | Adresse observée |
|---|---|---|
| LAB-UBUNTU-01 | eth0 | `fe80::215:5dff:fe01:7800/64` |
| LAB-UBUNTU-02 | eth0 | `fe80::215:5dff:fe01:7801/64` |

## Tests et résultats

Depuis VM2 vers VM1, avec aide pour construire la commande

```bash
ping -6 -c 4 fe80::215:5dff:fe01:7800%eth0
```

Résultat relevé pendant la manipulation, 4 paquets transmis et 4 reçus, sans perte.

Depuis VM1 vers VM2, commande construite et exécutée seul avec les notes précédentes disponibles

```bash
ping -6 -c 4 fe80::215:5dff:fe01:7801%eth0
```

Résultat confirmé sur la capture de la séance, 4 paquets transmis, 4 reçus, 0 % de perte. Temps moyen aller-retour affiché de 0,229 ms.

| Élément de commande | Rôle |
|---|---|
| `-6` | Utiliser IPv6 |
| `-c 4` | Envoyer quatre demandes |
| Adresse après les options | Désigner la VM de destination |
| `%eth0` | Choisir l’interface locale de sortie pour cette destination de lien local |

## Enseignements et points restant à vérifier

- Une carte réseau peut être active avec IPv6 désactivé dans son profil.
- Une VM peut posséder son adresse de lien local même lorsque l’autre VM est éteinte. Une réponse du voisin nécessite que celui-ci soit disponible.
- Vérifier le profil de connexion plus tôt aurait réduit le nombre d’essais.
- Le diagnostic et la correction de VM1 ont été guidés. Le ping inverse a été réalisé seul avec les notes disponibles.
- Le maintien des réglages après redémarrage reste à contrôler.

[Retour à la présentation](../README.md)
