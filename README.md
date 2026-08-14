# Modélisation et analyse exploratoire, scoring du risque d'annulation

Ce dossier regroupe l'ensemble du travail data du mémoire : l'analyse exploratoire des données, la construction et l'évaluation du modèle de prédiction, ainsi que l'étude de l'apport des signaux d'engagement email.

Le projet a été réalisé dans le cadre de mon mémoire de Master 2 Data et Intelligence Artificielle (Nexa Digital School), en alternance chez Maeva (groupe Pierre et Vacances Center Parcs).

L'objectif est d'estimer, dès la réservation, la probabilité qu'un dossier soit annulé, à partir de ses caractéristiques structurelles (canal de vente, condition tarifaire, anticipation) et du comportement email du client.

## Prérequis

- **Python 3.10 ou une version plus récente**
- **Jupyter**, ou **Visual Studio Code** avec les extensions Python et Jupyter (recommandé)
- Environ **1 Go d'espace disque** pour l'environnement virtuel et les dépendances
- Les fichiers de données dans le dossier `data/` (voir la section Données)

Pour vérifier votre version de Python :

```bash
python --version
```

## Installation

### 1. Récupérer le projet et se placer dans le dossier

```bash
cd memoire_M2
```

### 2. Créer et activer un environnement virtuel

```bash
python -m venv .venv
```

```bash
# Windows (PowerShell)
.venv\Scripts\activate

# macOS ou Linux
source .venv/bin/activate
```

### 3. Installer les dépendances

```bash
pip install -r requirements.txt
```

Les versions sont volontairement figées (notamment scikit-learn 1.5.0), afin de garantir que le modèle sauvegardé se recharge sans avertissement de compatibilité. L'installation comprend aussi `ipykernel`, nécessaire pour exécuter les notebooks.

## Authentification

Les notebooks s'exécutent localement et ne demandent aucun identifiant. La connexion par mot de passe (`maeva` / `maeva2026`) concerne uniquement l'application Streamlit, livrée séparément.

## Exécution des notebooks

### Choisir le bon noyau (important)

Dans VSCode, ouvrez un notebook puis, en haut à droite, cliquez sur le sélecteur de noyau et choisissez l'environnement **`.venv`** que vous venez de créer. C'est l'erreur la plus fréquente : un noyau pointant vers un autre environnement provoque des `ModuleNotFoundError` alors que tout est bien installé.

### Ordre d'exécution

Les notebooks se lancent dans cet ordre, car chacun produit un fichier utilisé par le suivant :

| Ordre | Notebook | Rôle | Produit |
|---|---|---|---|
| 1 | `EDA.ipynb` | Analyse exploratoire, nettoyage, feature engineering exploratoire | `data/dataset_annulation_clean.csv`, figures |
| 2 | `modele_final.ipynb` | Comparaison de trois modèles, optimisation de XGBoost, recalibration des probabilités, cadre de décision | `models/xgboost_optimise.joblib`, `models/calibrateur.joblib`, `data/dataset_entrainement.csv` |
| 3 | `modele_email_exploratoire.ipynb` | Étude comparative de l'apport des signaux email (protocole A vs B, test de significativité) | figures |

Pour lancer un notebook complet, utilisez la commande **Run All** une fois le noyau sélectionné.

## Données

Les notebooks lisent leurs données dans le dossier `data/`. Le nettoyage part d'une version anonymisée (sans email ni identifiant direct).

| Fichier | Contenu |
|---|---|
| `data/dataset_anon.csv` | Jeu brut anonymisé, point de départ de l'analyse exploratoire |
| `data/dataset_annulation_clean.csv` | Jeu nettoyé, produit par `EDA.ipynb` |
| `data/dataset_entrainement.csv` | Jeu final avec features dérivées, produit par `modele_final.ipynb` |
| `data/sample_dataset.csv` | Échantillon léger pour la démonstration |

Le volume analysé est de 310 862 dossiers, pour un taux d'annulation de 7,27 pour cent.

Ces fichiers contiennent des données commerciales réelles de Maeva. Ils sont couverts par un accord de confidentialité et ne doivent pas être diffusés en dehors du cadre de l'évaluation académique.

## Structure du dossier

```
memoire_M2/
├── EDA.ipynb                        Analyse exploratoire et nettoyage
├── modele_final.ipynb                  Modélisation, optimisation, calibration, décision
├── modele_email_exploratoire.ipynb  Étude de l'apport des signaux email
├── requirements.txt                 Dépendances Python
├── data/                            Jeux de données (confidentiels)
├── models/                          Modèle et calibrateur sérialisés
└── figures/                         Graphiques générés par les notebooks
```

## Principaux résultats

- Le modèle retenu est un XGBoost intégré dans un pipeline scikit-learn (préprocessing embarqué), optimisé par recherche aléatoire avec validation croisée.
- Sur le jeu de test, il obtient une aire sous la courbe ROC de 0,76 et une aire sous la courbe précision rappel de 0,26, soit environ 3,6 fois la référence aléatoire.
- Les probabilités brutes étant trop élevées à cause de la pondération de la classe minoritaire, elles sont recalibrées par régression isotonique. Le score de Brier passe ainsi de 0,17 à 0,06.
- L'étude comparative montre que les signaux d'engagement email apportent un gain faible et non significatif une fois le modèle structurel complet. Les variables structurelles (assurance, fidélité, canal, anticipation) restent déterminantes.

## Stack technique

Python, pandas, scikit-learn, XGBoost, SHAP, imbalanced-learn, Matplotlib, seaborn.

## Auteur

**Hady Coulibaly**, Master 2 Data et Intelligence Artificielle.

Projet académique. Les données réelles restent soumises à l'accord de confidentialité Maeva.
