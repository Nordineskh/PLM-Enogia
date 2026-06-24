# Architecture technique — PLM Enogia

## Vue d'ensemble

```
┌─────────────────────────────────────────────────────────┐
│                    UTILISATEURS ENOGIA                   │
│   Bureau d'études · Méthodes · Achats · Production       │
└──────────────────────┬──────────────────────────────────┘
                       │  HTTPS (navigateur)
                       ▼
┌─────────────────────────────────────────────────────────┐
│              GitHub Pages (CDN GitHub)                   │
│                  public/index.html                        │
│           Abel + Roboto (Google Fonts CDN)                │
└──────────────────────┬──────────────────────────────────┘
                       │  Firebase SDK (JS module)
                       ▼
┌─────────────────────────────────────────────────────────┐
│                  FIREBASE (Europe West)                   │
│                                                          │
│  ┌─────────────────┐    ┌───────────────────────────┐   │
│  │  Firebase Auth  │    │   Firestore Database       │   │
│  │                 │    │                            │   │
│  │  @enogia.fr     │    │  produits/                 │   │
│  │  Email + Google │    │  bom/                      │   │
│  └─────────────────┘    │  eco/                      │   │
│                         │  documents/                │   │
│                         │  fournisseurs/             │   │
│                         │  flags/                    │   │
│                         │  journal/   (immuable)     │   │
│                         └───────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
                       ▲
                       │  Futures intégrations
┌─────────────────────────────────────────────────────────┐
│              SYSTÈMES ENOGIA (via API REST)              │
│                                                          │
│   SolidWorks PDM ──► Firebase Functions (webhook)        │
│   ERP Sage X3    ──► Firebase Functions (sync BOM)       │
│   MES Atelier    ──► Firebase Functions (statuts)        │
└─────────────────────────────────────────────────────────┘
```

## CI/CD Pipeline

```
Developer
   │
   │  git push origin feature/xxx
   ▼
GitHub Repository (private)
   │
   ├── Pull Request → quality.yml
   │     ├── Vérification structure
   │     └── Détection secrets exposés
   │
   │  Merge → main
   ▼
deploy.yml (GitHub Actions)
   ├── Checkout code
   ├── Injection secrets Firebase
   ├── Ajout métadonnées (SHA, date)
   ├── Upload artifact
   └── Deploy → GitHub Pages
         │
         ▼
   https://enogia.github.io/plm-enogia/
```

## Modèle de données

### Produit
```typescript
interface Produit {
  id: string;           // "PF-2201"
  name: string;         // "Échangeur compact EC-40"
  type: "Catalogue" | "ETO";
  phase: "Concept" | "Développement" | "Validation" | "Production" | "Fin de vie";
  rev: string;          // "A", "B", "C"...
  resp: string;         // email ou nom
  famille: string;      // "ORC", "Échangeur"...
  statut: string;
  maj: Timestamp;
  desc: string;
  createdAt: Timestamp;
  createdBy: string;    // email utilisateur
}
```

### Événement journal (immuable)
```typescript
interface JournalEvent {
  id: string;
  type: "creation" | "modification" | "validation" | "eco" | "sync";
  produit: string;
  msg: string;
  user: string;         // email @enogia.fr
  date: Timestamp;
  // Jamais modifiable — règles Firestore : allow update, delete: if false
}
```

## Sécurité

- Repo GitHub : **Private** — accès équipe Enogia uniquement
- Secrets Firebase : GitHub Secrets (jamais dans le code)
- Firestore Rules : lecture/écriture restreinte aux `@enogia.fr`
- Journal : immuable par règle Firestore (audit trail)
- HTTPS : garanti par GitHub Pages + Firebase Hosting

## Performances estimées

| Métrique | Valeur |
|----------|--------|
| Taille index.html | ~67 Ko |
| Chargement initial | < 1s (CDN GitHub) |
| Sync Firestore | Temps réel (< 100ms) |
| Disponibilité | 99.9% (GitHub SLA) |
| Coût mensuel | 0 € (plans gratuits) |
