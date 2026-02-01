## Navigation
Pour naviguer entre les différentes étapes du processus veuillez sélectionner les sous-branches nommées dans l'ordre d'exécution.
*insérer capture d'écran ici*

## 🏢 Contexte
Ce projet a pour but de fournir une vision fiable et robuste de la trajectoire commerciale d'un réseau de 45 magasins. Il permet également de transformer un historique de ventes brutes en un outil de pilotage de la performance, et d'aide à la prise de décision stratégique.

## 🎯 Objectifs
- Anticiper la trajectoire des ventes : Projeter les revenus du réseau sur un horizon de 8 semaines afin de s'adapter aux variations de l'activité.
- Superviser la performance : Fournir un outil clair et orienté business, permettant à la fois la projection et la rétrospection des résultats.
- Fiabiliser les chiffres et auditer le traitement de la donnée : Proposer un code auditable et une méthodologie documentée, permettant de justifier les chiffres affichés avec une précision de 94%.
- Optimiser le reporting : Automatiser la consolidation des données et la création d'un Excel directement exploitable sous Power BI. Garantie d'une mise à jour peu chronophage.

## 🚀 Résultats
- Fiabilité des projections : 94,18% de précision (moyenne pondérée sur l'ensemble du réseau).

Validation robuste : Performance testée et confirmée sur un historique de 26 semaines (~6 mois).

Aide à la décision : Réduction de l'incertitude globale sous le seuil des 6% grâce à un arbitrage de modèles conservateur.

Gain de productivité : Automatisation complète du pipeline (du calcul Python à la visualisation Power BI), garantissant une mise à jour rapide et sans saisie manuelle.

## 🔁 Workflow
1. Récupération du dataset Wallmart (donnée publiée sur le site Kagle) et préparation du fichier source.
2. Déploiement d'un moteur d'analyse sous Python pour chaque entité du réseau. 
3. Génération automatisée d'un fichier structuré, éliminant les processus manuels et les risques d'erreurs de saisie.
4. Import des données sous Power BI et visualisation dynamique des résultats.

## 🏗️ Outils utilisés
- Power BI : DAX
- Excel
- Python : librairies Pandas, NumPy, Statsmodels, XGBoost (algorithme de machine learning)

## 📁 Contenu du projet
- Etape 1 : Méthodologie de construction des modèles statistiques et présentation de leurs performances.
- Etape 2 : Présentation de l'outil de pilotage de la performance.
- Etape 3 : Mise à jour du forecast en contexte opérationnel.
