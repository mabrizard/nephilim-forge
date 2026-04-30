# Nephilim — Forge des Âmes

Application web de création et gestion de personnages pour un jeu de rôle de table. Projet personnel développé avec une stack No-Code / AI-assisted, du prototypage jusqu'au déploiement en production.

**Live** → [nephilim-forge.vercel.app](https://nephilim-forge.vercel.app)

---

## Stack technique

| Couche | Technologie |
|--------|------------|
| Frontend | HTML5 · CSS3 · JavaScript vanilla (ES2022) |
| Backend & base de données | [Supabase](https://supabase.com) (PostgreSQL + Auth + Realtime) |
| Déploiement | [Vercel](https://vercel.com) (CI/CD automatique sur push) |
| Versioning | Git + GitHub |
| Développement | Prompt engineering via Claude AI (Anthropic) |

Architecture **single-page application** sans framework — un fichier HTML/CSS/JS autonome (~360 Ko, ~6 300 lignes) déployé comme site statique.

---

## Fonctionnalités techniques

### Frontend
- **Rendu dynamique** : moteur de rendu custom (`render()`) qui reconstruit le DOM à chaque changement d'état, sans Virtual DOM ni framework
- **State management** : objet `S` centralisé et typé, persisté en `localStorage` (offline-first) et synchronisé vers Supabase (cloud)
- **Routing par état** : navigation entre 9 étapes gérée par un champ `step` dans le state, sans router externe
- **SVG dynamique** : génération programmatique d'un radar chart (Pentacle) calculé en temps réel à partir des scores du personnage
- **Composants réutilisables** : fonction `h()` de création de nœuds DOM (inspiration VNode), système de toasts, modales, overlays, dropdowns
- **Calculs de règles** : moteur complet — scores Ka, attributs dérivés, budget PI, pools de degrés gratuits, limites d'époques par profil, sorts gravés vs focus
- **Grimoire interactif** : chargement asynchrone depuis Supabase avec cache mémoire, filtres multi-critères, vérification d'accessibilité par élément/cercle/construct (y compris éléments composites `Terre/Eau`)
- **Export PDF** : génération d'un document HTML stylisé via Blob URL (contournement des popup blockers), prêt pour impression navigateur
- **Responsive** : barre d'action compacte avec menu contextuel `⋯` pour mobile, CSS Grid + Flexbox

### Système de règles implémenté
- **9 étapes de création** guidées : identité, Ka, profil, incarnations, sciences occultes, alchimie, simulacre, attributs, fiche
- **Arbre de Kabbale** : graphe de prérequis avec logique OR multi-nœuds (ex : Tiphereth accessible depuis Hod OU Yesod OU Netzah)
- **5 Mondes kabbaliques** : Zakaï, Meborack, Sohar, Aresh, Pachad — avec logique pacte permanent/éphémère selon le Monde Tutélaire
- **Grimoire canonique** : 79 sorts, 85 invocations, 90 formules alchimiques avec vérification d'accessibilité complète
- **Incarnations** : 31 époques historiques avec limites canoniques par profil (Éveillé : 3 payantes + 2 gratuites, Révélé : 5+2, Immortel : 7+2), distinction époques gratuites (Atlantide, Arcadia) vs payantes
- **Sorts gravés/focus** : calcul automatique via `calcSortsInfo()` — gravés limités au Ka dominant, surplus en focus
- **Simulacre** : 20 profils canoniques avec Ka-Soleil, vécu contemporain, ressources, contraintes et alerte Effet Rosenkreutz

### Backend (Supabase)
- **Authentification** : Magic Link (email, sans mot de passe), gestion de session avec polling automatique
- **PostgreSQL** : 5 tables (`personnages`, `personnage_versions`, `sorts`, `invocations`, `formules`) avec Row Level Security configurable
- **Realtime** : souscription WebSocket sur `personnages` pour mise à jour temps réel de l'interface MJ
- **Versioning des données** : historique de snapshots JSON avec métadonnées (label, date, PI, Ka dominant)
- **254 entrées de contenu** : grimoire chargé dynamiquement avec cache mémoire (`grimCache`)

### CI/CD
- Push Git → déploiement Vercel automatique en ~30 secondes
- Configuration `vercel.json` pour le routing des sous-pages (`/mj`)
- Validation syntaxique Node.js (`--check`) systématique avant chaque commit

---

## Architecture

```
nephilim-forge/
├── index.html          # Application principale (SPA, ~360 Ko)
└── mj/
    └── index.html      # Interface Game Master
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
  "max_ep": 7,
  "max_ep_opt": 5,
  "incarnations": {
    "[epoque_id]": {
      "vecu": "string", "savoirs": [], "quete": "string",
      "arcane_id": "string", "arcane_deg": 0, "vecus_deg": {}
    }
  },
  "magie": { "bas": 0, "haut": 0, "grand": 0, "voie": "string" },
  "sephiroth": { "[id]": 0 },
  "mondes_integres": { "[monde_id]": true },
  "monde_tutelaire": "string",
  "alchimie": { "noir": 0, "blanc": 0, "rouge": 0 },
  "constructs": { "athanor": 0, "creuset": 0, "cornue": 0, "alambic": 0, "aludel": 0 },
  "effets_magiques": { "gravés": [], "focus": [] },
  "chutes": { "khaiba": 0, "narcose": 0, "ombre": 0 },
  "chutes_min": 0,
  "metier": "string",
  "ka_soleil_base": 3,
  "vecu_simulacre_extra": 0,
  "nom_simulacre": "string",
  "age_simulacre": 0
}
```

---

## Interface MJ (Game Master dashboard)

Page séparée (`/mj`) avec :
- Vue en temps réel de toutes les fiches joueurs via Supabase Realtime (WebSocket) et RLS configurable
- Affichage du pentacle SVG, attributs calculés, sciences occultes, quêtes/savoirs cumulés, grimoire
- Export TXT complet par joueur (incarnations détaillées, savoirs, grimoire, budget PI)
- Export PDF de fiche (Blob URL, prêt pour impression)
- Calcul autonome des scores sans dépendance aux données statiques côté MJ

---

## Points techniques notables

- **Offline-first** : fonctionne sans connexion (localStorage), sync cloud optionnelle
- **Zero-dependency** : aucun npm, aucun bundler, aucun framework — chargement instantané
- **Vérification d'accessibilité multi-éléments** : les formules alchimiques à double élément (`Terre/Eau`, `Feu/Lune`…) sont parsées et vérifiées élément par élément contre les scores Ka
- **Guards défensifs** : toutes les propriétés optionnelles des objets de données (`ep.sciences||[]`, `ep.quetes||[]`) sont sécurisées pour éviter les crashs silencieux sur accès `.forEach()` ou `.length`
- **Migration de données** : compatibilité ascendante dans `doLocalLoad()` — les anciennes fiches sont migrées automatiquement vers le nouveau schéma à la lecture
- **Sécurité** : clé `anon` Supabase publique par design, protégée par politiques RLS configurées côté base

---

## Prompt engineering

Ce projet a été développé en collaboration avec **Claude (Anthropic)** via un processus itératif de prompt engineering :

- Spécification fonctionnelle détaillée → génération de code ciblée par section
- Debug systématique : analyse statique Node.js `--check`, tests jsdom, reproduction de crash en environnement isolé
- Patches chirurgicaux identifiés par grep/analyse de position exacte dans le fichier source
- Validation syntaxique + tests de rendu automatisés avant chaque déploiement
- Gestion de la dette technique : résumés de session, journalisation des bugs résolus, comparaisons de hash MD5 pour vérifier la cohérence des fichiers livrés

L'ensemble représente plusieurs centaines d'itérations de prompts techniques sur plusieurs semaines, de la maquette initiale au produit en production.

---

## Remerciements

Un grand merci à la **communauté Nephilim** pour son engouement, ses retours terrain et son rôle de QA involontaire — les bugs les plus tordus n'auraient pas été trouvés sans des joueurs qui cliquent exactement là où il ne faut pas, exactement dans le bon ordre.

Merci aux auteurs et à l'éditeur **Mnémos** pour l'univers et les règles sur lesquels repose cette application. Une demande d'autorisation a été transmise à l'éditeur ; l'application est mise à disposition gratuitement pour la communauté.

---

## Auteur

Développé par **mabrizard** — passionné de JDR et de No-Code.
