# 🏦 DataBank — Pipeline de Données Financières avec PostgreSQL

## 📋 Description

**DataBank** est un projet de pipeline de données financières qui automatise le chargement, la normalisation et l'analyse de données bancaires dans une base de données relationnelle PostgreSQL. Il permet de transformer un fichier CSV brut (`financecore_clean.csv`) en un schéma structuré avec des tables normalisées, et d'en extraire des KPIs métier via des requêtes SQL avancées.

---

## 🗂️ Structure du Projet

```
DataBank/
├── DataBank.ipynb          # Notebook principal (pipeline complet)
├── data/
│   └── financecore_clean.csv  # Fichier source des données financières
├── .env                    # Variables d'environnement (connexion DB)
└── README.md
```

---

## ⚙️ Prérequis

- Python 3.8+
- PostgreSQL (en cours d'exécution localement ou à distance)
- Les bibliothèques Python suivantes :

```bash
pip install pandas sqlalchemy psycopg2-binary python-dotenv
```

---

## 🔐 Configuration de l'environnement

Créez un fichier `.env` à la racine du projet avec les informations de connexion à votre base de données PostgreSQL :

```env
DB_USER=votre_utilisateur
DB_PASSWORD=votre_mot_de_passe
DB_HOST=localhost
DB_PORT=5432
DB_NAME=nom_de_la_base
```

> ⚠️ **Ne jamais versionner le fichier `.env`.** Ajoutez-le dans votre `.gitignore`.

---

## 🗄️ Schéma Relationnel

Le projet crée les tables suivantes dans PostgreSQL :

| Table          | Description                                      |
|----------------|--------------------------------------------------|
| `segments`     | Segments clients (ex. : Premium, Standard...)    |
| `agences`      | Agences bancaires                                |
| `produits`     | Produits financiers                              |
| `clients`      | Clients avec score de crédit et segment          |
| `comptes`      | Comptes bancaires liés aux clients et agences    |
| `transactions` | Transactions financières (cœur du modèle)        |

### Relations clés
- `clients` → `segments` (N:1)
- `comptes` → `clients`, `agences` (N:1)
- `transactions` → `clients`, `agences`, `produits`, `comptes` (N:1)

---

## 🚀 Utilisation

Lancez le notebook `DataBank.ipynb` dans l'ordre des sections :

### 1. Connexion & Chargement des données
Chargement du fichier CSV source et connexion à PostgreSQL via SQLAlchemy.

### 2. Création du schéma relationnel
Suppression et recréation des tables avec les clés primaires, étrangères et les contraintes d'intégrité.

### 3. Insertion & Normalisation des données
- Insertion des tables de dimensions (segments, agences, produits)
- Insertion des clients (dédoublonnage par `client_id`)
- Insertion des transactions (dédoublonnage par `transaction_id`)
- Insertion des comptes (agrégation du solde par client et agence)
- Mise à jour du lien `compte_id` dans les transactions

### 4. Analyse SQL Avancée & KPIs Métier
Exemples de requêtes analytiques :
- Moyenne mensuelle des transactions par agence
- Clients avec un solde inférieur à la moyenne
- Taux de défaut par segment client

### 5. Vues analytiques (Dashboard KPIs)
Création de vues SQL réutilisables :
- `v_kpi_global_transactions` : total, somme et moyenne des montants
- `v_kpi_statut_transactions` : répartition par statut de transaction

---

## 📊 Exemples de KPIs

```sql
-- Moyenne mensuelle des transactions par agence
SELECT 
    a.nom_agence, 
    EXTRACT(MONTH FROM t.date_transaction) AS mois,
    ROUND(AVG(t.montant_eur), 2) AS moyenne_mensuelle
FROM transactions t
JOIN agences a ON t.id_agence = a.id_agence
GROUP BY a.nom_agence, mois
ORDER BY a.nom_agence, mois;

-- Taux de défaut par segment
SELECT 
    s.nom_segment,
    COUNT(*) AS total,
    SUM(CASE WHEN t.statut = 'defaut' THEN 1 ELSE 0 END) AS nb_defauts,
    ROUND(SUM(CASE WHEN t.statut = 'defaut' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS taux_defaut
FROM transactions t
JOIN clients cl ON t.client_id = cl.client_id
JOIN segments s ON cl.id_segment = s.id_segment
GROUP BY s.nom_segment;
```

---

## 🛡️ Bonnes pratiques intégrées

- ✅ Chargement incrémental (évite les doublons à chaque exécution)
- ✅ Utilisation de variables d'environnement pour les credentials
- ✅ Transactions atomiques avec `engine.begin()`
- ✅ Normalisation des données (3NF)
- ✅ Vues SQL pour faciliter les analyses BI

---

## 👤 Auteur

Projet développé dans le cadre d'un brief de Data Engineering / Analyse Financière.