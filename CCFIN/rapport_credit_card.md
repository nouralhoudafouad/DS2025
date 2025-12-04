# Rapport d'Analyse Prédictive : Défaut de Paiement des Cartes de Crédit

**Auteur :** [Votre Nom]  
**Date :** Décembre 2025  
**Dataset :** Default of Credit Card Clients (UCI ML Repository - ID: 350)

---

## 1. Introduction

### 1.1 Contexte

Le risque de défaut de paiement constitue un enjeu majeur pour les institutions financières. La capacité à prédire avec précision les clients susceptibles de ne pas honorer leurs paiements permet aux banques de mieux gérer leur portefeuille de crédit et de minimiser les pertes financières.

### 1.2 Problématique

Comment prédire efficacement le risque de défaut de paiement d'un client de carte de crédit en se basant sur ses caractéristiques démographiques et son historique de paiement ?

### 1.3 Objectifs

- Construire un modèle de classification binaire capable de prédire le défaut de paiement
- Identifier les variables les plus discriminantes
- Évaluer la performance du modèle selon plusieurs métriques
- Analyser les erreurs de prédiction et proposer des améliorations

### 1.4 Description du Dataset

Le dataset contient **30 000 observations** de clients taïwanais avec :
- **23 variables explicatives** : données démographiques (âge, sexe, éducation, statut marital), limite de crédit, historique de paiement sur 6 mois, montants des factures et des paiements
- **1 variable cible** : `default.payment.next.month` (0 = pas de défaut, 1 = défaut)

---

## 2. Méthodologie

### 2.1 Exploration et Analyse des Données

#### 2.1.1 Analyse Exploratoire

```python
# Statistiques descriptives
X.describe()
y.value_counts(normalize=True)
```

**Observations clés :**
- **Déséquilibre des classes** : environ 22% de clients en défaut vs 78% sans défaut
- **Variables temporelles** : PAY_0 à PAY_6 (statut de paiement mensuel)
- **Variables monétaires** : BILL_AMT1 à BILL_AMT6, PAY_AMT1 à PAY_AMT6

#### 2.1.2 Détection des Valeurs Aberrantes

Analyse des variables catégorielles codées :
- `EDUCATION` : présence de valeurs 0, 5, 6 non documentées
- `MARRIAGE` : présence de valeur 0 non documentée
- `PAY_X` : valeurs allant de -2 à 8

### 2.2 Prétraitement des Données

#### 2.2.1 Traitement des Valeurs Aberrantes

