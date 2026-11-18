# Manuel d’installation et de configuration — PharmaGest

Version du 18 novembre 2026. Destiné à l’installation **locale** du projet Django et à sa présentation académique. Les commandes ci-dessous sont à exécuter par l’installateur ; la rédaction de ce manuel n’a lancé aucune migration ni initialisation de données.

## 1. Prérequis

| Élément | Version / utilité |
|---|---|
| Python | 3.12 ou version supérieure compatible avec les dépendances ; environnement observé : 3.14.8 |
| Django | 6.0.4, installé par requirements.txt |
| Pillow | 12.1.0, validation et manipulation des photos |
| ReportLab | >= 4.4 et < 5, production des documents PDF |
| Navigateur | Navigateur récent, sur ordinateur ou mobile |
| Accès réseau | Pour télécharger les dépendances si elles ne sont pas déjà disponibles localement |

SQLite est fourni avec Python : aucun serveur MySQL ou PostgreSQL n’est requis. Bootstrap, les icônes, le CSS et le JavaScript de l’application sont locaux. Aucun build front, Node.js ou npm n’est nécessaire pour démarrer l’application. Node.js sert seulement aux tests JavaScript facultatifs.

## 2. Se placer à la racine du projet

Ouvrir le terminal de VS Code dans le dossier contenant `manage.py` et `requirements.txt`.

Sous Linux/macOS, dans cet environnement :

```bash
cd /home/kakarot/Documents/Django/Pharmacie
python3 --version
```

Sur un autre poste, adapter le chemin. Sous Windows PowerShell, utiliser `cd` avec le chemin local du projet et vérifier Python avec `py --version`.

## 3. Préparer l’environnement Python

### Linux / macOS

Si un environnement `.venv` fonctionnel existe déjà, l’activer directement ; ne pas le recréer inutilement.

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

### Windows PowerShell

