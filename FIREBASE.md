# Migration Firebase — PLM Enogia

## Pourquoi Firebase ?

| Besoin PLM | Solution Firebase |
|-----------|-------------------|
| Données partagées entre utilisateurs | Firestore (temps réel) |
| Authentification `@enogia.fr` | Firebase Auth |
| Historique des modifications | Firestore + timestamps |
| Accès multi-rôles (BE, Achats...) | Firestore Security Rules |
| Pas de serveur à gérer | Firebase = serverless |

---

## Étape 1 — Créer le projet Firebase

1. Aller sur [console.firebase.google.com](https://console.firebase.google.com)
2. **Add project** → Nom : `enogia-plm`
3. Désactiver Google Analytics (optionnel)
4. **Create project**

### Activer Firestore
1. **Build** → **Firestore Database** → **Create database**
2. Mode : **Production mode** (règles strictes dès le départ)
3. Région : `europe-west3` (Frankfurt — données en Europe)

### Activer Authentication
1. **Build** → **Authentication** → **Get started**
2. Activer **Email/Password**
3. Optionnel : activer **Google** (connexion avec compte Google `@enogia.fr`)

### Récupérer la config
1. **Project Settings** → **Your apps** → **Add app** → icône Web `</>`
2. App nickname : `PLM Enogia`
3. Copier l'objet `firebaseConfig`

---

## Étape 2 — Structure Firestore

```
firestore/
├── produits/                    ← Collection principale
│   └── {produitId}/
│       ├── id: "PF-2201"
│       ├── name: "Échangeur compact EC-40"
│       ├── phase: "Développement"
│       ├── type: "Catalogue"
│       ├── resp: "M. Durand"
│       ├── rev: "B"
│       ├── statut: "Attente"
│       ├── maj: Timestamp
│       └── desc: "..."
│
├── bom/                         ← Nomenclatures BOM
│   └── {produitId}/
│       └── nodes: [ ...composants... ]
│
├── eco/                         ← ECR / ECO
│   └── {ecoId}/
│       ├── type: "ECO"
│       ├── titre: "..."
│       ├── produit: "PF-2201"
│       ├── statut: "En cours"
│       └── createdAt: Timestamp
│
├── documents/                   ← GED
│   └── {docId}/
│       ├── name: "..."
│       ├── statut: "Signé"
│       └── ...
│
├── fournisseurs/                ← Fournisseurs
├── flags/                       ← Anomalies IA
└── journal/                     ← Traçabilité complète
    └── {eventId}/
        ├── type: "modification"
        ├── produit: "PF-2201"
        ├── msg: "..."
        ├── user: "m.durand@enogia.fr"
        └── date: Timestamp
```

---

## Étape 3 — Règles de sécurité Firestore

Copier dans **Firestore** → **Rules** :

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Seuls les utilisateurs authentifiés avec @enogia.fr ont accès
    function isEnogiaUser() {
      return request.auth != null &&
             request.auth.token.email.matches('.*@enogia\\.fr$');
    }

    // Lecture : tous les utilisateurs Enogia
    // Écriture : tous les utilisateurs Enogia (ajuster par rôle si besoin)
    match /{collection}/{document=**} {
      allow read: if isEnogiaUser();
      allow write: if isEnogiaUser();
    }

    // Journal : lecture seule (pas de suppression possible)
    match /journal/{eventId} {
      allow read: if isEnogiaUser();
      allow create: if isEnogiaUser();
      allow update, delete: if false; // immuable
    }
  }
}
```

---

## Étape 4 — Remplacement de window.storage par Firestore

Dans `public/index.html`, remplacer la section `loadDB / saveDB` :

```html
<!-- Ajouter avant </body> -->
<script type="module">
  import { initializeApp } from 'https://www.gstatic.com/firebasejs/10.7.0/firebase-app.js';
  import { getFirestore, collection, getDocs, doc, setDoc, addDoc,
           onSnapshot, serverTimestamp }
    from 'https://www.gstatic.com/firebasejs/10.7.0/firebase-firestore.js';
  import { getAuth, signInWithEmailAndPassword, onAuthStateChanged }
    from 'https://www.gstatic.com/firebasejs/10.7.0/firebase-auth.js';

  const firebaseConfig = {
    apiKey: "__FIREBASE_API_KEY__",
    authDomain: "__FIREBASE_PROJECT_ID__.firebaseapp.com",
    projectId: "__FIREBASE_PROJECT_ID__",
    storageBucket: "__FIREBASE_PROJECT_ID__.appspot.com",
    appId: "__FIREBASE_APP_ID__"
  };

  const app = initializeApp(firebaseConfig);
  const db = getFirestore(app);
  const auth = getAuth(app);

  // Remplace loadDB()
  async function loadFromFirestore() {
    const [produitsSnap, ecoSnap, docsSnap] = await Promise.all([
      getDocs(collection(db, 'produits')),
      getDocs(collection(db, 'eco')),
      getDocs(collection(db, 'documents'))
    ]);
    DB.produits = produitsSnap.docs.map(d => d.data());
    DB.eco = ecoSnap.docs.map(d => d.data());
    DB.documents = docsSnap.docs.map(d => d.data());
    renderAll();
  }

  // Remplace saveDB() — sauvegarde un produit spécifique
  async function saveProduitFirestore(produit) {
    await setDoc(doc(db, 'produits', produit.id), produit);
  }

  // Listener temps réel sur les flags
  onSnapshot(collection(db, 'flags'), (snapshot) => {
    DB.flags = snapshot.docs.map(d => d.data());
    updateNavBadges();
    renderDashboard();
  });

  // Auth guard
  onAuthStateChanged(auth, (user) => {
    if (user && user.email.endsWith('@enogia.fr')) {
      loadFromFirestore();
    } else {
      // Afficher écran de connexion
      showLoginScreen();
    }
  });
</script>
```

---

## Étape 5 — Ajouter les secrets GitHub

```bash
# Dans GitHub → Settings → Secrets and variables → Actions

FIREBASE_API_KEY        = votre-api-key
FIREBASE_PROJECT_ID     = enogia-plm
FIREBASE_APP_ID         = votre-app-id
```

Le workflow `deploy.yml` injecte automatiquement ces valeurs
dans le HTML à chaque déploiement.

---

## Étape 6 — Créer les premiers utilisateurs

Dans **Firebase Console** → **Authentication** → **Users** :

```
m.durand@enogia.fr      → rôle : responsable produit
c.martin@enogia.fr      → rôle : responsable produit
a.bernard@enogia.fr     → rôle : responsable produit
admin@enogia.fr         → rôle : admin PLM
```

---

## Récapitulatif migration

```
Avant (actuel)          Après (Firebase)
─────────────────────   ──────────────────────────────
window.storage          Firestore (cloud, temps réel)
Données locales         Partagées entre tous les users
Pas d'auth              Firebase Auth @enogia.fr
Pas de temps réel       onSnapshot() — sync instantané
Pas de traçabilité      Journal Firestore immuable
```

---

## Coût estimé

Firebase Spark (gratuit) couvre largement le besoin Enogia :
- 50 000 lectures/jour
- 20 000 écritures/jour
- 1 Go de stockage Firestore

Pour référence : le PLM Enogia génère environ 500-2000 lectures/jour.
**Coût prévisible : 0 €/mois sur le plan gratuit.**
