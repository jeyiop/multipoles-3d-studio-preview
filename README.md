# Multipoles 3D Studio — Sandbox

Version standalone du simulateur PLV & Packaging, isolée du site principal pour itérer librement.

## Source

Extrait le 19 avril 2026 depuis `BOXSITEMP_OPTIM/public/studio-3d.html` (version du 1er avril 2026).
Le simulateur utilisé en production sur le site est intégré via iframe à cette même version.

## Stack

- Fichier HTML unique auto-portant (`index.html`, ~118 Ko)
- Three.js 0.170.0 chargé via CDN jsdelivr (importmap)
- Aucun build, aucune dépendance npm, aucun bundler
- Polices Google : Orbitron + Rajdhani

## Architecture 3D

Tout est **procédural** : aucun fichier 3D externe n'est chargé (`.glb`, `.gltf`, `.obj`, `.fbx`, `.stl`).

### Mode PLV — `buildPLV()`
Géométrie reconstruite à partir des paramètres utilisateur :
- Base, dos, côtés → `BoxGeometry`
- Fronton (3 formes : rectangle, vague bombée, pointe) → `ExtrudeGeometry` à partir d'une `Shape` bezier
- Étagères + rebords (lips) → `BoxGeometry`
- Grille produit (facing × profondeur) → `BoxGeometry` + `PlaneGeometry` pour l'étiquette
- Contours nets → `EdgesGeometry` + `LineBasicMaterial`

Paramètres : `prodW`, `prodD`, `prodH`, `facing`, `prodDepth`, `nbShelves`, `wallT`, `baseH`, `frontonH`, `frontonShape`, `shelfLip`, `headroom`.

### Mode Packaging — `buildPackaging()`
- Boîte 6 panneaux + rabats poussière + patte de collage → `BoxGeometry` + `ExtrudeGeometry`
- Dieline 2D (patron à plat) tracée via Canvas 2D avec labels COLLE/COTE/FACE/COTE/DOS et cotes en mm

Paramètres : `boxW`, `boxD`, `boxH`, `construction`, `printType` (offset/num), `finish` (mat/brillant), `paper`, `colors`, `luxe`.

## Lancer en local

```bash
# Option 1 : ouvrir directement le fichier
open index.html

# Option 2 : serveur local (recommandé pour éviter les restrictions CORS éventuelles)
python3 -m http.server 8000
# puis http://localhost:8000
```

## Déploiement

- GitHub Pages : push sur `main`, activer Pages sur la branche `main` racine, URL dispo immédiatement.
- Cloudflare Pages / Netlify / Vercel : déployer le dossier tel quel, aucun build step.

## À faire (idées backlog)

- [ ] Ajouter `GLTFLoader` + UI d'upload pour charger des modèles produits clients (flacons, bouteilles)
- [ ] Export GLTF / STL / OBJ de la configuration courante
- [ ] Export PDF de la dieline
- [ ] Persistance locale des configurations (localStorage)
- [ ] Mode mobile (UI actuelle pensée desktop)

## Upload de fichiers

L'input file accepte uniquement les formats brief (`.pdf`, `.jpg`, `.png`, `.ai`, `.eps`, `.doc`, `.docx`) pour pièces jointes au devis, pas pour charger de la 3D.
