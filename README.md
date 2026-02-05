# 📈 Méthodologie & Résultats des modèles
Cette section détaille le cœur analytique du projet, c'est-à-dire comment l'architecture construite à partir de Python transforme un historique brut en une projection fiable.

## 🎯 Objectifs de l'étape
- Construire un code capable de comparer dynamiquement plusieurs modèles statistiques pour chaque point de vente, en sélectionnant le plus précis (benchmark).
- Mesurer la précision de ces modèles via l'indicateur WAPE (Weighted Absolute Percentage Error).
- Adopter une approche business en pondérant l'erreur individuelle (WAPE) par le poids du chiffre d'affaires.
- Justifier les choix de modélisation et donc les chiffres finaux, à partir d'une méthode documentée et reproductible.

## 🔍 Récupération du dataset & Analyse visuelle de la série temporelle
- Récupération du dataset Walmart disponible librement sur Kaggle.
- Données des colonnes : Store (numéro du magasin), ds (date), y (ventes hebdomadaires du magasin), Holiday_Flag (binaire).

#### Analyse simple de la série temporelle : Consolidation des données historiques et visualisation de la distribution des ventes ci-dessous.
<img width="1238" height="378" alt="image" src="https://github.com/user-attachments/assets/111a0656-9045-4e80-9afa-49805a164c24" />

#### Analyse : D'après la période étudiée (données de début 2010 à fin 2011), l'activité des 45 magasins Wallmart est extremement saisonnière, notre distribution prend donc visuellement une forme bimodale. -> Justifie l'approche via Python plutot qu'Excel.

#### Conséquence : L'intensité de l'activité est représentée par deux régimes distincts (baseline / pics) pour lesquels les intervalles de confiance doivent être adaptés dynamiquement pour refléter la différence de tailles des erreurs (hétéroscédasticité) des deux régimes.
- La vérification de la taille des erreurs (hétéroscédasticité), représentée ici par le WAPE (Weighted Absolute Percentage Error) permet de prouver qu'elles dépendent du régime de l'activité (baseline / pics).
- Exemple : Sur la baseline (scénario stable), les déviations à la moyenne (écart-type) vont etre beaucoup plus faibles que sur les scénarios de pics d'activité. Plus simplement, la série à tendance à rompre sa moyenne momentanément, ces moments doivent faire l'objet d'une attention particulière.

## 📊 Statistiques Descriptives

### 1. Comparaison des régimes d'activité
> **💡 Diagnostic :**
> - La segmentation de l'activité a été réalisée par le choix du 90ème Percentile des ventes hebdomadaires consolidées.
> - Le point de bascule du régime "baseline" au régime "pics" a été statisquement quantifié à 49.88 M$ ci-dessous. Dans 90% du temps, le montant total des ventes est situé sous ce seuil.
> - Ce choix permet d'isoler mathématiquement la "Queue de distribution" (Tail Risk), c'est-à-dire les 10% d'événements "extremes".

| Métrique | REGIME 1 (Baseline) | REGIME 2 (Pics) |
| :--- | :--- | :--- |
| **Nb. Semaines** | 128 (90%) | 15 (10%) |
| **CA Moyen (μ)** | **45,767,633 $** | **58,597,458 $** |
| **Écart-type (σ)** | 2,132,545 $ | **10,075,269 $** |
| **Volatilité (CV)** | **4.66 %** | **17.19 %** |
| **Amplitude CA** | [39.6M$ - 49.7M$] | [49.9M$ - 80.9M$] |

> **💡 Diagnostic :**
> - L'écart-type est multiplié par **4.7** lors du passage de l'activité "normale" aux pics. Cette explosion de la volatilité des ventes prouve l'**hétéroscédasticité** de la série (l'erreur de prévision n'est pas constante).
> - Cette approche est supérieure à une moyenne simple qui aurait simplement surestimé la volatilité des ventes futures dans un scénario de baseline et sous-estimé celle lors des pics d'activité.

---

### 2. Analyse de l'incertitude

| Indicateur | Valeur | Impact Stratégique |
| :--- | :--- | :--- |
| **Ratio d'incertitude** | **4.72x** | Le risque de rupture est 4.7 fois plus élevé lors des pics d'activité saisonniers |
| **Incertitude Baseline** | **3.88 %** | Précision de 96.12% dans 90% de l'année (optimisation du BFR). |
| **Incertitude Pics** | **18.31 %** | Marge de sécurité nécessaire pour couvrir la volatilité des pics. |
| **Valeur du point de WAPE** | **~471 k$** | Pour chaque point d'incertitude réduit, le BFR peut-etre optisé en réduisant les dépenses liées aux stocks|

> **💡 Diagnostic :** En isolant le régime "pics" le chiffre d'affaires est sécurisé. On accepte une incertitude de 18.31% sur les 10% des semaines avec les plus fortes ventes pour garantir un taux de service maximal, tout en maintenant une gestion tendue le reste de l'année (incertitude de 3.88% pour la baseline).


## 🛠️ Conception & Explication du Benchmark
- Le script *insérer nom du script* présent en pièce-jointe fait concourir **3 approches** pour chaque magasin et sélectionne dynamiquement la plus performante sur une période de validation étendue.
- Le modèle est évalué et entrainé sur un horizon de 26 points de données hebdomadaires consolidées **soit 26 semaines (~6 mois)**.

---

### 1. Benchmark Naïf (Baseline Saisonnier)
* **Logique** : Projection directe des ventes de l'année précédente ($y = y_{t-52}$). On reprend tout simplement la valeur de l'année précédente.
* **Rôle** : Sert de garde-fou. Si un modèle complexe ne bat pas le Naïf, il est rejeté.
*  **Inconvénient** : Ce modèle capture parfaitement la saisonnalité pure mais ignore les tendances de croissance et les relations complexes.

### 2. Holt-Winters (Triple Lissage Exponentiel)
* **Logique** : Modèle statistique classique décomposant la série en Niveau + Tendance + Saisonnalité ($Additive$).
* **Force** : Très efficace sur les magasins ayant des cycles saisonniers très réguliers et peu de bruit aléatoire.

### 3. XGBoost (Machine Learning)
* **Logique** : Algorithme de Machine Learning utilisant des régresseurs avancés (Lags tels que y-52, Moyennes Mobiles, Flag des jours fériés et pics saisonniers).
* **Méthode Itérative** : Le modèle prédit $y+1$, réintègre cette valeur dans son historique, recalcule les moyennes mobiles, puis prédit $y+2$.
* **Force** : Capacité unique à capturer les ruptures de régime (ex: Thanksgiving et Noel) et les relations complexes dans les données.

---

## 🛡️ Sécurisation des Prévisions
Plutôt que d'appliquer des bornes fixes, le modèle adapte ses intervalles de confiance en fonction du régime d'activité identifié lors de l'audit statistique.

| Régime Détecté | Logique d'Incertitude | WAPE Cible | Stratégie de Stock |
| :--- | :--- | :--- | :--- |
| **Baseline** | Activité standard (< 49.88 M$) | **3.88 %** | **Flux tendus** : Réduction maximale de l'immobilisation financière. |
| **Extreme Peaks** | Pics saisonniers (> 49.88 M$) | **18.31 %** | **Marge de sécurité** : Élargissement du tunnel pour couvrir la volatilité (Risk-Off). |

> **💡 Note :** Le passage du WAPE de 3.88% à 18.31% n'est pas une perte de performance, mais une **calibration sur le risque réel**. En multipliant les bornes de confiance par **4.72** lors des pics, le modèle garantit un taux de service optimal là où un modèle standard provoquerait des ruptures massives.

---





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

