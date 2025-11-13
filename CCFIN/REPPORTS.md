# ===============================================
# Rapport détaillé automatique sur le dataset Iris
# DOI: 10.24432/C56C76
# Google Colab Ready
# ===============================================

# Installer et importer les bibliothèques nécessaires
!pip install pandas matplotlib seaborn --quiet

import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from sklearn.datasets import load_iris

# ------------------------------
# 1. Charger le dataset Iris
# ------------------------------
iris = load_iris()
X = iris.data
y = iris.target
features = iris.feature_names
species = iris.target_names

# Création d'un DataFrame
df = pd.DataFrame(X, columns=features)
df['species'] = [species[i] for i in y]

# ------------------------------
# 2. Informations générales sur le dataset
# ------------------------------
print("=== Informations générales sur le dataset Iris ===")
print(f"Nombre total d'échantillons : {df.shape[0]}")
print(f"Nombre de variables : {df.shape[1]-1} (hors variable cible)")
print(f"Nom des variables : {features}")
print(f"Classes présentes : {list(df['species'].unique())}\n")

# Nombre d'échantillons par classe
print("=== Répartition des classes ===")
print(df['species'].value_counts(), "\n")

# ------------------------------
# 3. Statistiques globales
# ------------------------------
print("=== Statistiques descriptives globales ===")
display(df.describe())

# ------------------------------
# 4. Statistiques par espèce
# ------------------------------
print("=== Statistiques par espèce (moyenne et écart-type) ===")
display(df.groupby('species').agg(['mean', 'std']))

# ------------------------------
# 5. Visualisations
# ------------------------------

# 5.1 Histogrammes pour chaque variable
df[features].hist(figsize=(12,10), bins=15, color='lightgreen', edgecolor='black')
plt.suptitle("Histogrammes des variables", fontsize=18)
plt.show()

# 5.2 Boxplots par espèce
plt.figure(figsize=(14,10))
for i, feature in enumerate(features):
    plt.subplot(2,2,i+1)
    sns.boxplot(x='species', y=feature, data=df, palette='Set2')
    plt.title(f"Boxplot de {feature} selon l'espèce", fontsize=12)
plt.tight_layout()
plt.show()

# 5.3 Pairplot pour visualiser les relations entre variables
sns.pairplot(df, hue='species', palette='Set2', diag_kind='hist', corner=False)
plt.suptitle("Pairplot des variables selon l'espèce", y=1.02, fontsize=16)
plt.show()

# 5.4 Matrice de corrélation
plt.figure(figsize=(8,6))
sns.heatmap(df[features].corr(), annot=True, cmap='coolwarm', fmt=".2f", linewidths=0.5)
plt.title("Matrice de corrélation des variables", fontsize=16)
plt.show()

# ------------------------------
# 6. Analyse approfondie par variable
# ------------------------------
print("=== Analyse approfondie par variable ===")
for feature in features:
    print(f"\nVariable : {feature}")
    print(f"  - Moyenne générale : {df[feature].mean():.2f}")
    print(f"  - Écart-type général : {df[feature].std():.2f}")
    for specie in species:
        mean_val = df[df['species']==specie][feature].mean()
        std_val = df[df['species']==specie][feature].std()
        print(f"    * {specie} -> moyenne: {mean_val:.2f}, écart-type: {std_val:.2f}")


