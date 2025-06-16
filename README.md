# Tutoriel dbt (data build tool)

[Image DBT](dbt_image.png)

## Table des matières

- [1. Qu'est-ce que DBT ?](#1-quest-ce-que-dbt-)
- [1.1 Installer et configurer DBT](#11-installer-et-configurer-dbt)
- [1.2 Configurer DBT avec ton entrepôt de données](#12-configurer-dbt-avec-ton-entrepôt-de-données)
  - [1.2.1 Installation de PostgreSQL](#121-installation-de-postgresql)
  - [1.2.2 Créer une base de données pour DBT](#122-créer-une-base-de-données-pour-dbt)
  - [1.2.3 Créer le fichier profilesyml](#123-créer-le-fichier-profilesyml)
  - [1.2.4 Tester la connexion](#124-tester-la-connexion)
- [1.3 Initialiser un projet DBT](#13-initialiser-un-projet-dbt)
- [1.4 Préparer un modèle](#14-préparer-un-modèle)
- [1.5 Exécuter et tester ton modèle](#15-exécuter-et-tester-ton-modèle)
- [2. Développer des modèles dans DBT](#2-développer-des-modèles-dans-dbt)
  - [2.1 Structure des fichiers DBT](#21-structure-des-fichiers-dbt)
  - [2.2 Utilisation des sources DBT](#22-utilisation-des-sources-dbt)
  - [2.3 Utilisation des tests DBT](#23-utilisation-des-tests-dbt)
    - [2.3.1 Les différents types de tests](#231-les-différents-types-de-tests)
    - [2.3.2 Lancer un test](#232-lancer-un-test)
- [3. Utilisation des variables dans DBT](#3-utilisation-des-variables-dans-dbt)
  - [3.1 Définir une variable dans le fichier dbt_project.yml](#31-définir-une-variable-dans-le-fichier-dbt_projectyml)
  - [3.2 Utiliser une variable dans un modèle SQL](#32-utiliser-une-variable-dans-un-modèle-sql)
  - [3.3 Redéfinir une variable depuis la ligne de commande](#33-redéfinir-une-variable-depuis-la-ligne-de-commande)
  - [3.4 Utilisation avancée : variables conditionnelles](#34-utilisation-avancée--variables-conditionnelles)
  - [3.5 Définir dynamiquement un schéma ou un nom de table avec des variables](#35-définir-dynamiquement-un-schéma-ou-un-nom-de-table-avec-des-variables)
- [4. Utiliser les macros dans DBT](#4-utiliser-les-macros-dans-dbt)
  - [4.1 Built-in macros (macros intégrées)](#41-built-in-macros-macros-intégrées)
  - [4.2 Créer des custom macros](#42-créer-des-custom-macros)
  - [4.3 Utiliser des macros dans des boucles et conditions](#43-utiliser-des-macros-dans-des-boucles-et-conditions)
  - [4.4 Le package dbt_utils](#44-le-package-dbt_utils)
- [5. Les commandes utiles à DBT](#5-les-commandes-utiles-à-dbt)
- [6. Documentation DBT](#6-documentation-dbt)

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

## 3. Utilisation des variables dans DBT

Dans un projet **DBT**, il est possible d'utiliser des **variables personnalisées** pour rendre tes modèles plus dynamiques et configurables. Ces variables sont définies dans le fichier `dbt_project.yml` ou passées en ligne de commande, puis utilisées dans les modèles SQL avec la fonction `var()`.

### 3.1 Définir une variable dans le fichier `dbt_project.yml`

Tu peux définir des variables globales dans la section `vars:` du fichier `dbt_project.yml`.

Par exemple :
```yaml
# dbt_project.yml
name: my_first_project
version: '1.0'
config-version: 2

vars:
  date_limite: '2022-01-01'
```

Ici, on définit une variable `date_limite` contenant une date en texte.

### 3.2 Utiliser une variable dans un modèle SQL

Une fois définie, tu peux réutiliser cette variable dans un modèle grâce à la fonction `var()` de Jinja :

```sql
-- models/transform_users_variable.sql
SELECT 
    id,
    name,
    created_at
FROM {{ source('raw_data', 'users') }}
WHERE created_at > '{{ var("date_limite") }}'
```

⚠️ **Attention** : la fonction `var()` retourne une chaîne, donc si tu veux l’utiliser dans une clause SQL, pense à bien gérer les guillemets si c’est une date, une chaîne, ou autre type littéral.

### 3.3 Redéfinir une variable depuis la ligne de commande

Il est également possible de **surcharger une variable** depuis la ligne de commande avec l’option `--vars`, ce qui permet d’exécuter un modèle avec des paramètres différents sans avoir à modifier le fichier `dbt_project.yml`.

Exemple :
```bash
dbt run --models transform_users_variable --vars '{date_limite: "2023-01-01"}'
```

Cela permet d’exécuter le modèle avec une autre date limite sans modifier le projet.

### 3.4 Utilisation avancée : variables conditionnelles

Tu peux aussi ajouter de la logique conditionnelle autour des variables dans ton modèle, par exemple :

```sql
{% set seuil_date = var("date_limite", "2020-01-01") %}

SELECT 
    id,
    name,
    created_at
FROM {{ source('raw_data', 'users') }}
WHERE created_at > '{{ seuil_date }}'
```

Ici, si la variable `date_limite` n'est pas fournie, elle prend la valeur par défaut `"2020-01-01"`.

---  
Grâce à l'utilisation des **variables**, tu rends tes modèles plus souples, dynamiques et adaptables à différents cas d'usage sans dupliquer du code ou modifier manuellement les fichiers sources.

### 3.5 Définir dynamiquement un schéma ou un nom de table avec des variables

DBT permet également de définir dynamiquement le **schéma** ou le **nom d'une table** en utilisant des variables dans les configurations des modèles. Cela peut être utile pour gérer plusieurs environnements (dev, prod, staging) ou projets en parallèle.

#### Exemple : utiliser une variable pour le schéma de destination

Tu peux définir une variable `target_schema` dans `dbt_project.yml` :
```yaml
vars:
  target_schema: "dev_schema"
```

Puis, dans un modèle SQL, configurer le schéma cible avec cette variable :
```sql
-- models/transform_users_variable.sql
{{ config(
    schema=var("target_schema", "public")
) }}

SELECT
    id,
    name,
    created_at
FROM {{ source('raw_data', 'users') }}
```

Cela permettra à DBT de créer la table ou la vue dans le schéma `dev_schema`.

#### Exemple : utiliser une variable pour le nom de table

Tu peux également personnaliser dynamiquement le nom de la table cible :
```sql
-- models/transform_dynamic_table.sql
{% set table_suffix = var("suffix", "default") %}

{{ config(
    alias="transform_users_" ~ table_suffix
) }}

SELECT
    id,
    name,
    created_at
FROM {{ source('raw_data', 'users') }}
```

Puis tu exécutes le modèle avec :
```bash
dbt run --models transform_dynamic_table --vars '{suffix: "2024"}'
```

Le modèle générera alors une table nommée `transform_users_2024`.

---

Avec cette approche, tu peux adapter dynamiquement les schémas, noms de table ou d'autres éléments en fonction du contexte ou des paramètres passés, ce qui rend tes projets DBT plus flexibles et facilement déployables dans différents environnements.

## 4. Utiliser les macros dans DBT

Les **macros** sont des blocs de code réutilisables écrits en Jinja, qui permettent de factoriser des portions SQL ou de la logique complexe dans tes projets DBT. Elles sont particulièrement utiles pour automatiser des tâches répétitives, encapsuler des règles métier ou générer dynamiquement des requêtes.

### 4.1 Built-in macros (macros intégrées)

DBT fournit un ensemble de **macros intégrées** prêtes à l’emploi. Ces macros couvrent des besoins courants comme l’accès à l’environnement, les chemins de dépendance ou la gestion du contexte.

Voici quelques exemples utiles :

- `{{ ref('model_name') }}` : fait référence à un autre modèle DBT.
- `{{ source('schema', 'table') }}` : référence une source de données déclarée.
- `{{ config(...) }}` : permet de configurer dynamiquement les paramètres d’un modèle.
- `{{ this }}` : fait référence au modèle courant (nom complet `database.schema.table`).
- `{{ target.name }}` : donne le nom de l’environnement (`dev`, `prod`, etc.).
- `{{ run_started_at }}` : retourne la date/heure de début d'exécution du run DBT.

### 4.2 Créer des custom macros

Tu peux définir tes propres macros dans le dossier `macros/` de ton projet. Chaque fichier `.sql` peut contenir une ou plusieurs macros personnalisées.

Exemple :

```sql
-- macros/format_date.sql
{% macro format_date(column_name) %}
    TO_CHAR({{ column_name }}, 'YYYY-MM-DD')
{% endmacro %}
```

Et son utilisation dans un modèle :

```sql
SELECT
    id,
    name,
    {{ format_date('created_at') }} AS created_at_formatted
FROM {{ source('raw_data', 'users') }}
```

### 4.3 Utiliser des macros dans des boucles et conditions

Jinja permet aussi d’utiliser des **structures de contrôle**, comme des boucles ou des conditions, dans les macros :

#### Exemple avec boucle :

```sql
-- macros/select_columns.sql
{% macro select_columns(columns) %}
    {% for col in columns %}
        {{ col }}{% if not loop.last %}, {% endif %}
    {% endfor %}
{% endmacro %}
```

Utilisation :

```sql
SELECT {{ select_columns(['id', 'name', 'created_at']) }}
FROM {{ source('raw_data', 'users') }}
```

#### Exemple avec condition :

```sql
{% macro filter_by_env(column_name) %}
    {% if target.name == 'prod' %}
        {{ column_name }} IS NOT NULL
    {% else %}
        TRUE
    {% endif %}
{% endmacro %}
```

### 4.4 Le package `dbt_utils`

Le package [**dbt_utils**](https://github.com/dbt-labs/dbt-utils) est une collection communautaire de macros prêtes à l’emploi pour accélérer le développement DBT. Il contient des fonctions utiles comme :

- `dbt_utils.get_column_values()`
- `dbt_utils.star()`
- `dbt_utils.date_spine()`

#### Installation de dbt_utils

1. Dans le fichier `packages.yml` de ton projet, ajoute :

```yaml
packages:
  - package: dbt-labs/dbt_utils
    version: 1.1.1  # ou la dernière version stable
```

2. Exécute ensuite la commande :

```bash
dbt deps
```

Cela télécharge les packages dans le dossier `dbt_modules`.

#### Exemple d'utilisation :

```sql
SELECT *
FROM {{ dbt_utils.union_relations(relations=[ref('table1'), ref('table2')]) }}
```

---

Les macros, qu'elles soient intégrées, personnalisées ou issues de packages comme `dbt_utils`, sont un outil puissant pour améliorer la réutilisabilité, la lisibilité et la maintenabilité de tes projets DBT.

## 5. Les commandes utiles à DBT

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

- Installation de packages :
Pour utiliser des packages communs entre plusieurs projet, tu peux importer le fichier packages.yml dans le projet existant et utiliser la commande : `dbt deps`
Cela rendra son utilisation possible. Il est donc également possible de télécharger des packages open sources, comme vu précédemment.  

## 6. Documentation DBT

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
- La documentation de **Pluralsight** (accès payant).

