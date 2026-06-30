# 🧪 Minilab "LinuxIsGood" - Infrastructure Linux Centralisée, Sécurisée & Résiliente

Ce dépôt rassemble l'ensemble des scripts, configurations et documentations techniques pour le déploiement, la sécurisation avancée et l'évolution de l'infrastructure réseau et système de l'association **LinuxIsGood**.

---

## 🌟 Les Avantages de l'Architecture "LinuxIsGood"

* **Gestion Centralisée des Identités** : Centralisation absolue des comptes et des groupes via openLDAP. Un utilisateur créé sur le serveur Master se connecte instantanément sur n'importe quelle machine du parc.
* **Sécurité Périmétrique et Chiffrement** : Isolation des flux d'administration externes (VPN), chiffrement des authentifications (LDAPS) et double facteur (MFA) pour endiguer l'usurpation d'identité.
* **Maintenance Industrielle Automatisée** : Script Bash centralisé permettant de mettre à jour l'intégralité du parc en une seule commande via des liaisons SSH asymétriques sécurisées.
* **Résilience et Continuité (PCA/PRA)** : Haute disponibilité face aux coupures réseau (redondance LDAP) et protocole de reconstruction rapide en cas de sinistre majeur (RTO < 2h, RPO < 24h).
* **Agilité face aux Contraintes Matérielles** : Intégration de matériel reconditionné (Bornes Wi-Fi Cisco) et contournement des verrous système (gestion alternative des dossiers `/home`).

---

## 🗺️ Cartographie de l'Infrastructure
Le parc est segmenté sur le sous-réseau privé `192.168.15.0/24` :

| Machine / Nœud | Rôle Système | IP Fixe | Rôle Technique / Composant |
| :--- | :--- | :--- | :--- |
| **Master** | Serveur Principal / Maître | `192.168.15.254` | Hébergement openLDAP (`slapd`), DNS/DHCP, centralisation SSH |
| **Slave** | Réplicat / Serveur Secondaire | `192.168.15.252` | Relais d'authentification local (`nslcd`), secours et cible sauvegarde |
| **Client** | Station de Travail Utilisateur | `192.168.15.X` | Machine cliente consommant les ressources de l'association |
| **Bornes Wi-Fi** | Points d'accès sans-fil | `192.168.15.50-60`| Couverture Wi-Fi autonome pour les utilisateurs et invités |

---

## 🛠️ Phase Opérationnelle : Les 5 Étapes Initiales
1. **Initialisation Réseau** : Déploiement des VM (Debian/Ubuntu), attribution des adresses IP statiques et validation de la connectivité via des tests de `ping` croisés.
2. **Déploiement openLDAP (Master)** : Installation de `slapd`, structuration de l'arborescence DN (`dc=linuxisgood,dc=local`) avec les OU `people` et `groups`, et création de l'utilisateur `client` (UID `2002`).
3. **Liaison Clients (Slave & Client)** : Interconnexion via `libnss-ldapd` et `libpam-ldapd`. Déportation des dossiers personnels des utilisateurs dans `/var/home_ldap/` pour contourner le verrou d'environnement virtuel du dossier `/home` natif.
4. **Automatisation par Clés SSH** : Génération d'une paire de clés asymétriques `RSA 4096 bits` sur le Master et déploiement via `ssh-copy-id` pour permettre l'exécution transparente du script de mise à jour centralisé `update_all.sh`.
5. **Plan de Secours Initial (PCA/PRA)** : Configuration de redondances d'URI dans `nslcd.conf` et automatisation d'extractions de l'annuaire via `slapcat` exportées chaque nuit sur le Slave.

---

## 🔒 6. Sécurisation Avancée (Pour aller plus loin)

* **Filtrage Périmétrique (Accès VPN Unique)** : Configuration d'un pare-feu strict (`UFW` ou `iptables`) bloquant tous les flux extérieurs. Seul le port du serveur VPN (ex: WireGuard `UDP/51820`) est ouvert pour permettre l'administration sécurisée à distance.
* **Sécurisation des flux LDAP (LDAPS / StartTLS)** : Chiffrement de l'authentification réseau à l'aide de certificats TLS, forçant l'utilisation du port sécurisé `636` ou de `StartTLS` pour interdire le transit des mots de passe en clair.
* **Interface Graphique de Gestion Unifiée (UI)** : Déploiement de solutions comme FusionDirectory, Webmin ou Cockpit afin de piloter graphiquement l'ensemble des services critiques (NFS, LDAP, DNS, DHCP) sans ligne de commande.
* **Authentification Forte (MFA)** : Intégration du module `libpam-google-authenticator` ou d'un serveur PrivacyIDEA pour exiger un code de validation TOTP temporaire lors des ouvertures de session.

---

## 📈 7. Évolutions Logicielles et Stockage (Pour aller encore plus loin)

* **Gestionnaire d'Impression Centralisé** : Déploiement de **CUPS** pour unifier les files d'attente d'impression et attribuer les droits selon les groupes LDAP de l'association.
* **Sauvegarde des Configurations** : Suivi rigoureux des modifications du répertoire `/etc` sur l'ensemble du parc à l'aide d'**etckeeper** couplé à un dépôt Git interne.
* **Duplication des Données NFS** : Automatisation de la réplication des partages de fichiers via `rsync` ou `rclone` vers un **NAS** physique secondaire ou un espace **Cloud sécurisé / S3** chiffré.

---

## 📡 8. Intégration Wi-Fi Cisco Aironet 1142 & Alcasar

Pour exploiter le don de bornes Wi-Fi Cisco Aironet 1142 configurées initialement en mode Managé (Lightweight/WLC), l'association réalise une reprogrammation de leur firmware.

* **Mode Autonome (Standalone)** : Flashage du micrologiciel pour rendre les bornes indépendantes de tout contrôleur matériel coûteux.
* **Couplage Alcasar** : Raccordement des bornes autonomes sur un VLAN étanche (ex: VLAN 10) redirigé directement vers une VM **Alcasar** pour fournir un portail captif sécurisé et conforme aux réglementations d'accès public à Internet.

*(Pour obtenir les instructions détaillées de flashage étape par étape, veuillez vous référer au document PDF joint : `Procedure_flashage_cisco_1142.pdf`).*

---
> 🚀 **Déployé et documenté pour l'association LinuxIsGood.**
