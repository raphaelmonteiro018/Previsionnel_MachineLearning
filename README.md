## 📈 Méthodologie et résultats des modèles prédictifs
Cette branche détaille la **partie cœur du projet** : comment les prévisions sont calculées, quelles méthodes sont comparées, et quels en sont les résultats.

## 🎯 Objectifs de cette partie
- Réaliser un benchmark de modèles prévisionnels à partir des données historiques de Wallmart
- Mesurer leur précision à l'aide de l'indicateur MAPE (Mean Absolute Percentage Error : écart en pourcentage entre les valeurs réelles et les prédictions produites par le modèle)
- Choisir automatiquement la meilleure méthode **pour chaque magasin**
- Produire un audit clair et pondéré par le chiffre d'affaires

## 🛠️ Méthodes comparées
Le script présent en pièce-jointe teste **4 approches** pour prévoir les ventes sur 8 semaines :

1. **Naïf saisonnier**  
- Le modèle le plus basique qui consiste à dire : « Cette année, la semaine 1 aura les mêmes ventes que la semaine 1 de l’année dernière ».  
- On copie simplement ce qu'il s’est passé il y a exactement 52 semaines (capture automatique de la saisonnalité).

2. **XGBoost CV one-step**  
- Ce modèle cumule moyenne mobile et lags (ventes d'il y a X temps) pour essayer de capter la tendance récente et de prédire sur le très court terme.
- Dans notre cas de ventes retail la saisonnalité est tellement forte que seul le lag-52 (semaine identique de l'année dernière) est réellement pris en compte, les résultats de ce modèle sont donc similaires à ceux du "Naïf saisonnier".

3. **XGBoost Itératif**  
- Ce modèle utilise ses propres prévisions en plus des données réelles pour effectuer les prédictions.
- Cette approche peut entraîner une propagation des erreurs à mesure que l’horizon de prévision s’allonge, dans la mesure où les prévisions passées servent de base aux suivantes.
- Ce risque est volontairement limité par notre approche court terme visant à prédire 8 points de données par magasin (8 semaines).

4. **XGBoost Rolling Refit (Re-Fit Forecasting)**
- Ce modèle effectue les prévisions semaine par semaine en réentraînant le modèle à chaque nouvelle observation disponible.
- Pour prédire la semaine S+1, le modèle utilise les ventes réelles jusqu’à la semaine S et non ses propres prévisions.
- Cette approche permet de réduire la propagation des erreurs et d’obtenir des prévisions plus précises, au prix d’un temps de calcul plus long, puisque le modèle est réentraîné à chaque pas.

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
- Comparaison objective et automatique de 4 approches
- Choix du modèle adapté en fonction des données historiques de chaque magasin
- Auditabilité complète grace au fichier Excel généré : Il contient les détails par semaine, par magasin, les résultats consolidés et propose également des bandes d'incertitudes.
- Résultats défendables : On sait exactement pourquoi une méthode a été choisie plutot qu'une autre, de plus, les scores sont pondérés par le CA ce qui favorise l'approche business-oriented.

## 📂 Contenu de cette branche
- `walmart_forecast_final.py` : script principal avec tout le calcul
- Dossier `PowerBI_Ready` (généré au lancement) : fichier Excel avec les 4 onglets (historique, consolidé, audit, synthèse)

## ➡️ Prochaine étape
Une fois cette partie validée, passez à la branche **Étape 2** pour voir comment exploiter ces prévisions dans Power BI (visualisations, tableaux de bord, etc.).

