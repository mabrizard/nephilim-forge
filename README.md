Je ne suis pas développeur. Je suis Senior Manager Presales avec 20 ans d'expérience en enterprise software.

Ce projet est né d'une passion personnelle — le jeu de rôle — et d'une conviction : l'IA change qui peut créer, pas seulement comment on crée.

Le prompt engineering et les outils No-Code rendent aujourd'hui accessibles à des profils non-techniques ce qui était réservé aux développeurs.

48h. Zéro ligne écrite à la main. Une app en production.

---

# Nephilim — Forge des Âmes

Application web de création et gestion de personnages pour un jeu de rôle sur table. Projet personnel développé avec une stack No-Code / AI-assisted, du prototypage jusqu'au déploiement en production.

**Live** → [nephilim-forge.vercel.app](https://nephilim-forge.vercel.app)

---

## Stack technique

| Couche | Technologie |
|--------|------------|
| Frontend | HTML5 · CSS3 · JavaScript vanilla (ES2022) |
| Backend & base de données | [Supabase](https://supabase.com) (PostgreSQL + Auth + Realtime) |
| Déploiement | [Vercel](https://vercel.com) (CI/CD automatique sur push) |
| Versioning | Git + GitHub (dépôt public) |
| Développement | Prompt engineering via Claude AI (Anthropic) |

Architecture **single-page application** sans framework — un fichier HTML/CSS/JS autonome (~215 Ko, ~3 900 lignes) déployé comme site statique.

---

## Fonctionnalités techniques

### Frontend
- **Rendu dynamique** : moteur de rendu custom (`render()`) qui reconstruit le DOM à chaque changement d'état, sans Virtual DOM ni framework
- **State management** : objet `S` centralisé, persisté en `localStorage` (sauvegarde locale) et synchronisé vers Supabase (sauvegarde cloud)
- **Routing par état** : navigation entre 9 étapes gérée par un champ `step` dans le state, sans router externe
- **SVG dynamique** : génération programmatique d'un radar chart (Pentacle) calculé à partir des données du personnage
- **Composants réutilisables** : fonction `h()` de création de nœuds DOM typée (inspiration VNode), système de toasts, modales, overlays
- **Responsive** : mise en page CSS Grid + Flexbox, compatible desktop et mobile

### Backend (Supabase)
- **Authentification** : Magic Link (lien de connexion par email, sans mot de passe), gestion de session avec polling automatique
- **Base de données PostgreSQL** : 5 tables (`personnages`, `personnage_versions`, `sorts`, `invocations`, `formules`) avec Row Level Security — chaque utilisateur ne voit que ses propres données
- **Realtime** : souscription WebSocket sur la table `personnages` pour mise à jour en temps réel de l'interface MJ
- **Versionning des données** : historique des snapshots JSON du personnage avec métadonnées (label, date, PI, Ka)
- **600+ entrées de contenu** : sorts, invocations, formules stockés en base, filtrés et chargés dynamiquement avec cache en mémoire

### CI/CD
- Push Git → déploiement Vercel automatique en ~30 secondes
- Configuration `vercel.json` pour le routing des sous-pages (`/mj`)

---

## Architecture de l'application

```
nephilim-forge/
├── index.html          # Application principale (SPA)
└── mj/
    └── index.html      # Interface MJ (protégée par mot de passe local)
```

### Flux de données

```
Utilisateur → State S (mémoire)
           → localStorage (persistance locale, offline-first)
           → Supabase (sync cloud, si connecté)
                └── personnages (fiche active)
                └── personnage_versions (historique)
```

### Modèle de données principal

```json
{
  "nom": "string",
  "ka_dom": "Air | Feu | Terre | Eau | Lune",
  "profil": "Éveillé | Révélé | Immortel",
  "incarnations": { "[epoque_id]": { "vecu": "...", "savoirs": [], "arcane_id": "..." } },
  "magie": { "bas": 0, "haut": 0, "grand": 0 },
  "sephiroth": { "[id]": 0 },
  "alchimie": { "noir": 0, "blanc": 0, "rouge": 0 },
  "effets_magiques": { "gravés": [], "focus": [] },
  "chutes": { "khaiba": 0, "narcose": 0, "ombre": 0 }
}

```

## Interface MJ (Game Master dashboard)

Page séparée (`/mj`) avec :
- Authentification locale par mot de passe (hashé en `btoa`, sessionStorage)
- Vue temps réel de toutes les fiches joueurs via Supabase Realtime (WebSocket)
- Affichage du pentacle SVG, attributs calculés, sciences occultes, historique de versions
- Export TXT par joueur
- Calcul automatique des scores Ka selon le profil (Dom=5, Neutres=4, Opposés selon profil)

---

## Points techniques notables

- **Offline-first** : l'application fonctionne sans connexion (localStorage), la sync cloud est optionnelle
- **Zero-dependency frontend** : aucun npm, aucun bundler, aucun framework — chargement instantané
- **Déduplication de données** : script Python de nettoyage des doublons dans la source Excel avant import SQL (195 → 150 invocations après déduplication par clé composite)
- **Migration de données** : compatibilité ascendante dans `doLocalLoad()` — les anciennes fiches sont migrées automatiquement vers le nouveau schéma à la lecture
- **Sécurité** : clé `anon` Supabase publique par design, protégée par les politiques RLS côté base de données

---

## Prompt engineering

Ce projet a été développé en collaboration avec Claude (Anthropic) via un processus itératif de prompt engineering :

- Spécification fonctionnelle détaillée → génération de code
- Cycles de debug via analyse statique (Node.js `--check`) et tests manuels
- Patches ciblés sur des sections de code identifiées par grep/analyse
- Validation syntaxique systématique avant chaque déploiement

L'ensemble représente plusieurs dizaines d'itérations de prompts techniques, de la maquette initiale au produit en production.

---

## Auteur

Développé par **Marc-Alexandre Brizard** — passionné de JDR et de No-Code.
