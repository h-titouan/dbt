# Tutoriel dbt (data build tool)

## 1. Qu'est-ce que DBT ?
DBT (Data Build Tool) est un outil conçu pour transformer des données dans un entrepôt de données (data warehouse) en suivant une approche ELT (Extract-Load-Transform), ce qui représente un fonctionnement différents des traditionnelles ETL, où les données sont transformées avant d'être chargées dans un entrepôt. DBT lui, transforme les données directement dans l'entrepôt. Cela a pour objectif de rendre les données plus facilement accessible et utilisables.

### Concepts clés :
Voici les concepts clés de dbt qui seront abordés dans ce tutoriel :   
- **Modèles** : Des fichiers SQL dans lesquels on définis des requêtes pour la transformation des données.  
- **Tests** : Permet de faire des vérifications automatiques pour garantir la qualité des données (exemple : unicité, non-nullité).  
- **Documentation** : Un des avatanges de DBT est qu'il génère une documentation interactive à partir du code.  
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
*Cela crée une base de données appelée **dbt_learning***    
Vérifie que la base à bien été créée avec la commande suivante : `\l`   
*Cela va lister les bases de données*    

Maintenant, on peut créer une table. Cela va nous permettre de l'utiliser ensuite avec dbt :
```
CREATE TABLE raw_data.users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    created_at TIMESTAMP
);
```
Puis 
```
INSERT INTO raw_data.users (name, created_at)
VALUES 
    ('Alice', '2023-01-01'),
    ('Bob', '2022-12-15'),
    ('Charlie', '2021-05-20');
```
Nous avons ajouté une table de 3 colonnes avec un ID, un nom et une date de création. On peut vérifier que tout à fonctionné avec la commande suivante : `SELECT * FROM raw_data.users;`

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
      schema: raw_data
```

### 1.2.4 Tester la connexion 

Lance la commande suivante dans le terminal pour vérifier la connexion entre **DBT** et **PostgreSQL** : `dbt debug`  
*Si tout est correctement configuré tu devrais voir des messages indiquant que la connexion au profile.yml est OK mais pas à dbt_project.yml. C'est normal nous n'avons pas encore initialisé le projet*  

## 1.3 Initialiser un projet DBT  

- Dans un terminal, place toi dans un dossier dans lequel tu souhaites créer un nouveau dossier pour ton projet dbt puis initialise ton projet **dbt** : `dbt init my_first_project`  
- Place toi ensuite dans ce dossier : `cd my_first_project`
- Enfin, mets à jour le fichier **dbt_project.yml** qui est dans ton dossier pour pointer vers le profil que nous avons créé avec **PostgreSQL** : `profile: dbt_postgres_project` et relance : `dbt debug`. Cette fois tout est OK.

## 1.4 Préparer un modèle

Commençons par créer un modèle dans le dossier : `models/`  
- Crée un fichier `transform_example.sql` et ajoute le code suivant :
```
SELECT 
    id,
    name,
    created_at
FROM {{ source('raw_data', 'users') }}
WHERE created_at > '2022-01-01'
```

Ici, la fonction source('raw_data', 'users') fait référence à la table **users** dans ta base de données. Il est important de déclarer cette source dans un fichier YAML pour que **DBT** la reconnaisse.  
Ainsi, il faut définir la source dans un fichier YAML : 
- Crée un fichier `models/src_raw_data.yml` dans ton projet **DBT** pour enregistrer la source de données **users** :
```
version: 2
sources:
  - name: raw_data
    tables:
      - name: users
```
*Ce fichier permet de faire le lien entre ta table users dans PostgreSQL et le modèle DBT.*

## 1.5 Exécuter et tester ton modèle  

Dans ton terminal, lance la commande suivante pour exécuter le modèle : `dbt run --models transform_users`  
Tu peux désormais vérifier le résultat dans **PostgreSQL** avec la commande suivante : `SELECT * FROM raw_data.transform_users;`  
*Tu devrais voir les utilisateurs ayant créé leur compte après le 1er janvier 2022.*

## 2. Développer des modèles dans DBT

Dans cette partie, nous allons voir plus en profondeur comment optimiser vos modèles **DBT** :

### 2.1 Structure des fichiers DBT

Les fichiers modèles dans **DBT** sont compilés et exécutés pour créer des tables ou des vues dans la base de données cible. Voici des options que tu peux utiliser dans ton fichier `dbt_project.yml` pour configurer la façon dont **DBT** gèrera ces modèles :  

- **Materialization** : **DBT** propose plusieurs types de matérialisation, dont :  
  - *table* : Crée une table.  
  - *view* : Crée une vue, plus légère en termes de stockage.  
  - *incremental* : Permet de n'ajouter que les nouvelles données, utile pour les grands ensembles de données.  
  - *ephemeral* : Ne crée aucune table/affichage, mais utilise la transformation temporaire dans d'autres modèles.

Exemple pratique : 
```
models:
  my_first_project:
    +materialized: view   # Utilise une vue au lieu d'une table
    +schema: raw_data
