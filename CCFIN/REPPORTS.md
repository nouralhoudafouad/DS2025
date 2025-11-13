# ===============================================
# Analyse du dataset Iris avec ACP
# DOI: 10.24432/C56C76
# ===============================================

# 1. Importer les bibliothèques nécessaires
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from sklearn.datasets import load_iris
from sklearn.preprocessing import StandardScaler
from sklearn.decomposition import PCA

# 2. Charger le dataset Iris
iris = load_iris()
X = iris.data      # Variables quantitatives
y = iris.target    # Classes
features = iris.feature_names
species = iris.target_names

# Créer un DataFrame pour faciliter l'analyse
df = pd.DataFrame(X, columns=features)
df['species'] = [species[i] for i in y]

# 3. Statistiques descriptives
print("=== Statistiques descriptives ===")
print(df.describe())

# 4. Visualisation exploratoire
# 4.1 Pairplot pour visualiser les relations entre variables
sns.pairplot(df, hue='species', palette='Set2')
plt.suptitle("Pairplot des attributs selon l'espèce", y=1.02)
plt.show()

# 4.2 Heatmap des corrélations
plt.figure(figsize=(8,6))
sns.heatmap(df[features].corr(), annot=True, cmap='coolwarm')
plt.title("Matrice de corrélation des variables")
plt.show()

# 5. Standardisation des données (très important pour l'ACP)
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# 6. Application de l'ACP
pca = PCA(n_components=2)  # Réduction à 2 composantes principales
X_pca = pca.fit_transform(X_scaled)

# Variance expliquée
print("\n=== Variance expliquée par chaque composante ===")
for i, var in enumerate(pca.explained_variance_ratio_):
    print(f"PC{i+1}: {var*100:.2f}%")

# 7. Visualisation des composantes principales
plt.figure(figsize=(8,6))
for i, specie in enumerate(species):
    plt.scatter(X_pca[y==i,0], X_pca[y==i,1], label=specie)
plt.xlabel('PC1')
plt.ylabel('PC2')
plt.title('Projection des Iris sur les 2 premières composantes principales')
plt.legend()
plt.grid(True)
plt.show()

# 8. Contribution des variables aux composantes principales
loadings = pd.DataFrame(pca.components_.T, columns=['PC1','PC2'], index=features)
print("\n=== Charges des variables sur les composantes principales ===")
print(loadings)

# 9. Biplot (optionnel)
plt.figure(figsize=(8,6))
plt.scatter(X_pca[:,0], X_pca[:,1], c=y, cmap='Set2', alpha=0.7)
for i, feature in enumerate(features):
    plt.arrow(0, 0, loadings.PC1[i]*3, loadings.PC2[i]*3, 
              color='r', alpha=0.7, head_width=0.1)
    plt.text(loadings.PC1[i]*3.2, loadings.PC2[i]*3.2, feature, color='r')
plt.xlabel('PC1')
plt.ylabel('PC2')
plt.title('Biplot des 2 premières composantes principales')
plt.grid()
plt.show()


