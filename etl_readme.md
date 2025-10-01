# 🚀 PROJET 1 : Pipeline ETL Classique

## 📋 Vue d'ensemble

Ce projet implémente un **pipeline ETL (Extract-Transform-Load) classique** avec orchestration Airflow, permettant d'extraire des données depuis PostgreSQL et des fichiers CSV, de les transformer avec Pandas, puis de les charger dans un Data Warehouse structuré en schéma dimensionnel.

### 🎯 Objectifs pédagogiques
- Comprendre l'approche ETL traditionnelle
- Orchestrer des pipelines avec Apache Airflow
- Implémenter un modèle dimensionnel (schéma en étoile)
- Visualiser les données avec Apache Superset

---

## 🏗️ Architecture

```
┌─────────────────┐
│ PostgreSQL      │──┐
│ Source (5433)   │  │
└─────────────────┘  │
                     │
┌─────────────────┐  │    ┌──────────────────┐
│ Fichiers CSV/   │──┼───▶│   Apache Airflow │
│ Excel (data/)   │  │    │   Orchestration  │
└─────────────────┘  │    └──────────────────┘
                     │             │
                     │             ▼
                     │    ┌──────────────────┐
                     └───▶│  Transformation  │
                          │  (Pandas/Python) │
                          └──────────────────┘
                                   │
                                   ▼
                          ┌──────────────────┐
                          │ PostgreSQL DWH   │
                          │ (5434)           │
                          │ ├─ staging       │
                          │ └─ marts         │
                          └──────────────────┘
                                   │
                        ┌──────────┴──────────┐
                        │                     │
                   ┌────▼────┐         ┌─────▼─────┐
                   │Superset │         │  Adminer  │
                   │ (8088)  │         │  (8081)   │
                   └─────────┘         └───────────┘
```

---

## 📦 Composants du Projet

### Services Docker

| Service | Port | Description | Credentials |
|---------|------|-------------|-------------|
| **postgres-source** | 5433 | Base de données source (prod simulée) | user: `source_user` / pass: `source_pass` |
| **postgres-dwh** | 5434 | Data Warehouse (staging + marts) | user: `dwh_user` / pass: `dwh_pass` |
| **airflow-webserver** | 8080 | Interface web Airflow | user: `admin` / pass: `admin` |
| **airflow-scheduler** | - | Planificateur de tâches Airflow | - |
| **superset** | 8088 | Outil de visualisation | user: `admin` / pass: `admin` |
| **adminer** | 8081 | Interface DB (comme phpMyAdmin) | - |

---

## 📂 Structure des Fichiers

```
projet1-etl-classic/
│
├── docker-compose.yml          # Configuration Docker Compose
├── README.md                   # Ce fichier
│
├── sql/
│   ├── pg_source_init.sql      # Init base source avec données de test
│   └── pg_dwh_init.sql         # Init DWH (staging + marts + dim_date)
│
├── airflow/
│   └── dags/
│       └── dag_etl_classic.py  # Pipeline ETL orchestré
│
├── data/
│   └── external_sales.csv      # Fichier CSV externe (créé auto)
│
└── docs/
    └── architecture.png        # Schéma d'architecture
```

---

## 🚀 Installation et Démarrage

### Prérequis
- **Windows 10/11** avec WSL2 activé
- **Docker Desktop** installé et démarré
- **Au moins 8 GB de RAM** disponible pour Docker

### Étape 1 : Créer la structure des dossiers

```bash
# Ouvrir PowerShell ou CMD
mkdir projet1-etl-classic
cd projet1-etl-classic

# Créer les sous-dossiers
mkdir sql
mkdir airflow\dags
mkdir airflow\logs
mkdir airflow\plugins
mkdir airflow\scripts
mkdir data
```

### Étape 2 : Copier les fichiers

1. **docker-compose.yml** → racine du projet
2. **pg_source_init.sql** → dossier `sql/`
3. **pg_dwh_init.sql** → dossier `sql/`
4. **dag_etl_classic.py** → dossier `airflow/dags/`

### Étape 3 : Configurer les permissions (Windows)

```bash
# Créer un fichier .env à la racine
echo AIRFLOW_UID=50000 > .env
```

### Étape 4 : Démarrer les services

```bash
# Lancer tous les conteneurs
docker compose up -d

# Vérifier que tout est démarré
docker compose ps
```

⏱️ **Attendez 2-3 minutes** que tous les services soient prêts (surtout Airflow).

### Étape 5 : Vérifier les logs

```bash
# Logs Airflow
docker compose logs airflow-webserver -f

# Logs PostgreSQL Source
docker compose logs postgres-source

# Logs PostgreSQL DWH
docker compose logs postgres-dwh
```

---

## 🎮 Utilisation

### 1️⃣ Accéder à Airflow

1. Ouvrir un navigateur : **http://localhost:8080**
2. Se connecter :
   - Username : `admin`
   - Password : `admin`
