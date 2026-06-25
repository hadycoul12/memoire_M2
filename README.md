# Scoring prédictif du risque d'annulation des réservations

Projet de mémoire — **Mastère 2 DIA Paris** · Alternance chez **Maeva** (Groupe Pierre & Vacances).

Modèle d'apprentissage supervisé estimant, dès la réservation, le **risque d'annulation** d'un dossier, à partir de ses caractéristiques structurelles (canal de vente, condition d'annulation Flex) et du comportement email du client (CRM).

---

## Problématique

Dans quelle mesure peut-on prédire, dès la réservation, le risque d'annulation d'un dossier à partir de ses caractéristiques structurelles (canal, condition Flex) ? Et au-delà de ces variables déterminantes mais déjà connues de l'entreprise, l'engagement email apporte-t-il un pouvoir prédictif additionnel permettant d'identifier les **annulations évitables** — celles de clients non couverts qu'une action de rétention ciblée pourrait prévenir ?

---

## Données

Trois tables de l'entrepôt BigQuery (projet `groupe-lfdnas`) :

| Source | Granularité | Apport |
|--------|-------------|--------|
| `dossiers_marketing` | 1 ligne / dossier | Cible, canal, condition d'annulation, contexte séjour |
| `behavior` (Batch) | 1 ligne / événement email | Engagement email (clics, ouvertures, désabonnements) |
| `produit` | n lignes / dossier | Détection de l'option « Tarif Flex » |

- **Cible** : `y_annulation` = 1 si le dossier est annulé, 0 sinon.
- **Périmètre** : conditions d'annulation propres à Maeva (Flexi, Flexi+, NoFlex, NoFlex <J30) ; dossiers RESA/ANNULE ; séjour déjà commencé (anti-censure).
- **Anti-fuite** : le comportement email n'est agrégé que sur les **90 jours précédant** la réservation.
- **Volume** : ~297 600 dossiers, taux d'annulation 7,13 %.

> ⚠️ Les données contiennent des informations clients : elles **ne sont jamais versionnées** (voir `.gitignore`). Le travail en local se fait sur un CSV **anonymisé** (colonne email supprimée).

---

## Structure du projet

```
memoire_annulation/
├── data/                          # CSV (ignoré par Git — RGPD)
│   └── dataset_annulation_anon.csv
├── figures/                       # graphiques générés par les notebooks
├── models/                        # modèles sérialisés (ignoré par Git)
├── sql/
│   └── v_dataset_annulation.sql   # construction de la vue BigQuery
├── export_bigquery.py             # export de la vue vers CSV
├── eda_annulation.ipynb           # analyse exploratoire
├── modelisation_annulation.ipynb  # feature engineering + 3 modèles + SHAP
├── requirements.txt
├── .gitignore
└── README.md
```

---

## Installation

```bash
# 1. Cloner le repo
git clone <url-du-repo-prive>
cd memoire_annulation

# 2. Créer l'environnement virtuel
python -m venv .venv
# Windows :
.venv\Scripts\activate
# Mac / Linux :
source .venv/bin/activate

# 3. Installer les dépendances
pip install -r requirements.txt
```

---

## Utilisation

### 1. Export des données (sur l'environnement professionnel uniquement)

L'accès BigQuery se fait via l'authentification utilisateur (aucune clé secrète) :

```bash
gcloud auth application-default login
python export_bigquery.py
```

Produit `data/dataset_annulation.csv`. Une version anonymisée (`_anon.csv`) est créée pour le travail en local.

### 2. Analyse exploratoire

Ouvrir `eda_annulation.ipynb` dans VSCode (extensions Python + Jupyter), sélectionner le kernel `.venv`, exécuter les cellules. Les graphiques sont sauvegardés dans `figures/`.

### 3. Modélisation

Ouvrir `modelisation_annulation.ipynb`. Au programme :
- feature engineering (sélection, limitation de cardinalité, pipeline de préprocessing) ;
- trois modèles comparés : **Régression Logistique → Random Forest → XGBoost** ;
- évaluation orientée déséquilibre (**PR-AUC**, rappel, courbes ROC & Précision-Rappel) ;
- interprétabilité **SHAP** ;
- **stratégie en deux temps** : modèle global puis modèle ciblé sur le sous-ensemble CRM.

---

## Méthodologie en bref

1. **Modèle global** (tous dossiers) — porté par les variables structurelles (`cond_annulation`, `canal`). Livrable de scoring opérationnel.
2. **Analyse ciblée** (`est_dans_crm = 1`) — sur les contacts disposant d'un historique email, pour mesurer l'apport réel de l'engagement CRM (présenté comme exploratoire, l'historique email Batch ne couvrant que 7 mois).

---

## Stack technique

`Python` · `pandas` · `scikit-learn` · `XGBoost` · `SHAP` · `BigQuery` · `VSCode`

---

## Limites assumées

- Historique email limité à 7 mois (≈ 9,3 % des dossiers couverts).
- Biais de sélection du sous-ensemble CRM (population engagée).
- Variables structurelles dominantes : la valeur du modèle réside dans la priorisation fine et l'identification des annulations évitables, non dans la redécouverte de règles métier connues.

---

*Auteur : Hady Coulibaly — Mastère 2 DIA Paris · Données arrêtées à juin 2026.*