```powershell
py -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

Les commandes suivantes utilisent `python` avec l’environnement activé sous Linux/macOS. Sous Windows, remplacer `python` par `.\.venv\Scripts\python.exe` ; cela évite de changer la politique d’exécution PowerShell.

L’installation peut échouer si Python est trop ancien ou si le téléchargement des paquets est impossible. Corriger cette cause avant de poursuivre.

## 4. Première installation sur une nouvelle base

Cette étape concerne une **nouvelle installation**. Si le projet contient déjà tes médicaments, ventes ou comptes, passer à la section 6 pour le démarrer. Ne pas supprimer `db.sqlite3` pour « réinstaller ».

### 4.1 Créer les tables

```bash
python manage.py migrate
```

Django crée les tables techniques et métier dans `db.sqlite3`, sauf si `DATABASE_PATH` désigne un autre fichier. Il n’est pas nécessaire d’exécuter `makemigrations` : les migrations du projet sont livrées.

### 4.2 Initialiser les rôles

```bash
python manage.py setup_roles
```

La commande crée ou remet à jour les cinq groupes : Administrateur, Pharmacien responsable, Gestionnaire de stock, Caissier et Responsable de gestion. Une nouvelle exécution réattribue les permissions prévues par le code : ne pas l’utiliser systématiquement si les droits ont été personnalisés.

### 4.3 Créer le premier administrateur

```bash
python manage.py createsuperuser
```

Renseigner un **nom d’utilisateur**, une adresse courriel facultative et un mot de passe. Le mot de passe ne s’affiche pas pendant la saisie ; c’est normal. Aucun identifiant ou mot de passe universel n’est fourni.

Le superutilisateur dispose de tous les droits. Il n’est à créer qu’une fois. Les utilisateurs ordinaires pourront ensuite être créés dans **Utilisateurs et rôles**.

### 4.4 Vérifier la configuration

```bash
python manage.py check
```

Le résultat attendu est l’absence d’anomalie Django. Cette commande ne remplace pas une recette navigateur.

## 5. Configuration locale

Le projet lit les variables d’environnement dans `config/settings.py`. **Un fichier `.env` n’est pas chargé automatiquement.** `.env.example` est un exemple de configuration, pas un fichier exécuté par Django.

| Variable | Usage / comportement par défaut |
|---|---|
| `DJANGO_DEBUG` | `1` pour le développement local |
| `DJANGO_ALLOWED_HOSTS` | localhost, 127.0.0.1 et adresse de boucle locale IPv6 |
| `DATABASE_PATH` | Chemin de la base SQLite ; par défaut, db.sqlite3 à la racine |
| `PHARMACY_NAME` | Nom de pharmacie affiché ; par défaut PharmaGest |
| `PHARMACY_ADDRESS` | Adresse affichée sur les documents |
| `PHARMACY_LEGAL` | Mention légale des documents, à adapter au contexte réel |
| `DJANGO_SECRET_KEY` | Secret d’application ; indispensable et privé en production |
| `EMAIL_BACKEND` | Console en développement ; SMTP à configurer pour envoyer de vrais courriels |

Exemple Linux/macOS, à exécuter dans le terminal qui lancera le serveur :

```bash
export DJANGO_DEBUG=1
export PHARMACY_NAME='PharmaGest'
```

Équivalent PowerShell :

```powershell
$env:DJANGO_DEBUG = '1'
$env:PHARMACY_NAME = 'PharmaGest'
```

Les montants sont en ariary avec deux décimales, l’interface est en français et le fuseau est `Indian/Antananarivo`. Pour les courriels réels, configurer un serveur SMTP : EMAIL_BACKEND, EMAIL_HOST, EMAIL_PORT, EMAIL_HOST_USER, EMAIL_HOST_PASSWORD, EMAIL_USE_TLS et DEFAULT_FROM_EMAIL. Ne pas mettre les secrets dans le code, le rapport ou les documents partagés.

## 6. Démarrer et arrêter le projet

À chaque nouvelle ouverture du terminal Linux/macOS :

```bash
cd /home/kakarot/Documents/Django/Pharmacie
source .venv/bin/activate
python manage.py runserver
```

Sous Windows, depuis la racine :

```powershell
.\.venv\Scripts\python.exe manage.py runserver
```

Ouvrir **http://127.0.0.1:8000/**. Se connecter avec le **nom d’utilisateur et le mot de passe**, pas avec l’adresse courriel. Garder le terminal ouvert. Arrêter le serveur avec **Ctrl+C**.

Si le port est occupé :

```bash
python manage.py runserver 8001
```

Ouvrir alors **http://127.0.0.1:8001/**. Utiliser le port réellement annoncé dans le terminal. Une base ou un serveur de recette séparé peut avoir des comptes différents.

Un démarrage normal ne nécessite ni migrate, ni setup_roles, ni createsuperuser, ni seed_demo. Après une mise à jour du code contenant de nouvelles migrations, leur application relève d’une maintenance préparée avec sauvegarde.

## 7. Données fictives facultatives

Pour un catalogue vide destiné exclusivement à une démonstration :

```bash
python manage.py seed_demo
```

La commande crée des données fictives et un compte `demo_admin`, avec un mot de passe aléatoire affiché une seule fois dans le terminal. Elle refuse un catalogue déjà rempli si ce compte n’existe pas ; si `demo_admin` existe déjà, elle ne modifie rien. Elle ne remet pas une base à zéro et ne remplace pas un mot de passe existant.

Pour une base déjà utilisée, conserver les données présentes et suivre le parcours manuel du [guide d’utilisation](UTILISATION.md). Ne pas effacer les opérations existantes pour forcer une démonstration.

## 8. Sauvegarde et restauration

Les fichiers à préserver sont la base SQLite et le dossier `media/` contenant les photos. Ne pas considérer une copie du code ou des migrations comme une sauvegarde des opérations.

Exemple Linux/macOS, serveur arrêté pour faciliter une sauvegarde cohérente avec les médias :

```bash
mkdir -p backups
python manage.py backup_db backups/pharmagest-sauvegarde.sqlite3
```

Choisir un nom de destination neuf à chaque sauvegarde : la commande refuse d’écraser un fichier. Copier également `media/` avec l’outil de fichiers habituel, s’il existe. Conserver les sauvegardes dans un emplacement privé distinct du dossier partagé.

Pour restaurer vers un **nouveau fichier** :

```bash
python manage.py restore_db backups/pharmagest-sauvegarde.sqlite3 restauration-verifiee.sqlite3
```

La commande contrôle le fichier source et refuse une destination existante. Elle ne remplace pas automatiquement la base active. Une bascule vers le fichier restauré doit être décidée séparément, serveur arrêté et sauvegarde conservée. Ce manuel n’a pas exécuté ces opérations.

## 9. Contrôles facultatifs pour le développeur

Depuis l’environnement Python du projet :

```bash
python manage.py check
python manage.py makemigrations --check --dry-run
python manage.py test
```

La deuxième commande vérifie le décalage modèles/migrations sans écrire de migration. Les tests Django utilisent une base de test distincte. Avec Node.js disponible :

```bash
node tools/operation_math.test.mjs
node tools/cashier_math.test.mjs
```

L’absence de ReportLab peut entraîner l’utilisation du PDF de secours et l’ignorance de certains tests ReportLab. Une installation complète des dépendances active le moteur ReportLab.

## 10. Dépannage rapide

| Problème | Action |
|---|---|
| `No module named django` | Utiliser le Python de .venv et installer requirements.txt |
| Python trop ancien | Installer/utiliser Python >= 3.12, puis créer l’environnement adapté |
| `no such table` sur une installation neuve | Vérifier DATABASE_PATH puis exécuter migrate sur la base voulue |
| Port déjà utilisé | Choisir 8001 et ouvrir cette même adresse |
| Identifiants refusés | Vérifier nom d’utilisateur, compte actif, base et port ; ne pas recréer un compte à chaque démarrage |
| Mot de passe oublié de l’administrateur | Depuis le terminal autorisé : `python manage.py changepassword NOM_UTILISATEUR` |
| `DisallowedHost` | Vérifier l’adresse utilisée et DJANGO_ALLOWED_HOSTS ; en local, utiliser 127.0.0.1 |
| Styles anciens | Rechargement complet du navigateur avec Ctrl+Maj+R |
| Courriel non reçu | En développement, le contenu du courriel est affiché dans le terminal ; SMTP requis pour un véritable envoi |
| Lot ou médicament absent de la caisse | Vérifier la fiche, l’autorisation et le stock ; voir le guide d’utilisation |

## 11. Limite : déploiement public

`runserver` est un serveur de développement. Une exploitation publique nécessite un hébergement adapté à Django/WSGI, HTTPS, un secret privé, DJANGO_DEBUG=0, les hôtes autorisés, les fichiers statiques collectés et une configuration SMTP/sauvegarde. Avec DEBUG=0, la configuration impose HTTPS. Les photos doivent rester protégées par authentification et permission ; ne pas publier directement tout media/ via le serveur web. Les règles professionnelles et mentions des documents doivent être validées localement avant usage réel.

## 12. Portée de vérification de ce manuel

Commandes et paramètres confrontés aux fichiers du projet, aux aides des commandes et aux métadonnées des dépendances installées. `manage.py check` a été exécuté sans anomalie ; les aides CLI de migrate, setup_roles, createsuperuser, backup_db et restore_db ont été consultées sans exécuter ces opérations. Une réinstallation complète, le parcours Windows, une restauration et un déploiement public n’ont pas été exécutés pour rédiger ce document.
