# 📈 Méthodologie et résultats des modèles prédictifs
Cette section détaille le cœur analytique du projet, c'est-à-dire comment l'architecture construite à partir de Python transforme un historique brut en une projection fiable.

## 🎯 Objectifs de cette partie
- Construire un code capable de comparer dynamiquement plusieurs modèles statistiques pour chaque point de vente, en sélectionnant le plus précis (benchmark).
- Mesurer la précision de ces modèles via l'indicateur WAPE (Weighted Absolute Percentage Error).
- Adopter une approche business en pondérant l'erreur individuelle (WAPE) par le poids du chiffre d'affaires.
- Justifier les choix de modélisation et donc les chiffres finaux, à partir d'une méthode documentée et reproductible.

## 🔍 Récupération du dataset & Analyse visuelle de la série temporelle
- Récupération du dataset Walmart disponible librement sur Kaggle.
- Données des colonnes : Store (numéro du magasin), ds (date), y (ventes hebdomadaires du magasin), Holiday_Flag (binaire).
- Analyse simple de la série temporelle : Consolidation des données historiques et visualisation de la distribution des ventes ci-dessous.

<img width="1238" height="378" alt="image" src="https://github.com/user-attachments/assets/111a0656-9045-4e80-9afa-49805a164c24" />

#### Analyse : D'après la période étudiée (données de début 2010 à fin 2011), l'activité des 45 magasins Wallmart est extremement saisonnière, notre distribution prend donc visuellement une forme bimodale. -> Justifie l'approche via Python plutot qu'Excel.

#### Conséquence : L'intensité de l'activité est représentée par deux régimes distincts (baseline / pics) pour lesquels les intervalles de confiance doivent être adaptés dynamiquement pour refléter la différence de tailles des erreurs (hétéroscédasticité) des deux régimes.
- La vérification de la taille des erreurs (hétéroscédasticité), représentée ici par le WAPE (Weighted Absolute Percentage Error) permet de prouver qu'elles dépendent du régime de l'activité (baseline / pics).
- Exemple : Sur la baseline (scénario stable), les déviations à la moyenne (écart-type) vont etre beaucoup plus faibles que sur les scénarios de pics d'activité. Plus simplement, la série à tendance à rompre sa moyenne momentanément, ces moments doivent faire l'objet d'une attention particulière.

## 📊 Statistiques Descriptives

### 1. Analyse globale de la série temporelle
| Statistique | Valeur |
| :--- | :--- |
| **Moyenne Hebdomadaire Réseau** | **47,113,419.49 $** |
| **Écart-type (σ)** | 5,425,137.12 $ |
| **Coefficient de Variation (CV)** | **11.52 %** |

| Indicateur | Valeur | Impact Stratégique & Modélisation |
| :--- | :--- | :--- |
| **Ventes Moyennes** | **~47.1 M$** | Enjeu financier massif : 1% d'erreur représente **~471k$** d'incertitude sur le P&L. |
| **Volatilité (CV)** | 11.52 % | Indique une nervosité du réseau. Une simple moyenne mobile serait inefficace car incapable de capturer les déviations brutales. |
| **Structure** | Bimodale | Les deux pics extrêmes imposent l'usage de **Flags** et de **Lags** pour anticiper les ruptures de rythme. |

> **💡 Insight :** L'écart-type massif (5.4 M$) par rapport à la moyenne indique que le réseau ne tourne jamais en "vitesse de croisière". La volatilité de 11.52% confirme que le pilotage manuel sur Excel est statistiquement condamné à l'erreur (sur-stockage ou rupture).

---

### 2. Audit de Segmentation P90 (Gestion du Tail Risk)
Pour affiner la précision, j'ai segmenté le réseau via le **90ème percentile (P90)**. Cette approche isole mathématiquement la "queue de distribution" (les 10% d'événements extrêmes) du reste de l'activité.

| Métrique | Régime 1 : **Baseline** | Régime 2 : **Extreme Peaks** |
| :--- | :--- | :--- |
| **Seuil de CA (P90)** | < 49.88 M$ | **> 49.88 M$** |
| **Volatilité (CV)** | **4.66 %** | **17.19 %** |
| **Hétéroscédasticité** | Régime Stable | **Incertitude x 4.72** |

> **💡 Insight :** On observe que l'incertitude ne progresse pas de manière linéaire : elle explose. En isolant les 15 semaines de "Peak", on découvre que le risque est **4.72 fois plus élevé** que le reste de l'année. Cette segmentation permet d'adapter les politiques de stock spécifiquement pour les périodes de haute tension.

---

### 3. Fiabilité Conditionnelle des Prévisions (Impact BFR)
Le modèle n'applique pas une erreur uniforme. La précision (WAPE) est ajustée dynamiquement selon le régime de vente détecté pour optimiser le Besoin en Fonds de Roulement (BFR).

| Régime détecté | Précision (WAPE) | Impact sur le Pilotage (BFR) |
| :--- | :--- | :--- |
| **Baseline** | **3.88 %** | Libération de cash : flux tendus sécurisés 90% de l'année. |
| **Extreme Peaks** | **18.31 %** | Protection CA : extension des stocks de sécurité lors des pics. |

> **💡 Insight :** Le score de 18.31% en période de pic n'est pas une faiblesse du modèle, mais une **modélisation réaliste de la volatilité intrinsèque** (17.19%). Cette approche permet de basculer du simple *Forecasting* au *Prescriptive Analytics* : le modèle prévient que les bornes de confiance doivent s'élargir pour absorber le choc de demande.






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

