# Multi-Pôles — Simulateur 3D PLV & Packaging

Repo dédié au **simulateur 3D** de Multi-Pôles, accessible publiquement sur https://multi-poles.cloud/simulateur (intégré en iframe dans le site vitrine Next.js).

---

## Le contexte global

**Multi-Pôles** est un G.I.E. fondé en 1995, spécialisé dans la conception, la fabrication et la livraison de **PLV** (Publicité sur Lieu de Vente) et de **packaging** pour les marques cosmétique et pharmaceutique. Adresse : 53 rue des Deux Communes, 93100 Montreuil.

Le site `multi-poles.cloud` joue trois rôles :
1. **Vitrine** commerciale (présentation savoir-faire, réalisations, équipe, contact)
2. **Outil de devis 3D** : un simulateur web qui permet à un prospect de configurer son présentoir ou son packaging en temps réel et de demander un devis sans passer par un commercial
3. **Backend** pour les formulaires devis, le routing email, l'IA Inspire-moi (suggestions de configuration)

L'objectif business du simulateur : **diminuer le frottement commercial** — un client peut visualiser son projet en 3D depuis son tél, jouer avec les paramètres (dimensions, finitions, matériaux, fronton) et envoyer son brief, sans aller-retour de mockup avec un graphiste.

---

## Architecture en 3 compartiments

Le projet est volontairement compartimenté pour qu'on puisse **modifier chaque partie sans casser les autres**.

