# Tutoriel dbt (data build tool)

## 1 Qu'est-ce que DBT ?
DBT (Data Build Tool) est un outil conçu pour transformer des données dans un entrepôt de données (data warehouse) en suivant une approche ELT (Extract-Load-Transform), ce qui représente un fonctionnement différents des traditionnelles ETL, où les données sont transformées avant d'être chargées dans un entrepôt. DBT lui, transforme les données directement dans l'entrepôt. Cela a pour objectif de rendre les données plus facilement accessible et utilisables.

### Concepts clés :
Voici les concepts clés de dbt qui seront abordés dans ce tutoriel :   
- **Modèles** : Des fichiers SQL dans lesquels on définis des requêtes pour la transformation des données.  
- **Tests** : Permet de faire des vérifications automatiques pour garantir la qualité des données (exemple : unicité, non-nullité).  
- **Documentation** : Un des avatanges de DBT est qu'il génère une documentation interactive à partir du code.  
- **Snapshots** : Ce sont des fonctionnalités pour capturer des versions historisées des données.  
- **Macros** : Elles permettent de réutiliser du code dans certaines situations et sont écrites en *Jinja* (un moteur de templates).  
- **Ecosystème** : DBT s'intègre avec des entrepôts de données tels que *Snowflake*, *BigQuery*, *Redshift*, et *PostgreSQL*.  

## 1.1 Installer et configurer DBT

Dans ce tutoriel, nous allons nous intéresser à dbt-core qui s'utilise avec Python. Les pré-requis seront donc les suivants :   
- Avoir Python (DBT est une application Python).  
- Avoir accès à un entrepôt de données compatible (exemple : Snowflake, PostgreSQL, BigQuery).  
- Ouvrir un terminal (Windows, macOS ou Linux).  

**Installer DBT :**

Exécute la commande suivante dans ton terminal pour installer DBT : `pip install dbt-core`    

Puis installe un adaptateur DBT pour se connecter à l'entrepôt de données : `pip install dbt-snowflake dbt-postgres dbt-bigquery`   
Il n'est pas nécessaire d'installer les trois, fais le pip install de l'adaptateur que tu vas utiliser pour se connecter aux données.  
Une fois l'installation terminée, exécute dans le terminal : `dbt --version`  
*Cela doit afficher la version de DBT installée.*

## 1.2 Configurer DBT avec ton entrepôt de données
DBT utilise un fichier appelé profiles.yml pour gérer les connexions aux entrepôts de données. Pour ce tutoriel, nous utiliserons PostgreSQL en local. 

Pour suivre le tutoriel, il est donc nécessaire d'installer [**PostgreSQL**](https://www.postgresql.org/download/)  

### 1.2.1 Installation de PostgreSQL

Pendant l'installation de **PostgreSQL**, tu peux choisir le mot de passe par défaut *postgres* et utiliser le port par défaut *5432*  
Une fois **PostgreSQL** installé, tape ceci dans ton terminal pour vérifier sa bonne installation : `psql --version`  
*Si cela ne fonctionne pas, penses à l'ajouter aux variables d'environnement de ton ordinateur*

### 1.2.2 Créer une base de données pour DBT

Ouvre un terminal pour se connecter à **PostgreSQL** avec la commande suivante : `psql -u postgres`  

Une fois connecté, exécute les commandes suivantes : `CREATE DATABASE dbt_learning;`  
*Cela crée une base de données appelée **dbt_learning** *    
Vérifie que la base à bien été créée avec la commande suivante : `\l`   
*Cela va lister les bases de données*  

Dès lors, on peut configurer **DBT** avec **PostgreSQL**

### 1.2.3 Créer le fichier profiles.yml

Dans le dossier utilisateur de ton ordinateur, un dossier **.dbt** doit apparaître.  
On va alors ajouter le fichier **profiles.yml** dans ce dossier.   

Le fichier **profiles.yml** est organisé de la manière suivante 
``` profiles.yml
dbt_postgres_project:
  target: dev
  outputs:
    dev:
      type: postgres
      host: localhost
      user: dbt_user
      password: dbt_password
      port: 5432
      dbname: dbt_learning
      schema: public
```

### 1.2.4 Tester la connexion 

Lance la commande suivante dans le terminal pour vérifier la connexion entre **DBT** et **PostgreSQL** : `dbt debug`  
*Si tout est correctement configuré tu devrais voir des messages indiquant que la connexion est réussie*  

## 1.3 Initialiser un projet DBT  

- Dans un terminal, place toi dans un dossier dans lequel tu souhaites créer un nouveau dossier pour ton projet dbt puis initialise ton projet **dbt** : `dbt init my_first_project`  
- Place toi ensuite dans ce dossier : `cd my_first_project`
- Enfin, mets à jour le fichier **dbt_project.yml** qui est dans ton dossier pour pointer vers le profil que nous avons créé avec **PostgreSQL** : `profile: dbt_postgres_project`

## 1.4 Lancer un test 

Commençons par créer un modèle dans le dossier : `models/`  
- Crée un fichier `example.sql` et ajoute le code suivant : `SELECT 1 AS exemple_column`
- Lance la commande suivante pour exécuter le modèle SQL et créer une table dans la base de données `dbt run`
- Vérifie que la table est bien créée dans **PostgreSQL** en exécutant ce code dans le terminal avec lequel tu t'es connecté à **PostgreSQL** : `SELECT * FROM public.example;`  


