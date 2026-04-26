# Générateur de modèles PLV/Pack — Prompt master

Outil pour générer rapidement de nouveaux modèles 3D à intégrer au simulateur Multipoles, en gardant la même DA et la même architecture technique que la PLV de sol existante.

---

## 1. Analyse esthétique de la PLV de référence

La PLV existante (capture d'écran de référence) repose sur quelques principes graphiques très lisibles :

**Direction artistique (DA)**

- Fond **blanc/clair** (#F8F9FA), sans gradient ni image
- Présentoir rendu en **blanc translucide laiteux** (`opacity: 0.42`, `roughness: 0.72`, `metalness: 0.02`, `clearcoat: 0.0`)
- **Arêtes wireframe** dessinées par-dessus, fines, gris-bleuté (`#c8d6e8` à `opacity: 0.6`) — donne l'aspect "rendu CAO net" plutôt que photo-réaliste
- **Produits** transparents légèrement teintés (cyan léger), arêtes plus marquées (`opacity: 0.78`)
- **Sol** clair, ombres douces (PCFSoft, opacity 0.3)
- **Lumière** : ambiante faible + key light directionnelle douce + 2 fills, intensité globale ~0.7
- Aucune texture, aucune image de marque — uniquement géométrie + arêtes
- Aucun élément UI sur la scène : seulement le présentoir en lévitation calme dans un environnement neutre

**Effet visuel** : maquette d'architecte / rendu blueprint épuré. Permet au client de visualiser **les volumes** et **l'agencement** sans se laisser distraire par des matières ou couleurs.

---

## 2. Architecture technique de la PLV de référence

**Stack imposée**

- Fichier HTML unique, sans bundler, sans dépendance npm
- Three.js **0.170.0** via importmap CDN jsdelivr
- Modules add-ons : `OrbitControls`, `RoundedBoxGeometry`, `RoomEnvironment`
- Polices Google : Orbitron + Rajdhani (préchargées)

**Renderer**

```js
WebGLRenderer { antialias: true }
pixelRatio: min(devicePixelRatio, 2)
toneMapping: ACESFilmicToneMapping, exposure 1.0
shadowMap: enabled, type PCFSoftShadowMap
```

**Caméra et contrôles**

```js
PerspectiveCamera(fov: 32, aspect: 1, near: 0.01, far: 100)
position: (3.2, 1.4, 3.8)
OrbitControls: enableDamping, dampingFactor 0.08
minDistance 0.3, maxDistance 10
maxPolarAngle: Math.PI/2 + 0.05
target: (0, 0.9, 0)
```

**Lumières (thème jour blanc)**

- `AmbientLight(0x14233a, 0.16)`
- key `DirectionalLight(0xffffff, 0.76)` à (5, 10, 6), `shadow.mapSize: 2048×2048`
- fill 1 `DirectionalLight(0xffffff, 0.18)` à (-4, 6, -3)
- fill 2 `DirectionalLight(0xffffff, 0.08)` à (-1, 4, -6)
- `HemisphereLight(0x0A0A2E, 0x000000, 0.1)`

**Sol et environnement**

- `PlaneGeometry(30, 30)` + `MeshPhysicalMaterial { color: 0xf4f7fc, roughness: 0.9 }`
- Plane séparé avec `ShadowMaterial { opacity: 0.3 }` pour ombres douces
- `PMREMGenerator.fromScene(new RoomEnvironment(), 0.04).texture` comme `scene.environment`

**Matériau PLV (jour blanc translucide)**

```js
MeshPhysicalMaterial {
  color: 0xc8d6e8,           // gris-bleuté très clair
  roughness: 0.72,
  metalness: 0.02,
  clearcoat: 0.0,
  clearcoatRoughness: 1.0,
  side: DoubleSide,
  envMapIntensity: 0.0,
  transparent: true,
  opacity: 0.42
}
```

**Arêtes wireframe**

```js
new LineSegments(
  new EdgesGeometry(mesh.geometry, 20),  // 20° threshold
  new LineBasicMaterial {
    color: 0xc8d6e8,
    transparent: true,
    opacity: 0.6,
    depthTest: true,
    depthWrite: false
  }
)
```

À ajouter en enfant de chaque mesh.

**Produits (pack sirop 50×50×100 mm)**

- `BoxGeometry(0.05 × 0.97, 0.10 × 0.98, 0.05 × 0.97)` (scale 1 mm = 0.001 unité, marge 2-3 % pour éviter z-fighting)
- `MeshPhysicalMaterial { color: 0xb0e8ec, roughness: 0.7, opacity: 0.4, transparent: true }`
- Edges en `0xffe5c1` à opacity 0.78 (cyan léger plus marqué)
- Plane label sur la face avant (`PlaneGeometry(pW × 0.7, pH × 0.22)` blanc opacity 0.05)

**Construction PLV procédurale**

```js
function buildPLV() {
  while (plvGroup.children.length) plvGroup.remove(plvGroup.children[0]);
  const W, D, H, t = wallT, bH = baseH, fH = frontonH, sT = shelfT, lip = shelfLip;
  const iW = W - 2*t, iD = D - t, sS = (H - bH - fH) / nbShelves;

  // 1. base
  const base = new Mesh(new BoxGeometry(W, bH, D), cyberMat());
  base.position.y = bH/2;
  addEdges(base);
  plvGroup.add(base);

  // 2. dos
  const back = new Mesh(new BoxGeometry(W, H, t), cyberMat());
  back.position.set(0, H/2, -D/2 + t/2);
  addEdges(back);
  plvGroup.add(back);

  // 3. côtés (boucle x2)
  for (const s of [-1, 1]) {
    const wall = new Mesh(new BoxGeometry(t, H, D), cyberMat());
    wall.position.set(s * (W/2 - t/2), H/2, 0);
    addEdges(wall);
    plvGroup.add(wall);
  }

  // 4. fronton (3 formes : rectangle, arche bezier, pointe)
  // Shape + ExtrudeGeometry

  // 5. étagères (boucle nbShelves)
  // BoxGeometry(iW, sT, iD), positionnée à bH + i*sS + sT/2
  // + lip optionnelle (BoxGeometry(iW, lip, sT) en bord avant)

  // 6. produits (grille facing × prodDepth × nbShelves)
}
```

**Recompute / debounce**

```js
let timer;
function rebuild() {
  clearTimeout(timer);
  timer = setTimeout(buildPLV, 25);
}
```

**Loop**

```js
function animate() {
  controls.update();
  renderer.render(scene, camera);
  requestAnimationFrame(animate);
}
```

---

## 3. UI minimale attendue (4 boutons)

Pas de header, pas de panneau de contrôle, pas de menu thèmes, pas de formulaire devis. Juste un dock fixé en bas-centre avec **4 boutons − / +** :

- Étagères : `−` `+` (1 à 8)
- Facing : `−` `+` (1 à 12)
- Profondeur : `−` `+` (1 à 6)

(Optionnel : un 4ᵉ groupe pour la hauteur PLV en cm, +/− 100 mm)

Style des boutons : pastilles arrondies (`border-radius: 9999px`), fond `rgba(255,255,255,0.85)`, bordure `rgba(0,0,0,0.08)`, taille 48×48, gap 8 px entre groupes. Police Rajdhani.

À chaque clic, mettre à jour `S.{param}`, appeler `rebuild()`.

---

## 4. Le prompt à copier-coller dans une IA

> Recopie ce bloc tel quel, puis joins **deux images** :
> - **Image 1** : photo (frontale et 3/4) du modèle de PLV ou de pack que tu veux modéliser.
> - **Image 2** : la capture d'écran de la PLV de référence Multipoles (DA cible).

```
ROLE : Tu es un développeur Three.js qui produit des fichiers HTML autonomes pour un simulateur PLV/packaging.

OBJECTIF : Génère un fichier HTML unique qui rend en 3D la PLV (ou le pack) visible sur l'IMAGE 1, avec exactement la même direction artistique que celle de l'IMAGE 2 (rendu blanc translucide, arêtes wireframe gris-bleuté, fond clair). Le rendu doit être paramétrable via 4 boutons +/- (étagères, facing, profondeur, hauteur).

LIVRABLE :
- Un seul fichier HTML, autonome, sans build step
- Utilise Three.js 0.170.0 via importmap CDN jsdelivr
- Modules add-ons requis : OrbitControls, RoomEnvironment, RoundedBoxGeometry
- Géométrie 100 % procédurale (pas de GLTF, pas d'asset externe)
- Aucune texture, aucune image (sauf polices Google Fonts Orbitron + Rajdhani)

DIRECTION ARTISTIQUE (à respecter strictement, voir IMAGE 2) :
- Fond #F8F9FA, sans gradient
- Matériau structure : MeshPhysicalMaterial { color: 0xc8d6e8, roughness: 0.72, metalness: 0.02, clearcoat: 0.0, side: DoubleSide, envMapIntensity: 0.0, transparent: true, opacity: 0.42 }
- Arêtes wireframe : EdgesGeometry(mesh.geometry, 20) + LineBasicMaterial { color: 0xc8d6e8, opacity: 0.6, transparent: true, depthWrite: false } enfantées dans chaque mesh
- Sol clair (#f4f7fc) + ombres PCFSoft douces (ShadowMaterial opacity 0.3)
- Lumières : AmbientLight 0x14233a 0.16 + DirectionalLight key 0xffffff 0.76 à (5,10,6) avec shadows + 2 fill DirectionalLight 0.18 et 0.08 + HemisphereLight 0x0A0A2E/0x000000 0.1
- PMREMGenerator + RoomEnvironment pour le sol/réflexions douces
- Renderer : ACESFilmicToneMapping, pixelRatio min(devicePR, 2), shadowMap PCFSoft

ARCHITECTURE PLV (analyse l'IMAGE 1 et adapte) :
- Décompose le présentoir en primitives BoxGeometry et ExtrudeGeometry (formes courbes via THREE.Shape + bezier)
- Une fonction unique build() qui clear le group puis reconstruit toute la géométrie à partir d'un objet d'état S = { nbShelves, facing, prodDepth, plvW, plvD, plvH, baseH, frontonH, wallT, ... }
- Tous les meshes castShadow = true, receiveShadow = true
- Chaque pièce a ses arêtes wireframe ajoutées via une fonction addEdges(mesh, color, op)
- Échelle : 1 mm = 0.001 unité Three.js. Les dimensions PLV typiques sont en mm.

PRODUITS À AFFICHER DANS LES ÉTAGÈRES :
- Pack sirop pharma, format fixe 50 × 50 × 100 mm (W × D × H)
- BoxGeometry avec marge -2 à -3 % pour éviter z-fighting
- Matériau translucide cyan-blanc : color 0xb0e8ec, opacity 0.4, roughness 0.7
- Arêtes en 0xffe5c1 opacity 0.78
- Étiquette : Plane sur la face avant, blanc opacity 0.05
- Disposition en grille : facing colonnes × prodDepth rangées sur chaque étagère

CAMÉRA ET CONTRÔLES :
- PerspectiveCamera(32, aspect, 0.01, 100), position (3.2, 1.4, 3.8)
- OrbitControls avec damping 0.08, minDistance 0.3, maxDistance 10, maxPolarAngle Math.PI/2 + 0.05, target (0, 0.9, 0)
- Après le build, recadre la caméra sur la bounding box du présentoir si la hauteur a changé

UI MINIMALE (TRÈS IMPORTANT — ne pas ajouter d'autre UI) :
- Dock fixe en bas-centre de l'écran
- 3 ou 4 groupes de boutons − / + : Étagères, Facing, Profondeur (et optionnellement Hauteur)
- Pastilles arrondies blanches semi-opaques, taille 48px, gap 8px, police Rajdhani
- Affichage de la valeur courante entre chaque paire − +
- Aucun header, aucun panneau de contrôle, aucun menu thème, aucun formulaire devis, aucun toast
- Le canvas occupe 100 % de l'écran derrière le dock

DEBOUNCE :
- Chaque clic sur +/- met à jour l'état S puis appelle rebuild() qui clearTimeout + setTimeout(build, 25)

CONTRAINTES :
- Le résultat doit s'ouvrir en double-click sur n'importe quel navigateur récent et fonctionner immédiatement
- Le code doit être lisible, organisé en sections commentées (// === SETUP === / // === BUILD === / // === UI === / // === LOOP ===)
- Pas de console.log, pas de debug
- Tout en un seul <script type="module"> à la fin du body
- Importmap dans le head

DÉLIVRE UNIQUEMENT LE FICHIER HTML COMPLET, sans explication ni commentaire en dehors du code.
```

---

## 5. Variantes du prompt selon le sujet

### 5a. Pour un pack/étui (au lieu d'une PLV)

Remplace dans le prompt :
- "PLV" par "pack/étui"
- Section ARCHITECTURE PLV par : "Décompose l'étui en 6 panneaux (face, dos, 2 côtés, fond, dessus) + rabats (poussière, patte de collage, languette de fermeture)"
- Supprime la section PRODUITS
- UI : remplace par 3 boutons + (largeur, profondeur, hauteur) en mm, range 30–300 mm, step 5 mm

### 5b. Pour un présentoir comptoir (petit format)

Idem PLV, mais :
- baseH plus faible (50–100 mm)
- pas de fronton ou fronton minimaliste
- 1–3 étagères max
- camera position plus rapprochée : (1.6, 0.7, 1.9), target (0, 0.4, 0)

### 5c. Pour une tête de gondole

Idem PLV, mais :
- Pas de base haute (juste un socle)
- Ajout d'un bandeau publicitaire en haut (Plane vertical avec arêtes)
- 4–6 étagères standard, dimensions L 1000 × P 400 × H 1800 mm

---

## 6. Workflow d'utilisation

1. Tu prends une **photo nette** du modèle de PLV ou pack à reproduire (frontale + 3/4 idéalement).
2. Tu copies le **prompt master** ci-dessus.
3. Tu joins **2 images** dans ton chat IA : la photo du modèle (image 1) + la capture de la DA Multipoles (image 2).
4. L'IA renvoie un fichier HTML.
5. Tu enregistres le HTML dans `variations/{ia}-{version}-{nom-modele}/index.html`.
6. Tu pousses sur GitHub, GitHub Pages déploie automatiquement.
7. Preview live : `https://jeyiop.github.io/plv-template-sandbox/variations/{dossier}/`.

---

## 7. Points à valider après génération

- [ ] Le fichier s'ouvre en double-click et affiche la PLV
- [ ] Les 3-4 boutons +/- modifient bien la géométrie
- [ ] Pas de header, pas de panneau de contrôle, pas de devis
- [ ] Fond blanc, arêtes gris-bleuté, structure translucide
- [ ] Ombres douces sous le présentoir
- [ ] Camera tourne avec OrbitControls
- [ ] Les produits sont bien des packs 5×5×10 cm
- [ ] La PLV correspond au modèle de l'image 1 en silhouette générale

Si l'un de ces points manque, tu peux relancer avec un message court : *"Tu as oublié X, corrige."*
