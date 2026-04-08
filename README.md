# TP-Stockage-Cloud-Owncloud

# ☁️ Compte-Rendu TP : Déploiement d'ownCloud via Docker

Ce projet présente la mise en place d'une solution de stockage Cloud personnelle (**ownCloud**) sur un serveur Debian, en utilisant la technologie de conteneurisation **Docker**.

---

## 🚀 1. Objectifs du TP
* Installation et configuration de l'environnement Docker & Docker-Compose.
* Déploiement d'une architecture bi-tiers (Serveur Web + Base de données MariaDB).
* Résolution de conflits de services (ports réseaux).
* Validation de la synchronisation avec un client lourd Windows.

---

## 🛠️ 2. Architecture Technique

| Composant | Technologie | Port Hôte |
| :--- | :--- | :--- |
| **Système d'exploitation** | Debian 12 (VM Proxmox) | - |
| **Serveur Cloud** | ownCloud Server 10.13.1 | **8081** |
| **Base de données** | MariaDB 10.5 | 3306 |
| **Client distant** | Windows 10 | Client ownCloud Desktop |

---

## ⚙️ 3. Mise en œuvre et Résolution de problèmes

### Analyse des conflits (Question du TP)
Lors du premier lancement, une erreur `bind: address already in use` est apparue sur le port **8080**. 
* **Diagnostic** : La commande `sudo ss -lptn 'sport = :8080'` a révélé que le service `waptserver` occupait déjà ce port.
* **Solution** : Modification du fichier `docker-compose.yml` pour mapper le port interne 8080 vers le port externe **8081**.

### Configuration Docker-Compose
Le fichier final assure la liaison entre le serveur et la base de données via des variables d'environnement.


version: '3.1'

services:
  db:
    image: mariadb:10.5
    container_name: owncloud_db
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=password
      - MYSQL_DATABASE=owncloud
      - MYSQL_USER=owncloud
      - MYSQL_PASSWORD=owncloud

  owncloud:
    image: owncloud/server:10.13.1
    container_name: owncloud_server
    restart: always
    ports:
      - 8081:8080
    depends_on:
      - db
    environment:
      - OWNCLOUD_DOMAIN=192.168.20.39:8081
      - OWNCLOUD_DB_TYPE=mysql
      - OWNCLOUD_DB_NAME=owncloud
      - OWNCLOUD_DB_USER=owncloud
      - OWNCLOUD_DB_PASSWORD=owncloud
      - OWNCLOUD_DB_HOST=db


**Commandes de préparation du système :**

# Mise à jour et installation des outils
apt update && apt upgrade -y
apt install docker.io docker-compose -y

# Création de l'environnement de travail
mkdir owncloud-docker && cd owncloud-docker
nano docker-compose.yml

# Commandes de gestion du déploiement
docker-compose up -d      # Lancer les services
docker ps                 # Vérifier l'état des conteneurs
docker-compose down -v    # Tout supprimer (y compris les volumes)


## ⚙️ Étape 3 : Fonctionnalités et Possibilités d'ownCloud

ownCloud n'est pas qu'un simple espace de stockage ; c'est une plateforme extensible qui offre des fonctionnalités critiques pour l'administration système et la gestion d'entreprise.

### 1. Gestion des Annuaires (LDAP / Active Directory)
C'est l'une des fonctionnalités les plus puissantes pour un administrateur :
* **Centralisation** : Permet de connecter ownCloud à un serveur LDAP ou un Active Directory Windows.
* **Authentification Unique** : Les utilisateurs utilisent leur session Windows habituelle pour se connecter au Cloud.
* **Synchronisation des Groupes** : Les groupes créés dans l'annuaire (ex: Groupe "Comptabilité") sont automatiquement récupérés dans ownCloud pour gérer les permissions.

### 2. Gestion du Stockage et Partage
* **Partages Publics** : Possibilité de créer des liens de téléchargement protégés par mot de passe et avec une date d'expiration.
* **Fédérations de serveurs** : Permet de lier deux serveurs ownCloud distants (ex: entre deux sites d'une même entreprise) pour partager des fichiers de manière transparente.
* **Corbeille et Versions** : Récupération de fichiers supprimés par erreur et possibilité de revenir à une version précédente d'un document (Versionning).

### 3. Sécurité et Contrôle
* **Chiffrement** : Les données peuvent être chiffrées sur le disque du serveur (Encryption at rest).
* **Pare-feu de fichiers** : Possibilité d'interdire l'accès au Cloud selon l'adresse IP, le type d'appareil ou la zone géographique.
* **Audit Logs** : Journalisation complète de qui a accédé à quoi et quand (indispensable pour la conformité RGPD).

### 4. Marketplace et Applications
Le serveur ownCloud permet d'installer des "Apps" supplémentaires pour étendre ses capacités :
* **Calendar & Contacts** : Synchronisation des agendas (CalDAV) et des répertoires (CardDAV).
* **Collabora Online / ONLYOFFICE** : Ajout d'une suite bureautique pour éditer des documents Word/Excel directement dans le navigateur.
* **Antivirus** : Scan automatique des fichiers téléchargés pour bloquer les malwares.