```

### 2.2 Utilisation des sources DBT

Les sources dans **DBT** représentent des tables ou vues existantes dans la base de données qui ne sont pas gérées par **DBT**. Elles servent de point de départ pour vos transformations.

Par exemple, voici une source appelée `raw_data.users` définie dans `src_raw_data.yml` :
```
version: 2
sources:
  - name: raw_data
    schema: raw_data
    tables:
      - name: users
```

Pour utiliser cette source dans ton modèle, il faut lui faire référence avec la fonction source dans ton modèle :
```
-- models/transform_users.sql
SELECT
    id,
    name,
    created_at
FROM {{ source('raw_data', 'users') }}
WHERE created_at > '2022-01-01';
```
Cette syntaxe assure que **DBT** récupère les données de la table *users* située dans le schéma *raw_data*.

 ### 2.3 Utilisation des tests DBT

Les tests dans DBT permettent de vérifier que les données que tu manipules sont correctes.

### 2.3.1 Les différents types de tests 

DBT propose deux types de tests principaux :
- Les tests sur les **colonnes** :
  - **Test de non-nullité** (not_null)  
  - **Test d'unicité** (unique)  
  - **Test de valeur dans une liste** (accepted_values)  
- Les tests sur les modèles :
  - Ce sont des tests personnalisés à définir dans des fichiers de modèles `.sql`

Exemple d'un test de non-nullité :  
Dans le dossier `tests/`, créons un fichier pour tester que la colonne *id* dans la table **users n'est jamais nulle** :  
``` 
-- tests/not_null_id.sql
SELECT *
FROM {{ ref('transform_users') }}
WHERE id IS NULL;
```

Puis, tu peux l'exécuter en ajoutant ce test dans ton modèle `dbt_project.yml` :  
```
models:
  my_first_project:
    transform_users:
      +tests:
        - not_null:
            column_name: id
```

### 2.3.2 Lancer un test

Une fois les tests définis, tu peux les exécuter avec les commandes : `dbt run` pour lancer les modèles puis `dbt test` pour lancer les tests.

## 3. Les commandes utiles à DBT

Voici les commandes utiles pour faire fonctionner **DBT** :

- Compilation des modèles :
Avant d'exécuter un modèle, tu peux compiler les fichiers SQL pour vérifier qu'ils ne contiennent pas d'erreurs de syntaxe avec la commande suivante : `dbt compile`

- Exécution des modèles :
Pour exécuter tous les modèles (y compris les tests), tu peux utiliser la commande suivante : `dbt run`

- Exécution d'un modèle :
Pour exécuter un modèle spécifique, comme `transform_users`, voici la commande : dbt run --models transform_users`

- Vérification des tests :
Pour vérifier que les tests fonctionnent, exécute : `dbt test`

- Nettoyage des fichiers :
Parfois, il est nécessaire de nettoyer les fichiers générés par **DBT**, surtout après une série d'exécutions ou lors de bugs. Tu peux le faire avec la commande : `dbt clean`

## 4. Documentation DBT

DBT permet également de générer une documentation interactive pour ton projet, te permettant de visualiser les relations entre les modèles, sources, tests, etc.

- Générer de la documentation :
Pour générer une documentation automatique par dbt, utile la commande suivante : `dbt docs generate`

- Visualiser la documentation :
Une fois générée, tu peux lancer un serveur web local pour visualiser la documentation : `dbt docs serve`  

Cela ouvrira un navigateur sur [http://localhost:8000](http://localhost:8000), où tu pourras voir une vue interactive de ton projet DBT, avec les modèles, sources, et tests.

## Conclusion 

**DBT** est un outil puissant qui permet de transformer, tester et documenter des données de manière efficace. En suivant les étapes décrites, tu as désormais un projet **DBT** de test, avec des modèles de transformation, des tests, et de la documentation.

Quelques bonnes pratiques à retenir :

**Modularité** : Crée des modèles SQL simples et clairs, et utilise des fichiers de configuration pour les personnaliser.  
**Tests** : Ajoute des tests pour valider l'intégrité des données.  
**Documentation** : Génère et publie la documentation pour rendre ton projet plus compréhensible.  

Tu peux maintenant explorer davantage les fonctionnalités de DBT, comme les **macros** pour réutiliser du code SQL, les **hooks** pour automatiser des processus avant ou après les exécutions de modèle ou encore les **snapshots** pour historiser les différentes versions. Enfin, **DBT** permet de développer un projet *CI/CD* avec *Git* n'hésites pas à explorer ce domaine !

## Sources 

Voici les sources qui ont été utilisées pour rédiger ce tutoriel : 

- La documentation interne de Seenovate  
- La documentation officielle de [**DBT**](https://docs.getdbt.com/guides)  
- Introduction à dbt : [L'outil de construction de données](https://www.youtube.com/watch?v=7ovpDjDJs6w)  
- Introduction to dbt : [Step by Step Guide for Beginners](https://rasiksuhail.medium.com/introduction-to-dbt-step-by-step-instructions-for-beginners-a16d343c8826)  
- La documentation de **Snowflake** : [Data Engineering with Snowpark Python and dbt](https://quickstarts.snowflake.com/guide/data_engineering_with_snowpark_python_and_dbt/index.html#0)  
