## 📈 Méthodologie et résultats des modèles prédictifs
Cette section détaille le cœur analytique du projet, c'est-à-dire comment l'architecture construite à partir de Python transforme un historique brut en une projection fiable.

## 🎯 Objectifs de cette partie
- Construire un code capable de comparer dynamiquement plusieurs modèles statistiques pour chaque point de vente, en sélectionnant le plus précis (benchmark).
- Mesurer la précision de ces modèles via l'indicateur WAPE (Weighted Absolute Percentage Error).
- Adopter une approche business en pondérant l'erreur individuelle (WAPE) par le poids du chiffre d'affaires.
- Justifier les choix de modélisation et donc les chiffres finaux, à partir d'une méthode documentée et reproductible.

## 🔍 Récupération & Analyse du Dataset
- Récupération du dataset Wallmart disponible librement sur Kagle.
- Données des colonnes : Store (numéro du magasin), ds (date), y (ventes hebdomadaires du magasin), Holiday_Flag (binaire).
- Avec Python j'ai généré une analyse descriptive sur les données consolidées des 45 points de vente pour définir la "baseline" statistique du réseau et mieux comprendre la stucture du dataset. J'ai d'abord récupéré la moyenne des ventes hebdomadaires pour le réseau, puis j'ai choisi d'étudier la variance de la série temporelle à travers l'écart-type, pour enfin calculer le coefficient de variation du dataset.

📊 Statistiques Descriptives
==================================================

## Analyse de la série temporelle (consolidée)

| Statistique                       | Valeur                  |
|----------------------------------|------------------------|
| Moyenne Hebdomadaire Réseau       | 47,113,419.49 $        |
| Écart-type (en valeur absolue)         | 5,425,137.12 $         |
| Coefficient de Variation (CV)     | 11.52 %                |

## Indicateurs et impact sur le modélisation à réaliser

| Indicateur         | Valeur       | Impact sur le Modèle                                                                 |
|-------------------|-------------|-------------------------------------------------------------------------------------|
| Ventes Moyennes du Réseau       | **~47.1 M$** | Enjeu financier massif : 1% d'erreur représente ~470k$ d'incertitude.              |
| Volatilité (CV)    | 11.52 %      | Indique une nervosité du réseau. Une moyenne mobile simple serait inefficace car trop tardive à réaliser les déviations à la moyenne (forte saisonnalité) |
| Structure de la série        | Bimodale     | Deux pics extrêmes (Black Friday / Noël) imposent l'usage de Flags et de Lags pour anticiper les changements de rythmes brutaux de l'activité     |





## 🛠️ Méthodes comparées
Le script présent en pièce-jointe teste **3 approches** pour prévoir les ventes sur 8 semaines (y+8) :

1. **Naïf saisonnier**  
- Modèle de base du benchmark qui consiste à dire : « Cette année, la semaine 1 aura les mêmes ventes que la semaine 1 de l’année dernière ».  
- Le modèle copie simplement ce qu'il s’est passé il y a exactement 52 semaines (y-52).
- Avantage : capture automatique de la saisonnalité.
- Inconvénients : modèle peu sophistiqué, ne contient aucune variable et ne capte donc pas les phénomènes de tendances (moyennes mobiles).

2. **XGBoost Itératif**  
- Ce modèle intègre et capture les relations complexes entre les regresseurs (moyennes mobiles, points de données flagués comme importants, etc).
- Lorsque le modèle prédit la prochaine valeur il l'intègre dans son historique de données et l'utilise pour la prévision suivante.
- Cette approche peut entraîner une propagation des erreurs car une erreur à y+1, même minime, est répercutée à y+2, et ce jusqu'à la dernière prévision (ici y+8).
- Ce risque est volontairement maîtrisée de par mon approche court terme visant à prédire 8 points de données par magasin (8 semaines) et une bonne qualité du modèle (bon scoring au WAPE et donc faibles erreurs potentielles).

3. **XGBoost Rolling Refit (Re-Fit Forecasting)**
- Ce modèle effectue les prévisions semaine par semaine en réentraînant le modèle à chaque nouvelle prévision, le tout sans intégrer ses propres résultats dans l'historique (contrairement au précédent modèle). Il capture également les relations complexes entre les regresseurs.
- Le modèle s'entraîne sur une fenêtre glissante de 52 semaines.
- Avantage : réduction de la propagation des erreurs et obtention de prévisions plus précises (meilleur score au WAPE).
- Inconvénient : actualisation plus longue dans le cadre d'un reporting. Temps de calcul plus long car le modèle est réentraîné à chaque pas.

## 📊 Résultats globaux

| Méthode de prévision                  | Weighted MAPE |
|---------------------------------------|-----------------------|
| Naïf saisonnier                       | 5,92 %               |
| XGBoost CV one-step                   | 5,90 %               |
| XGBoost Itératif                      | 4,72 %               |
| XGBoost Rolling Refit                 | 4,32 %               |
| **Score consolidé de la sélection par meilleur modèle par magasin** | **4,22 %**   |

Lecture : Pour chaque magasin, si on avait uniquement sélectionné le "Naïf saisonnier" notre Weighted MAPE serait de 5,92%. En testant chaque magasin sur les 4 modèles pour ne retenir que le plus performant, nous gagnons drastiquement en précision avec un score final de 4,22%.

→ **Gain de précision de ~30 %** par rapport à la méthode naïve.

## 🔍 Éléments pris en compte dans les modèles (hors naif saisonnier)
- Impact des **jours fériés US** et du **Black Friday** (score "Holiday" de 1 ou de 0 qui permet d'identifier les semaines impactées par ces événements particuliers)
- Ventes des semaines précédentes (lag de 1, 4 et 52 semaines)
- Moyenne mobile sur 4 semaines
- Numéro de la semaine dans l’année (saisonnalité)

## ✅ Points forts de ma méthodologie
- Comparaison objective et automatique de 3 approches.
- Choix du modèle adapté en fonction des données historiques de chaque magasin.
- Auditabilité complète grace au fichier Excel généré : détails par semaine, par magasin,résultats consolidés et bandes d'incertitudes.
- Résultats défendables : On sait exactement pourquoi une méthode a été choisie plutot qu'une autre, de plus, les scores WAPE sont pondérés par le CA pour privilégier une approche business-oriented.

## 📂 Contenu de cette branche
- `walmart_forecast_final.py` : script principal avec tout le calcul
- Dossier `PowerBI_Ready` (généré au lancement) : fichier Excel avec les 4 onglets (historique par magasin, historique consolidé, audit, synthèse)

## ➡️ Prochaine étape
Une fois cette partie validée, passez à la branche **Étape 2** pour voir comment exploiter ces prévisions dans Power BI (visualisations, tableaux de bord, etc.).

