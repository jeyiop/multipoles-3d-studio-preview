# Veille technique — Bibliothèque de modèles 3D pour le simulateur PLV/packaging

Contexte : Multipoles (PLV/packaging pharma-cosmétique), simulateur Three.js single-file, géométrie procédurale. Cible : étendre vers une bibliothèque de modèles, maintenance par dev généraliste, tout dans le navigateur.

---

## Angle 1 — Catalogues de modèles 3D existants

### Places de marché généralistes

- **[Sketchfab](https://sketchfab.com/search?q=pos+display&type=models)** : export GLB/GLTF natif (idéal Three.js), licences CC-BY / Standard / Editorial claires, filtres "Downloadable + Commercial use". Qualité très inégale sur `pos display`, `retail shelf`, `product stand` — displays génériques, peu de PLV pharma. Meilleure source pour prototyper grâce au GLB direct.
- **[CGTrader](https://www.cgtrader.com/3d-models?keywords=retail+display)** : catalogue pro plus fourni (mobilier retail, gondoles, têtes de gondole, displays carton). Formats dominants : FBX/OBJ/MAX, GLB plus rare. Licences "Royalty Free" claires. 20–80 € le modèle propre, topology lourde (à décimer).
- **[TurboSquid](https://www.turbosquid.com/Search/3D-Models/retail-display)** : segment premium, FBX/MAX, peu de GLB. Programme **CheckMate** garantit une topologie propre — utile mais rarement GLB direct.
- **[Free3D](https://free3d.com/)** : dépannage, qualité basse.

### Bibliothèques spécialisées retail / mobilier

- **[BIMobject](https://www.bimobject.com/)** et **[Polantis](https://www.polantis.com/)** : catalogues BIM orientés architecture. Formats dominants : **Revit, IFC, DWG, SketchUp** — pas GLB natif. Conversion nécessaire via Blender. Intéressant pour l'agencement magasin (gondoles, caisses), peu pour la PLV carton.
- **[3D Warehouse (SketchUp)](https://3dwarehouse.sketchup.com/)** : énorme base de displays retail, licences floues (usage interne OK, redistribution grise). Export via SketchUp Free → GLB possible.

### Fabricants de systèmes PLV

- **[HL Display](https://www.hl-display.com/)**, **[Tegometall](https://www.tegometall.com/)**, **[Storflex](https://www.storflex.de/)** : fiches CAO 2D/3D fournies en B2B sur demande, formats DWG/STEP, rien de web-ready.
- **Tetra Pak, Quaker** : pas de bibliothèque 3D publique pour usage tiers.

**Verdict** : aucune bibliothèque pro "prête à intégrer en GLB". Sketchfab pour le prototypage, CGTrader pour l'achat ponctuel, rien de satisfaisant pour constituer un catalogue cohérent. **Construire son propre kit est incontournable.**

---

## Angle 2 — Outils de modélisation paramétrique PLV et packaging

### Suites pro packaging (dieline → 3D)

- **[Esko ArtiosCAD](https://www.esko.com/en/products/artioscad)** + **[Esko Studio](https://www.esko.com/en/products/studio)** : référence mondiale packaging carton. Paramétrique via bibliothèques ECMA/FEFCO. Export **ARD** (2D structurel), **3DS / Collada**, plus récemment **glTF** via Studio Visualizer. Licences perpétuelles ~8–15 k€ + maintenance ~20 %/an. **Hors budget PME.**
- **[Engview Packaging Suite](https://engview.com/)** : concurrent direct, pack-design paramétrique, export 3DS/OBJ/Collada. Plus accessible (~3–5 k€ licence initiale) mais lourd pour usage ponctuel.
- **[PackZ (Hybrid Software)](https://www.hybridsoftware.com/products/packz/)** : prépresse, pas un outil 3D à proprement parler.
- **[Kasemake (AG/CAD)](https://www.kasemake.com/)** : alternative UK à Esko/Engview, similaire en gamme.

### Approches CAO paramétrique accessibles

- **[Rhino 3D](https://www.rhino3d.com/)** + **[Grasshopper](https://www.grasshopper3d.com/)** : 995 € licence perpétuelle. Grasshopper permet de coder des présentoirs entièrement paramétriques (nb étagères, angles, découpes). Export via plugin **[Rhino.Inside](https://www.rhino3d.com/inside/)** ou plugins [Food4Rhino](https://www.food4rhino.com/) vers OBJ/FBX, glTF via Blender intermédiaire. Tutos retail/expo chez **[ParametricHouse](https://parametrichouse.com/)**.
- **[Blender](https://www.blender.org/) 4.x + Geometry Nodes** : **la solution la plus pertinente ici**. Gratuit, open-source, Geometry Nodes couvrent le paramétrique (inputs numériques exposés sur le modifier), export **glTF 2.0** natif avec [extensions KHR](https://github.com/KhronosGroup/glTF/tree/main/extensions). Tutos clés : [Erindale Woodford](https://www.youtube.com/@Erindale), [Default Cube](https://www.youtube.com/@DefaultCube), doc officielle [Blender Geometry Nodes](https://docs.blender.org/manual/en/latest/modeling/geometry_nodes/index.html).
- **[FreeCAD](https://www.freecad.org/)** + TechDraw : 100 % libre, paramétrique stricte, export STEP/OBJ. Plus adapté mécanique que PLV carton.
- **[OpenSCAD](https://openscad.org/)** : code-only, excellent pour générer des variantes par script, export STL/OBJ/AMF. Un peu rude pour le design.

**Verdict** : Esko/Engview hors-budget et hors-web. **Blender + Geometry Nodes** offre le meilleur ratio : gratuit, export glTF propre, paramètres exposables, pipeline compatible Three.js sans intermédiaire.

---

## Angle 3 — Workflow bibliothèque maison

### Standard glTF et variantes

- **[glTF 2.0 spec](https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html)** : format natif three.js via `GLTFLoader`.
- **[KHR_materials_variants](https://github.com/KhronosGroup/glTF/tree/main/extensions/2.0/Khronos/KHR_materials_variants)** : changement de matériau runtime (coloris présentoir) sans dupliquer la mesh. Supporté par three.js via `GLTFLoader` + `parser.getDependency`.
- **[KHR_animation_pointer](https://github.com/KhronosGroup/glTF/tree/main/extensions/2.0/Khronos/KHR_animation_pointer)** : expose des paramètres animables (hauteur étagère, écartement). Plus récent, support three.js via add-ons — **attendre stabilisation**.
- **Morph targets** et **node hierarchies** : variations géométriques simples supportées natif. Pour du vrai paramétrique, privilégier "kit of parts" ci-dessous.

### Pipeline d'assets

- **[gltf-transform](https://gltf-transform.dev/)** (Don McCurdy) : couteau suisse CLI/JS — compression Draco/Meshopt, dedup, resize textures, validate. **Indispensable** pour maintenir une biblio propre.
- **[three-mesh-bvh](https://github.com/gkjohnson/three-mesh-bvh)** : accélère raycasting pour drag & drop / snapping entre modules.
- **[gltfpack](https://github.com/zeux/meshoptimizer)** (Meshopt) : compression GLB agressive, utile bande passante.
- **[@pmndrs/gltfjsx](https://github.com/pmndrs/gltfjsx)** : génère composants JSX depuis un GLB, utile même hors React pour explorer la hiérarchie.

### Configurateurs web de référence

- **[IKEA Kreativ](https://www.ikea.com/fr/fr/home-design/)** : scan room + placement modules, approche kit-of-parts massive.
- **[Kartell](https://www.kartell.com/)**, **[Vitra Configurator](https://www.vitra.com/configure)**, **[Muuto](https://www.muuto.com/)** : variantes via glTF + KHR_materials_variants.
- **[Shapespark](https://www.shapespark.com/)**, **[Threekit](https://www.threekit.com/)** : SaaS configurateur produit, référence concurrentielle, coûteux mais inspirants UX.
- **Exemples R3F** : [drei `<Gltf>`](https://github.com/pmndrs/drei), démos kitbash sur [R3F examples](https://r3f.docs.pmnd.rs/getting-started/examples). Hors bundler : three.js importmaps suffit, pas besoin de React.

### Approche "kit of parts"

Architecture la plus adaptée : chaque présentoir = composition de modules GLB (base, colonne, étagère, fronton, accessoire) instanciés et snappés. Avantages : **un module modifié se propage**, ajout de nouveaux presentoirs par simple JSON de composition, compatible avec la géométrie procédurale existante. Référence : [Kitbash3D](https://kitbash3d.com/).

---

## Recommandation synthèse

Chemin concret pour Multipoles, optimisé dev généraliste / budget PME / pas de plugin :

1. **Abandonner l'idée d'un catalogue externe "clé en main"** : rien n'existe en GLB retail pharma. Investir dans une bibliothèque maison est payant à 6 mois.
2. **Pipeline de production** : [Blender 4.x](https://www.blender.org/) + Geometry Nodes pour chaque famille de présentoir, un fichier `.blend` par famille avec paramètres exposés, export GLB (glTF 2.0 binaire, Draco ON, textures WebP). Coût : 0 €, courbe 2–3 semaines pour un dev généraliste via tutos [Blender Guru](https://www.youtube.com/@blenderguru) et [CGCookie](https://cgcookie.com/).
3. **Architecture runtime** : garder la géométrie procédurale pour le présentoir de sol actuel (rapide, éditable en JS pur), passer en **kit-of-parts GLB** pour les nouveaux modèles. Manifeste JSON type `{ "family": "totem", "modules": [{"id":"base_A","pos":[0,0,0]}, ...] }`.
4. **Variantes** : [KHR_materials_variants](https://github.com/KhronosGroup/glTF/tree/main/extensions/2.0/Khronos/KHR_materials_variants) pour coloris/finitions, morphs ou modules interchangeables pour variations géométriques. Éviter KHR_animation_pointer tant que le support three.js n'est pas stable.
5. **Pipeline d'assets** : [gltf-transform](https://gltf-transform.dev/) en script npm (batch optimize, validate, dedup) avant commit. [three-mesh-bvh](https://github.com/gkjohnson/three-mesh-bvh) dès qu'il faudra snapping/picking avancé.
6. **Intégration sans bundler** : three.js via [importmap](https://threejs.org/docs/#manual/en/introduction/Installation) CDN, `GLTFLoader` + `DRACOLoader` en modules ES natifs — compatible single-file HTML actuel.
7. **Achats ponctuels** : [Sketchfab](https://sketchfab.com/) (filtre downloadable + CC/Standard) pour accessoires (bouteilles, packs pharma, produits de remplissage), jamais pour le présentoir principal — contrôle qualité difficile.
8. **À éviter** : Esko/Engview (hors budget, workflow desktop lourd), BIMobject/Polantis (mauvais formats), TurboSquid premium (FBX à reconvertir).

**Budget total estimé** : 0–500 €/an (quelques achats Sketchfab ponctuels) vs 10 k€+/an pour Esko.

---

## Familles de PLV à modéliser en priorité

Pour démarrer la bibliothèque maison, couvrir ces familles typiques pharma/cosmétique :

- **Présentoir de sol** (actuel, procédural) — garder tel quel.
- **Présentoir de comptoir** (petit, 20–40 cm) : totem, tiroir présentoir, box ouvert.
- **Tête de gondole** (bandeau haut + modules étagère standardisés).
- **Îlot / podium** (base surélevée + modules verticaux empilables).
- **PLV suspendue / wobbler / stop-rayon** (accessoires 2D/3D légers).
- **Vitrine réfrigérée / présentoir cosmétique éclairé** (avec LEDs).
- **Packaging étui** (existant procédural) — garder + ajouter : étui avec fenêtre, sleeve, coffret rigide, blister.

Chaque famille → un `.blend` paramétrique → un GLB exporté + un fichier `manifest.json` décrivant les paramètres exposés (dimensions min/max, couleurs, finitions, modules optionnels).

---

## Prochaines étapes concrètes

1. Installer [Blender 4.x](https://www.blender.org/) + addon **glTF 2.0 export** (natif depuis 3.0).
2. Prototyper un présentoir comptoir simple en Geometry Nodes (2–3 jours).
3. Exporter en GLB, ajouter un `GLTFLoader` au simulateur actuel, tester le chargement.
4. Définir le format du `manifest.json` (paramètres exposés, variantes matériau).
5. Créer une UI de sélection de famille dans le simulateur (toggle PLV sol / PLV comptoir / etc.).
6. Itérer : une famille par 2 semaines.
