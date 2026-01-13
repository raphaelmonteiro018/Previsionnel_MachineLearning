## 📈 Méthodologie et résultats des modèles prédictifs
Cette branche détaille la **partie cœur du projet** : comment les prévisions sont calculées, quelles méthodes sont comparées, et quels en sont les résultats.

## 🎯 Objectifs de cette partie
- Réaliser un benchmark de modèles prévisionnels à partir des données historiques de Wallmart.
- Mesurer leur précision à l'aide de l'indicateur WAPE (Weighted Absolute Percentage Error).
- Choisir automatiquement la meilleure méthode **pour chaque magasin**.
- Produire un audit clair des modèles et consolider les prévisions en pondérant chaque score WAPE par le chiffre d'affaires.


## Récupération, traitemement et description du dataset

insérer les étapes pour avoir un dataset propre et exploitable.

## 🛠️ Méthodes comparées
Le script présent en pièce-jointe teste **3 approches** pour prévoir les ventes sur 8 semaines (y+8) :

1. **Naïf saisonnier**  
- Le modèle le plus basique qui consiste à dire : « Cette année, la semaine 1 aura les mêmes ventes que la semaine 1 de l’année dernière ».  
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