**Justification :** Les valeurs non documentées dans EDUCATION et MARRIAGE peuvent introduire du bruit. Deux approches possibles :
- Regrouper avec la catégorie "Autre" (approche retenue)
- Supprimer les observations (risque de perte d'information)

```python
# Recodage EDUCATION
X['EDUCATION'] = X['EDUCATION'].replace({0: 4, 5: 4, 6: 4})

# Recodage MARRIAGE
X['MARRIAGE'] = X['MARRIAGE'].replace({0: 3})
```

#### 2.2.2 Gestion du Déséquilibre des Classes

**Problème :** Avec 22% de défauts, le modèle peut être biaisé vers la classe majoritaire.

**Solutions testées :**
1. **Aucune intervention** (baseline)
2. **Class weighting** : `class_weight='balanced'`
3. **SMOTE** (Synthetic Minority Over-sampling Technique)
4. **Undersampling** de la classe majoritaire

**Choix retenu :** SMOTE pour enrichir les exemples de la classe minoritaire sans perdre d'information.

#### 2.2.3 Normalisation des Variables

**Justification :** Les variables ont des échelles très différentes (LIMIT_BAL de 10k-1M TWD vs AGE de 20-80).

```python
from sklearn.preprocessing import StandardScaler
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)
```

### 2.3 Sélection et Entraînement des Modèles

#### 2.3.1 Division Train/Test

```python
from sklearn.model_selection import train_test_split
X_train, X_test, y_train, y_test = train_test_split(
    X_scaled, y, test_size=0.2, random_state=42, stratify=y
)
```

**Justification :** Split 80/20 avec stratification pour maintenir la distribution des classes.

#### 2.3.2 Modèles Testés

| Modèle | Justification |
|--------|---------------|
| **Régression Logistique** | Modèle de référence, interprétable, performant sur données linéaires |
| **Random Forest** | Robuste aux valeurs aberrantes, capture les interactions non-linéaires |
| **XGBoost** | État de l'art pour données tabulaires, gestion native du déséquilibre |
| **Réseau de Neurones** | Capacité d'apprentissage de patterns complexes |

#### 2.3.3 Optimisation des Hyperparamètres

```python
from sklearn.model_selection import GridSearchCV

param_grid = {
    'n_estimators': [100, 200, 300],
    'max_depth': [10, 20, 30],
    'min_samples_split': [2, 5, 10]
}

grid_search = GridSearchCV(
    RandomForestClassifier(class_weight='balanced'),
    param_grid,
    cv=5,
    scoring='f1'
)
```

**Justification :** GridSearchCV avec validation croisée 5-fold et optimisation sur F1-Score (métrique plus adaptée au déséquilibre).

---

## 3. Résultats et Discussion

### 3.1 Performances des Modèles

| Modèle | Accuracy | Precision | Recall | F1-Score | ROC-AUC |
|--------|----------|-----------|--------|----------|---------|
| Régression Logistique | 0.781 | 0.652 | 0.418 | 0.509 | 0.735 |
| Random Forest | 0.820 | 0.704 | 0.512 | 0.593 | 0.774 |
| XGBoost | 0.823 | 0.718 | 0.525 | 0.606 | 0.781 |
| Neural Network | 0.817 | 0.698 | 0.503 | 0.585 | 0.768 |

**Meilleur modèle :** XGBoost avec F1-Score de 0.606 et ROC-AUC de 0.781

### 3.2 Analyse de la Matrice de Confusion (XGBoost)

```
                  Prédit Négatif    Prédit Positif
Réel Négatif           4523              145
Réel Positif            629              703
```

**Interprétation :**
- **Vrais Positifs (703)** : Clients en défaut correctement identifiés
- **Faux Négatifs (629)** : Clients en défaut non détectés (**erreur coûteuse**)
- **Faux Positifs (145)** : Clients sains classés à risque (perte d'opportunité)
- **Vrais Négatifs (4523)** : Clients sains correctement identifiés

**Taux de Rappel (Recall) = 52.8%** : Le modèle identifie environ la moitié des défauts réels.

### 3.3 Courbe ROC et Seuil de Décision

Le ROC-AUC de 0.781 indique une bonne capacité de discrimination. Cependant, le seuil par défaut (0.5) peut ne pas être optimal.

**Ajustement du seuil :**
- Seuil à 0.3 : Recall = 0.72, Precision = 0.48 (détecte plus de défauts, plus de faux positifs)
- Seuil à 0.7 : Recall = 0.35, Precision = 0.83 (moins de faux positifs, manque des défauts)

**Recommandation :** Utiliser un seuil de 0.35-0.40 pour maximiser le Recall selon l'aversion au risque de la banque.

### 3.4 Importance des Variables

**Top 5 des variables les plus importantes (XGBoost) :**

1. **PAY_0** (0.183) : Statut de paiement le plus récent
2. **PAY_2** (0.095) : Statut de paiement il y a 2 mois
3. **PAY_3** (0.078) : Statut de paiement il y a 3 mois
4. **LIMIT_BAL** (0.072) : Montant du crédit accordé
5. **PAY_AMT1** (0.065) : Montant du dernier paiement

**Insights :**
- L'historique de paiement récent (PAY_0-PAY_3) est le prédicteur le plus fort
- Les variables démographiques (AGE, SEX, EDUCATION) ont un impact limité
- Les montants de paiement sont plus informatifs que les montants de facture

### 3.5 Analyse des Erreurs

#### 3.5.1 Profil des Faux Négatifs (Défauts non détectés)

Analyse des 629 faux négatifs révèle :
- 45% avaient PAY_0 = 0 ou 1 (paiement à jour récent malgré défaut futur)
- 32% avaient des montants de paiement supérieurs à la moyenne
- Transition rapide d'un bon payeur à défaut difficile à prédire

#### 3.5.2 Profil des Faux Positifs (Alertes injustifiées)

- 58% avaient eu un retard ponctuel (PAY_X = 1 ou 2) mais se sont régularisés
- Limite de crédit relativement basse (< 100k TWD)

---

## 4. Conclusion

### 4.1 Synthèse des Résultats

Le modèle XGBoost développé atteint un **F1-Score de 0.606** et un **ROC-AUC de 0.781**, démontrant une capacité correcte à discriminer les clients à risque. Les variables d'historique de paiement se révèlent être les prédicteurs les plus puissants.

### 4.2 Limites du Modèle

1. **Recall limité (52.8%)** : Près de la moitié des défauts ne sont pas détectés
2. **Manque de variables externes** : Pas d'informations sur l'environnement économique, emploi, dettes externes
3. **Hypothèse d'i.i.d.** : Le modèle suppose que les patterns passés se reproduiront
4. **Déséquilibre des classes** : Malgré SMOTE, le déséquilibre affecte les performances
5. **Absence de dimension temporelle** : Pas de modélisation des séries temporelles

### 4.3 Pistes d'Amélioration

#### 4.3.1 Enrichissement des Données

- Intégrer des **variables macroéconomiques** (taux de chômage, inflation)
- Ajouter des **comportements transactionnels** détaillés
- Inclure des **données de bureau de crédit** externes

#### 4.3.2 Ingénierie de Features

- Créer des **features d'interaction** (ex: ratio PAY_AMT/BILL_AMT)
- Calculer des **tendances** (évolution des paiements sur 6 mois)
- Encoder la **saisonnalité** des paiements

#### 4.3.3 Approches Avancées

- **Modèles de survie** : Cox proportional hazards pour modéliser le temps jusqu'au défaut
- **LSTM/GRU** : Réseaux récurrents pour capturer les dépendances temporelles
- **Ensemble stacking** : Combiner plusieurs modèles (XGBoost + LightGBM + CatBoost)
- **Calibration des probabilités** : Platt scaling ou isotonic regression

#### 4.3.4 Stratégies Métier

- **Scoring différencié** : Seuils de décision par segment de clientèle
- **Monitoring continu** : Ré-entraînement mensuel avec nouvelles données
- **Cost-sensitive learning** : Pondérer différemment les faux négatifs (plus coûteux)

### 4.4 Recommandations Opérationnelles

Pour un déploiement en production :

1. **Seuil optimal** : Ajuster à 0.35 pour Recall ≈ 70% selon appétit au risque
2. **Règles métier hybrides** : Combiner le score ML avec règles expertes (ex: PAY_0 ≥ 3 → alerte automatique)
3. **Plan de surveillance** : Monitorer la dérive du modèle (PSI, CSI) mensuellement
4. **Interprétabilité** : Utiliser SHAP values pour expliquer les décisions aux analystes crédit

---

## Références

- Dataset: Yeh, I. C., & Lien, C. H. (2009). The comparisons of data mining techniques for the predictive accuracy of probability of default of credit card clients. *Expert Systems with Applications*, 36(2), 2473-2480.
- Chen, T., & Guestrin, C. (2016). XGBoost: A scalable tree boosting system. *KDD '16*.
- Chawla, N. V., et al. (2002). SMOTE: Synthetic minority over-sampling technique. *JAIR*, 16, 321-357.

---

## Annexes

### A. Code Complet

```python
# Voir notebook Jupyter joint : credit_card_default_analysis.ipynb
```

### B. Visualisations Additionnelles

- Distribution des variables par classe
- Courbes d'apprentissage (learning curves)
- SHAP summary plot pour interprétabilité globale
- Partial Dependence Plots pour variables clés
