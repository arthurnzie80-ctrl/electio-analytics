# MSPR Electio-Analytics

Pipeline de données et modèles de Machine Learning pour prédire la famille politique arrivée en tête dans les communes du Rhône, à partir des résultats électoraux et d'indicateurs socio-économiques.

**Établissement :** EPSI — Projet MSPR (Mise en Situation Professionnelle Reconstituée)
**Périmètre géographique :** Département du Rhône (69) — 280 communes, 1,9 million d'habitants
**Scrutins collectés :** 8 premiers tours, de 2014 à 2024 (européennes 2014, 2019, 2024 ; présidentielles 2017, 2022 ; législatives 2017, 2022, 2024)

**Site de présentation :** ouvrir `index.html` · **Application interactive :** `electio-analytics.html`

---

## Objectif du projet

Prédire, pour chaque commune, la **famille politique en tête au premier tour** (Centre, Droite, Extrême droite ou Gauche), puis proposer une **projection indicative pour la présidentielle 2027**.

C'est un problème de **classification** : le modèle désigne un vainqueur par commune, il ne prédit pas les scores.

## Résultats

Résultats du notebook `scripts/05_machineLearning.ipynb`, sur les 266 communes des législatives 2024 (212 pour l'entraînement, 54 pour le test, découpage aléatoire stratifié 80/20).

Jeu de variables « master + deltas » (154 variables) :

| Modèle | Accuracy | F1 pondéré | Validation croisée |
|---|---|---|---|
| **Régression logistique** | **0,870** | **0,841** | 0,829 |
| Gradient Boosting | 0,852 | 0,817 | 0,859 |
| Random Forest | 0,833 | 0,801 | 0,850 |
| KNN | 0,833 | 0,796 | 0,833 |

Référence à battre : 70 % d'accuracy en répondant toujours « Extrême droite » (187 communes sur 266 en 2024).

Meilleure accuracy selon le jeu de variables (régression logistique) :

| Jeu de variables | Nombre de variables | Accuracy |
|---|---|---|
| Master complet | 142 | 0,852 |
| Deltas seuls | 12 | 0,778 |
| Master + deltas | 154 | 0,870 |

**Projection présidentielle 2027** (modèle entraîné sur la présidentielle 2022, accuracy 0,889 en test) : Centre en tête dans 189 communes, Extrême droite dans 64, Gauche dans 14.

### Limites

- **Classes déséquilibrées :** aucune des 3 communes « Droite » du jeu de test n'est reconnue, quel que soit le modèle.
- **Jeu de test réduit :** 54 communes, soit près de 2 points d'accuracy par erreur.
- **Test dans la même élection :** entraînement et test sont tirés au hasard dans le scrutin de 2024. Ce score ne mesure pas la capacité à prédire une élection future ; `03_processed/bilan_modeles.csv` conserve un autre essai, moins favorable (0,587 pour la régression logistique).
- **Notebook `07_projection_1à3ans.ipynb` :** le rééquilibrage SMOTE y est appliqué avant la séparation entraînement/test, ce qui surestime les scores (0,97), et la projection obtenue classe toutes les communes en « Extrême droite ». Ces résultats ne sont pas retenus.
- **La projection 2027 est indicative :** elle prolonge les données 2017–2022 sans tenir compte des candidats ni de la campagne.

---

## Équipe

Projet réalisé en groupe de 4, avec les rôles suivants :

| Rôle | Mission |
|---|---|
| Product Manager | Cadrage fonctionnel, priorisation |
| Data Analyst | Exploration et analyse des données |
| **Data Scientist** *(Paul Yvan)* | Feature engineering, modélisation ML |
| Data Engineer | Pipeline de données, infrastructure |

---

## Architecture du pipeline de données

```
01_raw/          → Données brutes issues des sources officielles (non versionnées)
02_staging/      → Données filtrées sur le Rhône, nettoyées et agrégées par commune
03_processed/    → Jeu final master_ml.csv (1 360 lignes × 40 colonnes)
04_outputs/      → Modèles entraînés et graphiques (eda, etl, electoral, ml)
scripts/         → Notebooks 00 à 07, dashboard Streamlit, diagrammes PlantUML
pipeline.py      → Pipeline ETL complet
```

Le pipeline suit deux branches, électorale et socio-économique, qui se rejoignent dans le master : filtrage sur le département 69, mapping des nuances vers les familles politiques, agrégation des bureaux de vote vers les communes, imputation des valeurs manquantes par la médiane, puis jointure. Le diagramme complet est dans `out/scripts/flux_ETL/flux_ETL.png`.

---

## Sources de données

| Source | Données |
|---|---|
| Ministère de l'Intérieur | Résultats électoraux par bureau de vote |
| INSEE — Recensement | Population, logements, emploi et chômage, immigration |
| INSEE — Filosofi | Revenu médian, pauvreté, déciles de revenu |
| SSMSI | Délinquance enregistrée par commune |

**Formats rencontrés :** CSV, Excel

---

## Approche « delta » (feature engineering)

En complément des indicateurs à une date donnée, le projet calcule des **indicateurs delta** : l'évolution d'un indicateur entre les millésimes 2017 et 2022, pour capter la dynamique des territoires.

**Principe :** toujours comparer deux millésimes issus de la **même source**.

Douze variables delta sont produites dans `05_machineLearning.ipynb`, à partir de six indicateurs :

| Indicateur | Source |
|---|---|
| Revenu médian | INSEE Filosofi |
| Population | INSEE Recensement |
| Densité | INSEE Recensement |
| Nombre de logements | INSEE Recensement |
| Nombre d'immigrés | INSEE Recensement |
| Total des délits | SSMSI |

Seuls, les deltas donnent de moins bons résultats que les variables de base ; ajoutés à celles-ci, ils améliorent trois des quatre modèles.

---

## Points de vigilance méthodologiques

- **Secret statistique INSEE :** certaines petites communes n'ont pas de valeur publiée ; 266 communes sur 280 sont conservées pour la modélisation.
- **Fusions de communes :** le nombre de communes passe de 280 (2017) à 266–267 (2022, 2024).
- **Toujours utiliser `.copy()`** lors de la création de DataFrames pandas, pour éviter les écrasements accidentels.

---

## Stack technique

- **Langage :** Python
- **Traitement de données :** pandas, SQLite
- **Machine Learning :** scikit-learn (régression logistique, Random Forest, Gradient Boosting, KNN), imbalanced-learn
- **Visualisation :** matplotlib, seaborn, Streamlit (`scripts/06_dashboard.py`)
- **Environnement :** Jupyter Notebooks
- **Web :** site de présentation en HTML/CSS/JS, application interactive en React
