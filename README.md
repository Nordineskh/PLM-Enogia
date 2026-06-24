# PLM Enogia — Source unique de vérité

Application PLM (Product Lifecycle Management) interne Enogia.  
Hébergée sur GitHub Pages · Données sur Firebase Firestore.

---

## Stack technique

| Couche | Technologie |
|--------|-------------|
| Frontend | HTML · CSS · JavaScript vanilla |
| Hébergement | GitHub Pages (branche `gh-pages`) |
| Base de données | Firebase Firestore (à venir) |
| Authentification | Firebase Auth — domaine `@enogia.fr` (à venir) |
| CI/CD | GitHub Actions |

---

## Structure du repo

```
plm-enogia/
├── public/
│   └── index.html          ← Application PLM complète
├── src/
│   ├── firebase.js         ← Config Firebase (à venir)
│   ├── auth.js             ← Authentification (à venir)
│   └── db.js               ← Couche Firestore (à venir)
├── docs/
│   ├── ARCHITECTURE.md     ← Architecture technique
│   ├── DEPLOIEMENT.md      ← Guide de déploiement
│   └── FIREBASE.md         ← Guide migration Firebase
├── .github/
│   └── workflows/
│       ├── deploy.yml      ← Deploy automatique → GitHub Pages
│       └── quality.yml     ← Vérifications qualité
├── .gitignore
├── .env.example            ← Variables d'environnement Firebase
└── README.md
```

---

## Démarrage rapide

### 1. Cloner le repo
```bash
git clone https://github.com/enogia/plm-enogia.git
cd plm-enogia
```

### 2. Ouvrir en local
```bash
# Aucune dépendance requise — ouvrir directement
open public/index.html
# ou avec un serveur local
npx serve public
```

### 3. Déployer
```bash
git add .
git commit -m "feat: description du changement"
git push origin main
# → GitHub Actions déploie automatiquement sur GitHub Pages
```

---

## Branches

| Branche | Rôle |
|---------|------|
| `main` | Code stable — déclenche le déploiement production |
| `develop` | Intégration des features en cours |
| `feature/*` | Une branche par fonctionnalité |
| `gh-pages` | Branche générée automatiquement par GitHub Actions |

---

## Variables d'environnement

Copier `.env.example` en `.env` et remplir les valeurs Firebase :

```bash
cp .env.example .env
```

> ⚠️ Ne jamais committer le fichier `.env` — il est dans `.gitignore`

---

## Contribuer

1. Créer une branche `feature/nom-de-la-feature`
2. Développer et tester en local
3. Ouvrir une Pull Request vers `develop`
4. Review → merge → déploiement automatique

---

## Contacts

| Rôle | Responsable |
|------|-------------|
| Référent PLM | À définir |
| Référent technique | À définir |
| Admin GitHub/Firebase | À définir |

---

*Enogia — Marseille · [www.enogia.com](https://www.enogia.com)*