3. Activer le DAG `etl_classic_pipeline`
4. Cliquer sur le bouton "▶️ Play" pour lancer manuellement

### 2️⃣ Vérifier les données dans Adminer

1. Ouvrir : **http://localhost:8081**
2. Se connecter à la **base source** :
   - Système : `PostgreSQL`
   - Serveur : `postgres-source`
   - Utilisateur : `source_user`
   - Mot de passe : `source_pass`
   - Base : `source_db`

3. Explorer les tables :
   - `public.customers` (15 clients)
   - `public.products` (15 produits)
   - `public.orders` (20 commandes)

4. Se connecter au **DWH** :
   - Serveur : `postgres-dwh`
   - Utilisateur : `dwh_user`
   - Mot de passe : `dwh_pass`
   - Base : `dwh_db`

5. Vérifier les données :
   - Schéma `staging.*` (données brutes)
   - Schéma `marts.*` (modèle dimensionnel)

### 3️⃣ Exécuter des requêtes SQL

```bash
# Se connecter au DWH
docker exec -it etl_postgres_dwh psql -U dwh_user -d dwh_db

# Exemples de requêtes
\dt staging.*        # Lister les tables de staging
\dt marts.*          # Lister les tables marts

# Voir les ventes par pays
SELECT * FROM marts.vw_sales_by_country LIMIT 10;

# Voir les KPIs
SELECT * FROM marts.vw_kpi_summary;

# Compter les ventes
SELECT COUNT(*) FROM marts.fact_sales;

# Quitter
\q
```

### 4️⃣ Visualiser avec Superset

1. Ouvrir : **http://localhost:8088**
2. Se connecter :
   - Username : `admin`
   - Password : `admin`

3. **Ajouter une connexion à la base** :
   - Settings → Database Connections → + Database
   - Nom : `DWH Analytics`
   - SQLAlchemy URI : 
     ```
     postgresql://dwh_user:dwh_pass@postgres-dwh:5432/dwh_db
     ```

4. **Créer un Dataset** :
   - Data → Datasets → + Dataset
   - Database : `DWH Analytics`
   - Schema : `marts`
   - Table : `vw_sales_by_country`

5. **Créer un Chart** :
   - Charts → + Chart
   - Sélectionner le dataset
   - Choisir un type (Bar Chart, Time Series, etc.)

---

## 📊 Pipeline ETL - Détails

### Étapes du DAG

1. **extract_from_source** : Extrait depuis PostgreSQL source
2. **extract_from_files** : Extrait depuis CSV/Excel
3. **transform_data** : Transforme avec Pandas (nettoyage, calculs)
4. **load_to_staging** : Charge dans `staging.*`
5. **load_dimensions** : Peuple `dim_customer`, `dim_product`, `dim_date`
6. **load_facts** : Charge `fact_sales`
7. **validate_data** : Vérifie la qualité des données

### Transformations appliquées

- **Nettoyage** : emails en minuscules, espaces supprimés
- **Enrichissement** : création de `full_name` pour les clients
- **Agrégation** : calculs de `net_amount`, `date_key`
- **Validation** : vérification des nulls, cohérence des clés

---

## 🧪 Tests et Validation

### Test 1 : Vérifier l'extraction

```bash
# Lister les fichiers extraits
docker exec -it etl_airflow_webserver ls -lh /opt/airflow/data/
```

### Test 2 : Comparer les totaux

```sql
-- Dans la base source
SELECT COUNT(*) FROM public.orders;

-- Dans le DWH
SELECT COUNT(*) FROM marts.fact_sales;
```

### Test 3 : Vérifier les vues

```sql
-- Ventes par catégorie
SELECT * FROM marts.vw_sales_by_category;

-- Top 5 clients
SELECT 
    c.full_name,
    COUNT(f.sale_key) as nb_orders,
    SUM(f.net_amount) as total_spent
FROM marts.fact_sales f
JOIN marts.dim_customer c ON f.customer_key = c.customer_key
GROUP BY c.full_name
ORDER BY total_spent DESC
LIMIT 5;
```

---

## 🔧 Commandes Utiles

### Gestion Docker

```bash
# Démarrer
docker compose up -d

# Arrêter
docker compose stop

# Supprimer (⚠️ efface les données)
docker compose down -v

# Redémarrer un service
docker compose restart airflow-scheduler

# Voir les logs en temps réel
docker compose logs -f airflow-webserver
```

### Debugging Airflow

```bash
# Tester une tâche manuellement
docker exec -it etl_airflow_webserver airflow tasks test etl_classic_pipeline extract_from_source 2024-01-01

# Vérifier la configuration
docker exec -it etl_airflow_webserver airflow config list

# Réinitialiser la DB Airflow
docker exec -it etl_airflow_webserver airflow db reset
```

