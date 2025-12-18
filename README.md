# Prévision de Ventes Hebdomadaires Walmart  
**FP&A Forecasting avec XGBoost – Time Series Multi-Step** 🚀

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue)](https://www.python.org/)
[![Pandas](https://img.shields.io/badge/Pandas-2.0%2B-green)](https://pandas.pydata.org/)
[![XGBoost](https://img.shields.io/badge/XGBoost-2.0%2B-orange)](https://xgboost.readthedocs.io/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Projet personnel inspiré d’un case study classique FP&A retail : prévision rolling sur **8 semaines** des ventes hebdomadaires de 45 magasins Walmart.

L’objectif est de produire une prévision **précise, défendable et directement exploitable** en Power BI ou en réunion budgétaire.

## 🎯 Contexte métier
- Données : ventes hebdomadaires par magasin (dataset public Kaggle "Walmart Recruiting - Store Sales Forecasting")
- Fréquence : hebdomadaire (alignée vendredi)
- Particularités retail : forte saisonnalité annuelle, impacts majeurs des holidays US et Black Friday
- Besoin FP&A : prévision consolidée + par magasin, avec audit clair des performances

## 📊 Résultats clés
MAPE pondéré par chiffre d'affaires (sur l'ensemble des magasins) :

| Modèle                          | MAPE Pondéré |
|---------------------------------|--------------|
| Naïf saisonnier (lag 52)        | 5.92%       |
| XGBoost CV one-step             | 5.90%       |
| XGBoost Itératif (récursif)     | 4.72%       |
| XGBoost Rolling Refit           | 4.32%       |
| **Best Model (sélection auto)** | **4.22%**   |

→ **Amélioration de ~29%** vs baseline Naïf grâce à la sélection intelligente magasin par magasin.

Répartition des modèles retenus :
- Rolling Refit : ~70% 🥇
- Itératif : ~20%
- Naïf : ~10%

## 🛠️ Méthodologie
Quatre stratégies sont benchmarkées de manière équitable sur un horizon multi-step de 8 semaines :

- **Naïf saisonnier** : ventes = mêmes semaines de l’année précédente (lag 52) – baseline simple et interprétable
- **XGBoost CV one-step** : évaluation optimiste à une étape (référence technique)
- **XGBoost Itératif (récursif)** 🔄 : un seul entraînement, prédictions récursives avec réinjection des prédictions → rapide, scalable, conservateur
- **XGBoost Rolling Refit** 🔁 : réentraînement complet à chaque étape avec les valeurs réelles observées → plus adaptatif, généralement le plus précis, mais ~8-10× plus coûteux

Le modèle avec le **MAPE le plus bas** est retenu automatiquement **par magasin**.  
La pondération par CA permet d’obtenir un indicateur global business-oriented.

**Note sur la scalabilité** ⚡ :  
Le Rolling Refit offre la meilleure précision mais devient coûteux sur de très gros volumes.  
En cas de besoin de production plus rapide, il est possible de désactiver le benchmark Rolling ou de réduire les hyperparamètres.

## 🔑 Features utilisées
- `Holiday_Flag` 🎄 : flag hebdomadaire (présence d’un jour férié US ou Black Friday dans la semaine)
- `lag_1`, `lag_4`, `lag_52` : valeurs décalées
- `ma_4` : moyenne mobile 4 semaines
- `week_of_year` : numéro de semaine ISO

## ⚙️ Fonctionnalités du script
- Gestion intelligente des holidays US + Black Friday
- Backtesting multi-step réaliste (glissant sur plusieurs fenêtres)
- Sélection automatique du meilleur modèle par magasin
- Prévisions futures sur 8 semaines avec intervalles de confiance approximatifs (±1σ historique)
- Exports Excel PowerBI-ready 📈 :
  - `Histo_Prévisions_Par_Magasins` : historique + prévisions détaillées
  - `Histo_Prévisions_Consolidées` : vue agrégée
  - `Audit_des_modèles` : performances par magasin
  - `Synthèse_Audit` : indicateurs globaux pondérés (une ligne)

## 🏗️ Stack technique
- Python 3.9+
- Pandas, NumPy
- XGBoost
- Scikit-learn (metrics, TimeSeriesSplit)
- Openpyxl (export Excel)

## 🚀 Comment exécuter
```bash
# 1. Cloner le repo
git clone https://github.com/ton-pseudo/walmart-sales-forecasting.git
cd walmart-sales-forecasting

# 2. Installer les dépendances
pip install pandas numpy xgboost scikit-learn openpyxl

# 3. Placer ton fichier walmart.xlsm à la racine (ou modifier file_path)

# 4. Lancer
python walmart_forecast_final.py
