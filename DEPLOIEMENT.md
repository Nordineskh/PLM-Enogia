# Guide de déploiement — PLM Enogia

## Étape 1 — Créer le repo GitHub

### 1.1 Créer le repo sur GitHub
1. Aller sur [github.com/organizations/enogia](https://github.com) (ou votre compte)
2. Cliquer **New repository**
3. Remplir :
   - **Name** : `plm-enogia`
   - **Visibility** : `Private` ← important
   - **Description** : PLM Enogia — Source unique de vérité produit
4. Ne pas cocher "Add README" (on a le nôtre)
5. Cliquer **Create repository**

---

### 1.2 Pousser le code depuis votre machine

```bash
# Cloner / initialiser le repo local
cd plm-enogia-repo

git init
git add .
git commit -m "feat: PLM Enogia initial — charte graphique + tous modules"

# Connecter au repo GitHub (remplacer VOTRE-ORG par votre org ou compte)
git remote add origin https://github.com/VOTRE-ORG/plm-enogia.git
git branch -M main
git push -u origin main
```

---

## Étape 2 — Activer GitHub Pages

### 2.1 Activer dans les paramètres
1. Dans le repo GitHub → **Settings** → **Pages**
2. **Source** : `GitHub Actions`
3. Sauvegarder

### 2.2 Ajouter les permissions au workflow
Dans **Settings** → **Actions** → **General** :
- **Workflow permissions** → `Read and write permissions` ✓
- `Allow GitHub Actions to create and approve pull requests` ✓

### 2.3 Déclencher le premier déploiement
```bash
# Le push sur main déclenche automatiquement le workflow
# Vérifier dans : Actions → Deploy PLM → GitHub Pages

# URL finale de votre PLM :
# https://VOTRE-ORG.github.io/plm-enogia/
```

---

## Étape 3 — Configurer les secrets GitHub

Pour la phase Firebase (étape suivante), ajouter les secrets :

1. **Settings** → **Secrets and variables** → **Actions**
2. Cliquer **New repository secret** pour chacun :

| Secret | Description |
|--------|-------------|
| `FIREBASE_API_KEY` | Clé API Firebase |
| `FIREBASE_PROJECT_ID` | ID du projet Firebase |
| `FIREBASE_APP_ID` | App ID Firebase |

---

## Étape 4 — Workflow Git quotidien

### Développement d'une nouvelle feature

```bash
# 1. Mettre à jour depuis main
git checkout main
git pull origin main

# 2. Créer une branche feature
git checkout -b feature/import-excel-produits

# 3. Modifier le code
# ... éditer public/index.html ...

# 4. Committer avec convention de message
git add .
git commit -m "feat: import Excel références produits"

# 5. Pousser la branche
git push origin feature/import-excel-produits

# 6. Créer une Pull Request sur GitHub
# → La PR déclenche les checks qualité automatiques
# → Merger dans main → déploiement automatique
```

### Convention des messages de commit

```
feat:     nouvelle fonctionnalité
fix:      correction de bug
style:    changement visuel / charte graphique
data:     modification des données / BOM
docs:     documentation
ci:       GitHub Actions / déploiement
chore:    maintenance
```

---

## Étape 5 — Vérifier le déploiement

```bash
# Voir les runs GitHub Actions
# → Onglet "Actions" du repo

# Vérifier l'URL de déploiement
# → Actions → dernier run → "Déploiement GitHub Pages" → URL affichée

# En cas d'erreur
# → Cliquer sur le job en échec → lire les logs
```

---

## Structure des branches recommandée

```
main ──────────────────────────────────► Production (GitHub Pages)
  │
  ├── develop ──────────────────────────► Intégration
  │     │
  │     ├── feature/import-excel
  │     ├── feature/firebase-auth
  │     └── feature/bom-avancee
  │
  └── hotfix/* ─────────────────────────► Corrections urgentes
```

---

## Prochaine étape : Firebase

Une fois GitHub opérationnel, passer à la migration Firebase :
→ Voir `docs/FIREBASE.md`

La migration remplace `window.storage` par Firestore,
ajoute l'authentification `@enogia.fr` et permet
la collaboration multi-utilisateurs en temps réel.