| Compartiment | Repo GitHub | Tech | Hébergement |
|---|---|---|---|
| **Site vitrine** | [`multipoles-site`](https://github.com/jeyiop/multipoles-site) | Next.js 15 + Tailwind v4 + Framer Motion | Docker → VPS Hostinger (`72.60.45.230`) |
| **Simulateur 3D** | [`multipoles-3d-studio-sandbox`](https://github.com/jeyiop/multipoles-3d-studio-sandbox) (privé) + [`multipoles-3d-studio-preview`](https://github.com/jeyiop/multipoles-3d-studio-preview) (public, GitHub Pages) | HTML monofichier + Three.js 0.170 (CDN) | Servi en iframe par le site vitrine, fichier `studio-3d.html` |
| **Backend devis & IA** | Intégré dans `multipoles-site` (routes API Next.js : `/api/studio/ai-propose`, formulaire devis) | Next.js API Routes + SDK Anthropic | Même container Docker que le site |

### Pourquoi compartimenter ?

- **Le simulateur HTML monofichier** peut être édité sans toucher Next.js (pas de `npm run build` requis)
- **Le site vitrine** peut être restylé sans toucher au moteur 3D
- **Le backend devis** peut évoluer (nouveaux champs formulaire, intégration CRM…) sans impacter le rendu 3D
- Chaque compartiment a son propre cycle de release

---

## Repos GitHub — état au 28 avril 2026

### Repos actifs (à jour)

- **[multipoles-site](https://github.com/jeyiop/multipoles-site)** — site vitrine Next.js. Source canonique du site web.
- **[multipoles-3d-studio-sandbox](https://github.com/jeyiop/multipoles-3d-studio-sandbox)** — repo privé de dev pour le simulateur. C'est ici qu'on bosse.
- **[multipoles-3d-studio-preview](https://github.com/jeyiop/multipoles-3d-studio-preview)** — clone public du sandbox + GitHub Pages activé pour preview live sur tél (https://jeyiop.github.io/multipoles-3d-studio-preview/).
- **[plv-template-sandbox](https://github.com/jeyiop/plv-template-sandbox)** — sandbox de variations IA (essais générés par Claude / GPT / Gemini / Perplexity / Grok / Meshy / Tripo / Luma).

### Repos archivés (read-only, pour archive)

- `SITE-MP-LEGACY`, `SITE-MP-BLANC`, `SITE-MP-BLEU`, `sitempv3`, `multipoles-site-master`, `site-MP-V2` — anciennes versions du site, conservées en historique mais plus utilisées.

---

## Le simulateur 3D — comment ça marche

### Stack technique

- **Fichier unique** `index.html` (~120 Ko, ~2700 lignes), pas de build, pas de bundler
- **Three.js 0.170** chargé via importmap CDN jsdelivr
- **Géométrie 100 % procédurale** : aucun fichier 3D externe (.glb / .gltf / .obj). Tout est généré au runtime à partir de primitives BoxGeometry + ExtrudeGeometry.
- **Polices** Google Fonts : Orbitron + Rajdhani

### Modèles disponibles

Le simulateur supporte plusieurs **silhouettes de PLV** sélectionnables via le bouton "⊞ STANDARD / ⊟ PALLET" (top-left mobile) :

1. **Standard** — PLV de sol classique : base + dos + côtés + fronton (3 formes : droit / arrondi / pointe) + étagères
2. **Pallet** — PLV sur palette EUR + base box fermée + parois latérales en trapèze + fronton rectangulaire à coins arrondis

Tous les modèles partagent la **même architecture paramétrique** : les dimensions de la PLV sont dérivées des produits (facing × `prodW`, prodDepth × `prodD`, nbShelves × `shelfSpacing`).

### Paramètres ajustables

Par sliders sur desktop (panneaux gauche + droit) ou par boutons +/− sur mobile (dock du bas) :

- **Étagères** : nombre (1–8)
- **Facing** : nombre de produits par rangée latérale (1–12)
- **Profondeur** : nombre de produits en profondeur par étagère (1–12)
- **Hauteur socle** (slider desktop)
- **Rebord étagère** (slider desktop)
- **Fronton** : forme + hauteur (slider desktop)
- **Dimensions produit** (largeur, profondeur, hauteur) (slider desktop)

### Modes de visualisation

- **PLEIN / VIDE** (bouton flottant top-right) : affiche ou masque les produits sur les étagères pour voir la structure pure
- **Thèmes visuels** (desktop seulement) : Light Navy, Full White, Navy Glass, Warm Pro, Studio Neutre — change l'ambiance du rendu

### Mobile vs Desktop

- **Desktop** : UI complète (panneaux paramètres, formulaire devis, IA Inspire-moi, sélecteur de thème)
- **Mobile** (< 768 px) : UI minimale — juste la PLV plein écran + dock 3 boutons (étagères/facing/profondeur) + 2 boutons flottants (modèle, plein/vide). Toute l'UI desktop est masquée pour ne pas saturer l'écran.

### Caméra responsive

La caméra s'adapte automatiquement à l'orientation (portrait / paysage) et à la taille de l'écran. En mobile portrait, la PLV est recadrée pour tenir dans le viewport sans se faire couper. Le pinch-zoom préserve la position de la caméra (l'app ne se "recentre" plus quand iOS Safari fait apparaître/disparaître la barre d'adresse).

---

## Architecture du simulateur (pour les futurs devs)

### Fichier `index.html`

Structure :

```
<head>
  <importmap> — three.js + addons depuis jsdelivr
  <style>     — CSS desktop + @media (max-width: 768px) overrides mobile
</head>
<body data-ui-theme="day" data-visual-preset="light_navy">
  <div class="app">
    <header>...</header>
    <div class="main">
      <div class="controls-left">...sliders structure...</div>
      <div class="viewport"><canvas id="canvas3d"/></div>
      <div class="controls-right">...formulaire devis + IA...</div>
    </div>
  </div>
  <script type="module">
    // 1. State : S (PLV), P (Pack), V (Visual theme)
    // 2. Three.js setup : renderer, scene, camera, lights, ground, OrbitControls
    // 3. Build functions :
    //    - buildPLVStandard()  — modèle standard
    //    - buildPLVPallet()    — modèle pallet
    //    - buildPLV() (wrapper) — dispatch selon S.plvModel
    //    - buildPackaging()    — mode pack carton
    // 4. UI bindings : sliders → state → rebuild
    // 5. Mobile patch (IIFE) :
    //    - fitCameraToPLV()   — recadre auto sur la bbox de plvGroup
    //    - resize handler     — uniquement update aspect (pas reset caméra)
    //    - mobile dock + boutons flottants modèle/vide
  </script>
</body>
```

### Comment ajouter un nouveau modèle PLV

1. Créer une nouvelle fonction `buildPLV<NomModele>()` qui suit la même architecture que `buildPLVPallet` :
   - Utilise `computeDims()` pour récupérer plvW/plvD/plvH/innerW/shelfSpacing
   - Génère la géométrie avec primitives + arêtes wireframe (cyberMat + addEdges)
   - Place les produits avec la même grille que le standard
   - Respecte `S.showProducts` pour le mode vide
2. Ajouter le modèle à la list dans `S.plvModel = 'standard' | 'pallet' | '<NomModele>'`
3. Mettre à jour le wrapper `buildPLV()` pour dispatcher
4. Ajouter le cycle dans le bouton flottant `m-model`

Voir `variations/pallet-v1/index.html` pour un exemple complet en standalone.

### Référence : prompt master pour générer de nouveaux modèles via IA

Le fichier `PROMPT_GENERATEUR_PLV.md` contient un prompt complet copiable-collable dans n'importe quelle IA (Claude, ChatGPT, Gemini, Grok, Perplexity) qui produit un nouveau fichier HTML standalone respectant la même DA et la même architecture. Il suffit de joindre une photo du modèle réel à reproduire.

---

## Déploiement en production

### Site vitrine (Next.js)

Hébergé sur VPS Hostinger `72.60.45.230` via Docker.

Pipeline :
```bash
# Sur le Mac
cd ~/Dropbox/7_APP_SANDOX/BOXSITEMP_OPTIM
npm run build
# Copier les artifacts vers le VPS
scp -r -P 22 -i ~/.ssh/id_ed25519_vps .next/standalone/.next/* root@72.60.45.230:/opt/site-new/.next/
scp -r -P 22 -i ~/.ssh/id_ed25519_vps public/* root@72.60.45.230:/opt/site-new/public/
# Reconstruire l'image Docker et redéployer
ssh -i ~/.ssh/id_ed25519_vps root@72.60.45.230 "cd /opt && docker compose build multipoles-site && docker compose up -d multipoles-site"
```

### Simulateur (HTML standalone)

Le `studio-3d.html` est servi par le container `multipoles-site` (chemin interne `/app/public/studio-3d.html`). Pour le mettre à jour seul, il faut **rebuild l'image Docker** car il n'y a pas de mount volume sur le `public/` :

```bash
# Mettre à jour le fichier dans le projet Next.js
cp ~/Dropbox/7_APP_SANDOX/SIMULATEUR_STANDALONE/index.html \
   ~/Dropbox/7_APP_SANDOX/BOXSITEMP_OPTIM/public/studio-3d.html

# Commit et push
cd ~/Dropbox/7_APP_SANDOX/BOXSITEMP_OPTIM
git add public/studio-3d.html
git commit -m "Simulateur : <description>"
git push multipoles main

# Rebuild + redeploy sur Hostinger
# (cf. pipeline ci-dessus)
```

**Note** : pour itérer rapidement sur le simulateur sans toucher au déploiement prod, on utilise `multipoles-3d-studio-preview` (GitHub Pages) qui sert le même fichier en autonome. Dès qu'on valide une version, on la propage dans `BOXSITEMP_OPTIM/public/studio-3d.html` puis on rebuild/redeploy.

---

## Roadmap simulateur

### À court terme (priorisé)

- [ ] Ajouter d'autres modèles PLV (totem, tête de gondole, présentoir comptoir, vitrine éclairée)
- [ ] Mode "vue technique" avec cotes en mm directement sur la 3D
- [ ] Export PDF de la fiche produit (config + dieline + brief client)
- [ ] Export image PNG de la PLV en haute résolution

### À moyen terme

- [ ] Charger des **modèles produits clients réels** via `GLTFLoader` (flacons, étuis spécifiques) — pour qu'un client cosmétique voie son flacon dans son présentoir
- [ ] Bibliothèque **kit-of-parts GLB** modulaire (base, colonne, étagère, fronton, accessoire) pour combiner librement
- [ ] Pipeline `gltf-transform` pour optimiser les assets

### À long terme

- [ ] Mode AR (Apple Quick Look / WebXR) pour visualiser la PLV à l'échelle dans le magasin du client
- [ ] Configurateur de marque : couleurs / logos / textures uploadées par le client appliquées sur la 3D

Voir `VEILLE_BIBLIOTHEQUE_MODELES.md` pour la veille technique complète sur les outils et workflows envisagés.

---

## Fichiers de référence

- `index.html` — le simulateur canonique (= source de vérité, copie de la prod après chaque sync)
- `PROMPT_GENERATEUR_PLV.md` — prompt master à coller dans une IA pour générer un nouveau modèle PLV
- `VEILLE_BIBLIOTHEQUE_MODELES.md` — veille techno sur les bibliothèques de modèles 3D, outils paramétriques, workflows
- `BLUEPRINT_SITE_MULTIPOLES.md` — analyse complète du site vitrine multi-poles.cloud (pages, sections, SEO, bugs détectés)
- `variations/pallet-v1/index.html` — fork standalone du modèle Pallet (référence pour comprendre comment dériver une variation)

---

## Contact

Jeremy Poirier — jeremy@multi-poles.net — gérant Multi-Pôles