### Accès aux bases de données

```bash
# PostgreSQL Source
docker exec -it etl_postgres_source psql -U source_user -d source_db

# PostgreSQL DWH
docker exec -it etl_postgres_dwh psql -U dwh_user -d dwh_db
```

---

## 🎓 Concepts Pédagogiques

### Pourquoi ETL (et pas ELT) ?

✅ **Avantages ETL** :
- Données nettoyées avant stockage
- Moins d'espace disque requis
- Contrôle total sur les transformations
- Idéal pour les bases de données traditionnelles

❌ **Inconvénients ETL** :
- Moins flexible (transformations figées dans le code)
- Difficile de revenir aux données brutes
- Requiert plus de développement Python

### Modèle Dimensionnel (Schéma en Étoile)

```
        ┌─────────────┐
        │ dim_date    │
        │             │
        └──────┬──────┘
               │
        ┌──────▼──────┐
        │             │
┌───────┤ fact_sales  ├────────┐
│       │             │        │
│       └──────┬──────┘        │
│              │               │
▼              ▼               ▼
┌──────────┐ ┌──────────┐ ┌──────────┐
│dim_      │ │dim_      │ │dim_      │
│customer  │ │product   │ │...       │
└──────────┘ └──────────┘ └──────────┘
```

**Dimensions** : Contexte (Qui ? Quoi ? Quand ?)
**Faits** : Mesures quantitatives (Combien ?)

---

## 🐛 Dépannage

### Problème : Airflow ne démarre pas

```bash
# Vérifier les logs
docker compose logs airflow-init

# Réinitialiser
docker compose down -v
docker compose up -d
```

### Problème : DAG ne s'affiche pas

```bash
# Vérifier que le fichier est bien présent
docker exec -it etl_airflow_webserver ls -l /opt/airflow/dags/

# Vérifier les erreurs de syntaxe
docker exec -it etl_airflow_webserver python /opt/airflow/dags/dag_etl_classic.py
```

### Problème : Connexion refusée PostgreSQL

```bash
# Vérifier que le conteneur tourne
docker compose ps postgres-source

# Tester la connexion
docker exec -it etl_postgres_source pg_isready -U source_user
```

### Problème : Port déjà utilisé

```bash
# Changer les ports dans docker-compose.yml
# Par exemple : "8080:8080" → "8090:8080"
```

---

## 📈 Améliorations Possibles

1. **Incrémental Loading** : Ne charger que les nouvelles données
2. **Gestion des erreurs** : Notification par email en cas d'échec
3. **Data Quality** : Intégrer Great Expectations
4. **Performance** : Utiliser PySpark pour gros volumes
5. **Monitoring** : Ajouter Prometheus + Grafana
6. **CI/CD** : Tests automatiques avec pytest

---

## 📚 Ressources

- [Documentation Airflow](https://airflow.apache.org/docs/)
- [Documentation Superset](https://superset.apache.org/docs/intro)
- [Modèle dimensionnel - Kimball](https://www.kimballgroup.com/)
- [PostgreSQL Best Practices](https://wiki.postgresql.org/wiki/Don%27t_Do_This)

---

## 📝 Notes Importantes

⚠️ **Ce projet est à des fins pédagogiques** :
- Pas de sécurité renforcée (mots de passe en clair)
- Pas de gestion des secrets (utiliser Vault en prod)
- Pas de haute disponibilité
- Données de test uniquement

💡 **Pour la production** :
- Utiliser des services managés (AWS RDS, etc.)
- Implémenter des secrets managers
- Ajouter des tests unitaires et d'intégration
- Mettre en place du monitoring
- Utiliser Kubernetes pour l'orchestration

---

## ✅ Checklist de Validation

- [ ] Tous les conteneurs démarrent sans erreur
- [ ] Airflow accessible sur http://localhost:8080
- [ ] DAG visible et activable
- [ ] Pipeline s'exécute sans erreur
- [ ] Données présentes dans `marts.fact_sales`
- [ ] Vues fonctionnelles (`vw_sales_by_country`, etc.)
- [ ] Superset accessible et connecté au DWH
- [ ] Adminer permet de visualiser les données

---

## 🎯 Prochaine Étape : Projet 2 (ELT Moderne)

Une fois ce projet maîtrisé, passez au **Projet 2** qui implémente une approche **ELT moderne** avec :
- **Airbyte** pour l'ingestion
- **dbt** pour les transformations SQL
- **Prefect** pour l'orchestration
- **Metabase** pour la visualisation

**Différences clés** :
- Load d'abord, Transform après (dans le DWH)
- Transformations déclaratives en SQL
- Plus de flexibilité et agilité

---

## 👨‍💻 Auteur

Projet pédagogique - Data Engineering Learning Path

Pour toute question ou amélioration, n'hésitez pas à ouvrir une issue !

---

**Bon apprentissage ! 🚀**