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
- On copie simplement ce qui s’est passé il y a exactement 52 semaines (capture automatique de la saisonnalité).

2. **XGBoost CV one-step**  
- Ce modèle cumule moyenne mobile et lags (ventes d'il y a X temps) pour essayer de capter la tendance récente et de prédire sur le très court terme.
- Dans notre cas de ventes retail la saisonnalité est tellement forte que seul le lag-52 (semaine identique de l'année dernière) est réellement pris en compte, les résultats de ce modèle sont donc similaires à ceux du "Naïf saisonnier".

3. **XGBoost Itératif**  
- Ce modèle utilise ses propres prévisions pour prédire les données semaine après semaine, il y a donc en théorie une accumulation d'erreur sur le long terme.
- Ce problème est minoré par notre approche court terme visant à prédire uniquement 8 points de données par magasin.

4. **XGBoost Rolling Refit**  
- Ce modèle calcule les prévisions de chaque semaine avec les ventes réelles les plus récentes.
- Pour prévoir la semaine 2, on utilise les ventes réelles de la semaine 1 (celles de l'année dernière et non celles qui viennent tout juste d'etre prédites).  
- Le modèle est donc plus précis mais plus long à exécuter (car pour chaque semaine à prévoir on entraine le modèle sur tout le jeu de données).

## 📊 Résultats globaux

Erreur moyenne pondérée par le chiffre d’affaires de chaque magasin :

| Méthode de prévision                  | Weighted MAPE |
|---------------------------------------|-----------------------|
| Naïf saisonnier                       | 5,92 %               |
| XGBoost CV one-step                   | 5,90 %               |
| XGBoost Itératif                      | 4,72 %               |
| XGBoost Rolling Refit                 | 4,32 %               |
| **Score consolidé de la sélection par meilleur modèle par magasin** | **4,22 %**   |

Lecture : Si on avait uniquement sélectionné le "Naïf saisonnier" pour chaque magasin notre Weighted MAPE serait de 5,92%. En testant chaque magasin sur les 4 modèles pour ne retenir que le plus performant, nous gagnons drastiquement en prévision avec un score final de 4,22%.

→ **Gain de précision de ~30 %** par rapport à la méthode naïve.

## 🔍 Éléments pris en compte dans le modèle
- Impact des **jours fériés US** et du **Black Friday** (flag spécial par semaine)
- Ventes des semaines précédentes (1, 4 et 52 semaines avant)
- Moyenne mobile sur 4 semaines
- Numéro de la semaine dans l’année (saisonnalité)

## ✅ Points forts de cette méthodologie
- Comparaison objective et automatique de 4 approches
- Choix adapté à chaque magasin (pas une méthode unique pour tous)
- Audit complet (fichier Excel avec tous les détails)
- Résultats défendables en réunion : on sait exactement pourquoi une méthode a été choisie

## 📂 Contenu de cette branche
- `walmart_forecast_final.py` : script principal avec tout le calcul
- Dossier `PowerBI_Ready` (généré au lancement) : fichier Excel avec les 4 onglets (historique, consolidé, audit, synthèse)

## ➡️ Prochaine étape
Une fois cette partie validée, passez à la branche **Étape 2** pour voir comment exploiter ces prévisions dans Power BI (visualisations, tableaux de bord, etc.).

