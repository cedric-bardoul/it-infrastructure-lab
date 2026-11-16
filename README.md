# it-infrastructure-lab

Laboratoire pratique de reconversion IT autour des réseaux, des systèmes et de la sécurité informatique.

Je documente ici mes configurations, mes diagnostics et mes résultats de tests. Les comptes rendus précisent les manipulations guidées, les exercices reproduits seul et les contrôles restant à effectuer.

## Travaux réalisés

| Date | Compte rendu | Résultat observé |
|---|---|---|
| 16 novembre 2026 | [Diagnostic IPv6 de lien local](reseaux/01-diagnostic-ipv6.md) | Adresses IPv6 présentes et pings réussis entre deux VM dans les deux sens |
| 16 novembre 2026 | [Passerelle, NAT et DNS](reseaux/02-passerelle-nat-dns.md) | Accès IPv4 à une adresse Internet et résolution de noms sur les deux VM |

## Environnement du lab

- Hôte Windows 11 Pro avec Hyper-V.
- Deux machines virtuelles Ubuntu, LAB-UBUNTU-01 et LAB-UBUNTU-02.
- Réseau IPv4 du lab en `192.168.10.0/24`.
- Commutateur initialement privé, passé en interne pour la manipulation NAT.
- Windows configuré avec l’adresse `192.168.10.1/24` côté lab et le NAT nommé `NAT-LAB`.

Le commutateur conserve le nom `LAB-PRIVE` dans les sorties de la séance, même après son passage au type interne.

## Démarche

Chaque intervention présente le contexte, le problème, les vérifications, la correction, les tests et les enseignements. Les commandes de modification sont liées à la configuration de ce laboratoire.

## Suite du parcours

Approfondissement des réseaux, puis Windows Server / Active Directory, dépannage Windows, Microsoft 365 / Entra ID, bases d’Intune, Linux et sécurité de l’infrastructure.

## Autres travaux

[Mon laboratoire AWS](https://github.com/cedric-bardoul/aws-security-labs).
