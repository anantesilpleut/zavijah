# Palette° v0.18 — Déploiement PWA / app mobile

## Contenu du package
```
palette-app/
├── index.html              ← l'app complète (identique à palette-mvp.html)
├── manifest.webmanifest    ← métadonnées d'installation
├── sw.js                   ← service worker (offline + cache CDN)
├── icons/
│   ├── icon-192.png
│   ├── icon-512.png
│   └── icon-512-maskable.png
└── DEPLOY.md
```

## Déploiement (Cloudflare Pages — ~3 minutes)
1. https://pages.cloudflare.com → Create project → Direct upload
2. Glisser le dossier `palette-app/` entier
3. Deploy. HTTPS automatique — indispensable : caméra, PWA et service worker exigent HTTPS.

Alternatives équivalentes : Vercel (`vercel deploy`), Netlify (drag & drop), GitHub Pages.

## Installation mobile (ce qui se passe côté utilisateur)
- **Android/Chrome** : bannière d'installation automatique après ~30 s d'usage,
  ou menu ⋮ → « Installer l'application ». Icône sur l'écran d'accueil, plein
  écran sans barre d'URL, splash screen aux couleurs Palette°.
- **iOS/Safari** : Partager → « Sur l'écran d'accueil ». Même résultat.
  Limites iOS connues : pas de bannière automatique ; IndexedDB peut être purgé
  par le système après ~7 jours sans utilisation (l'export JSON reste le backup).

## Ce qui marche hors-ligne après une première visite en ligne
- L'app entière (shell réseau-d'abord avec repli cache)
- Les modèles MediaPipe (~16 Mo, mis en cache à la première analyse)
- Les polices
- L'historique et les ancres (IndexedDB, indépendant du réseau)

Nécessite le réseau : recommandations produits réels (Shopify), rien d'autre.

## Mise à jour
Re-uploader `index.html` (et bumper `VERSION` dans `sw.js` : `palette-v018` → `v019`).
Le service worker propage aux clients au rechargement suivant.

## Plus tard, si distribution via stores souhaitée
Le même dossier s'empaquette sans modification :
- **Android (Play Store)** : Bubblewrap (`npx @bubblewrap/cli init --manifest <url>/manifest.webmanifest`) → Trusted Web Activity → AAB signé.
- **iOS (App Store)** : Capacitor (`npx cap add ios`) avec ce dossier en webDir. Nécessite un Mac + compte développeur Apple.
La PWA d'abord : zéro friction, zéro validation de store, mêmes fonctionnalités.
