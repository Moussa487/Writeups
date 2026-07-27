# Ripper — Vulnhub

**Difficulté :** Moyen
**Catégorie :** Web (Information Disclosure) / Privilege Escalation
**Date :** 2025

## Objectif

Compromettre la machine cible, obtenir un accès utilisateur via une fuite
d'information sur le service web, puis escalader les privilèges jusqu'à
root.

## Reconnaissance

Scan complet des ports et services :

```bash
sudo nmap -p- -sV <IP_CIBLE>
```

Résultats :

| Port | Service | Version |
|------|---------|---------|
| 22/tcp | ssh | OpenSSH 7.6p1 4ubuntu0.3 |
| 80/tcp | http | Apache httpd 2.4.29 (Ubuntu) |
| 10000/tcp | http | MiniServ 1.910 (Webmin) |

## Énumération

Énumération du contenu exposé par le service web (répertoires/fichiers
accessibles sous `/var/www/`). Découverte d'un fichier `secrets.php`
exposant en clair des identifiants applicatifs :

```
User : ripper
Pass : Gamespeopleplay
```

**Vulnérabilité identifiée :** exposition d'un fichier de configuration
contenant des identifiants en clair, accessible directement via le serveur
web (mauvaise gestion des permissions/configuration Apache).

## Exploitation

Réutilisation des identifiants trouvés pour se connecter directement en
SSH :

```bash
ssh ripper@<IP_CIBLE>
# password: Gamespeopleplay
```

Accès utilisateur `ripper` obtenu.

## Élévation de privilèges

Une fois connecté, énumération des droits sudo disponibles pour
l'utilisateur `ripper` :

```bash
sudo -l
```

L'utilisateur `ripper` est autorisé à exécuter un binaire système sans
restriction de mot de passe. Ce type de binaire (éditeur, interpréteur,
outil d'archivage...) permet une évasion vers un shell root via une
technique **GTFOBins** classique :

```bash
sudo <binaire_autorisé>
# évasion vers un shell root via la technique GTFOBins correspondante
```

Une fois le shell root obtenu :

```bash
id
# uid=0(root) gid=0(root)
cat /root/root.txt
```

## Résultat

- [x] Accès utilisateur obtenu (ripper)
- [x] Root flag obtenu

## Ce que j'ai appris

Cette machine met en évidence un classique du monde réel : des identifiants
laissés en clair dans un fichier de configuration accessible publiquement
suffisent souvent à obtenir un accès initial complet, sans avoir besoin
d'exploiter une faille complexe. La chaîne se termine par une mauvaise
configuration sudo, illustrant à quel point un droit d'exécution mal
restreint peut annuler tout le reste des mesures de sécurité en place.

## Remédiation

- Ne jamais stocker d'identifiants en clair dans des fichiers accessibles
  par le serveur web ; utiliser des variables d'environnement ou un
  gestionnaire de secrets.
- Auditer régulièrement les règles `sudoers` pour éviter d'autoriser des
  binaires exploitables via GTFOBins sans mot de passe.
- Appliquer le principe du moindre privilège : n'accorder que les droits
  strictement nécessaires à chaque compte.
- Activer les mises à jour automatiques pour les services exposés (dont
  Webmin, régulièrement concerné par des vulnérabilités connues).
- Appliquer une politique de mot de passe robuste (12+ caractères,
  complexité).
