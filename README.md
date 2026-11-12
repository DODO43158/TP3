# TP3 : Étude de cas Data et Big Data

Ce dépôt contient deux notebooks qui suivent le même cycle de projet data :

**Data → EDA → Prétraitement → Transformation → ML → Évaluation → Déploiement**

| Étape du TP | Notebook | Sujet | Outils |
|---|---|---|---|
| 1. Data | `étudesdecas(Data).ipynb` | Prédiction des retards de vols aux États-Unis (2015) | pandas, scikit-learn |
| 2. Big Data | `étudesdecas(BigData).ipynb` | Prédiction du diabète (enquête de santé BRFSS) | PySpark, Spark MLlib |

---

## Étape 1 : Data, retards de vols (pandas + scikit-learn)

**Données :** [2015 Flight Delays and Cancellations](https://www.kaggle.com/datasets/usdot/flight-delays) (U.S. Department of Transportation), environ 5,8 millions de vols intérieurs.

**Objectif :** prédire si un vol arrivera avec **plus de 15 minutes de retard**, à partir d'informations connues avant le départ (compagnie, mois, jour, heure, distance, durée prévue).

**Démarche :**
- **EDA** : taux de retard par compagnie, par mois et selon l'heure de départ.
- **Prétraitement** : suppression des vols annulés ou déroutés et des valeurs manquantes.
- **Transformation** : création de la cible `RETARD`, encodage de la compagnie (`get_dummies`), échantillon de 300 000 vols. Le retard au départ n'est pas utilisé, pour éviter une fuite d'information.
- **ML** : régression logistique et arbre de décision (`class_weight="balanced"` pour compenser le déséquilibre des classes).
- **Évaluation** : rapport de classification (précision, rappel, F1) et matrices de confusion.
- **Déploiement** : sauvegarde du modèle avec `joblib` et fonction `predire_retard(...)`.

---

## Étape 2 : Big Data, diabète (PySpark)

**Données :** [Behavioral Risk Factor Surveillance System](https://www.kaggle.com/datasets/cdc/behavioral-risk-factor-surveillance-system) (CDC), enquêtes de santé auprès d'environ 400 000 personnes par an, plus de 300 colonnes.

**Objectif :** prédire si une personne est **diabétique** à partir de son âge, de son sexe, de son IMC, de sa tension, de son cholestérol, du tabac, de l'activité physique et de son état de santé général.

**Pourquoi Spark ?** Spark découpe les données en partitions traitées en parallèle, sans tout charger en mémoire comme pandas. Le même code fonctionnerait sur un cluster de plusieurs machines.

**Démarche :**
- **Data** : lecture des fichiers **2013** et **2015**, chacun séparément (l'ordre des colonnes change d'une année à l'autre), puis fusion par nom de colonne (`unionByName`). Les questions sur la tension et le cholestérol ne sont posées que les années impaires : l'année 2014 ne peut donc pas être utilisée.
- **EDA** : taux de diabète global et par tranche d'âge (calculs avec `groupBy` dans Spark).
- **Prétraitement** : suppression des réponses « ne sait pas / refus » (codes 7, 9, 14) et des valeurs manquantes.
- **Transformation** : recodage des réponses en 0/1, IMC en vraie valeur, `VectorAssembler`.
- **ML** : régression logistique de Spark MLlib, avec une colonne de poids pour compenser le déséquilibre des classes.
- **Évaluation** : découpage temporel (**entraînement sur 2013 : 359 552 personnes**, **test sur 2015 : 313 466 personnes**), AUC, rappel et matrice de confusion.
- **Déploiement** : sauvegarde et rechargement du pipeline Spark (`PipelineModel`), puis prédiction pour une nouvelle personne.

---

## Comment exécuter les notebooks

**Étape 1 (pandas) :** sur Kaggle ou sur un ordinateur avec Jupyter.
- Télécharger le jeu de données depuis Kaggle et placer `flights.csv` et `airlines.csv` dans un dossier.
- Indiquer ce dossier dans la variable `DOSSIER` du notebook.

**Étape 2 (Spark) :** de préférence **sur Kaggle**, car l'installation de Spark sous Windows demande Java et une configuration supplémentaire.
- Ouvrir la page du jeu BRFSS, puis cliquer sur **New Notebook** (les données sont attachées automatiquement).
- Activer **Internet** dans les paramètres du notebook (pour `pip install pyspark`).
- Importer le notebook (**File → Import Notebook**), puis cliquer sur **Run All**.

Les fichiers de données ne sont pas inclus dans ce dépôt : ils sont trop volumineux pour GitHub et disponibles sur Kaggle.

---

## Limites

- **Étape 1** : le modèle est entraîné sur un échantillon de 300 000 vols sur 5,8 millions ; les aéroports ne sont pas utilisés.
- **Étape 2** : Spark tourne en mode local, sur une seule machine. Seules deux années d'enquête sont comparables pour les variables choisies.
- Les modèles sont des outils d'analyse : ils ne remplacent ni une décision opérationnelle, ni un avis médical.
