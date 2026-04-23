# Blueprint — www.multi-poles.cloud

Relevé effectué le 23 avril 2026 depuis les pages publiques.

---

## Stack technique

- **Framework** : Next.js (confirmé par headers `x-nextjs-cache`, `x-nextjs-prerender`, `X-Powered-By: Next.js`)
- **Serveur** : nginx 1.24 sur Ubuntu (reverse proxy)
- **Rendu** : SSG/ISR (sitemap avec `changefreq`, cache `s-maxage=31536000`)
- **Images** : next/image optimisé (multi-breakpoints + WebP)
- **Fonts** : deux woff2 préchargés (pas de Google Fonts en CDN au runtime)
- **Langue** : `<html lang="fr">`
- **Headers sécurité** : `X-Frame-Options: SAMEORIGIN`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: origin-when-cross-origin`

---

## Arborescence (sitemap officiel)

```
/                       priority 1.0   daily
├── /solutions          priority 0.8   weekly
│   ├── #plv            (anchor, pas de page dédiée)
│   ├── #packaging      (anchor)
│   └── #print          (anchor)
├── /realisations       priority 0.8   weekly
├── /apropos            priority 0.8   weekly
├── /equipe             priority 0.8   weekly   ⚠️ déclarée mais pas dans la nav
├── /blog               priority 0.8   daily    ⚠️ déclarée mais pas dans la nav
├── /contact            priority 0.9   weekly
├── /devis              priority 0.8   weekly
├── /simulateur         priority 0.8   weekly   (iframe vers studio-3d.html)
├── /mentions-legales
├── /politique-confidentialite
└── /cookies
```

---

## Page d'accueil — 11 sections

1. **Hero** — H1 "MULTIPÔLES", intro G.I.E. depuis 1995, thématiques étuis/PLV/packaging. CTA "Découvrir nos solutions".
2. **Packaging éco & premium** — cible cosmétique haut de gamme (pelliculage, gaufrage, dorure). Valeurs : naturalité, élégance, responsabilité.
3. **Packaging pharma & médical** — étuis, conformité réglementaire. Valeurs : rigueur, conformité, précision.
4. **Devis 3D Studio** — composant interactif, CTA "Ouvrir Devis 3D Studio" → `/simulateur`.
5. **PLV de comptoir** — retail cosmétique/pharmacie, conception + impression + façonnage + assemblage. Valeurs : visibilité, séduction, performance.
6. **Catalogue structures carton** — 30 structures (étuis, PLV sol, coffrets). CTA vers simulateur.
7. **Excellence pharmacie** — brief → prototype → production → logistique. CTA "Nous contacter".
8. **Vitrine réalisations** — galerie 16 images, lien vers `/realisations`.
9. **Vidéo** — play button sur logo, "réalisations en contexte point de vente".
10. **FAQ** — 4 Q/R (expertise, sur-mesure, innovation 3D, éco-responsabilité).
11. **CTA final** — "Demander un devis" + "Nous contacter".

---

## Page `/solutions`

- **H1** : "Solutions PLV & Packaging"
- **4 blocs H2** : PLV sur mesure, Packaging premium, Impression et finition, Devis 3D Studio
- **CTA final** : "Prêt à concrétiser votre projet ?" avec double CTA devis/contact
- **Anchors internes** : `#plv`, `#packaging`, `#print`

---

## Page `/realisations`

- Accroche : "Explorez nos réalisations — dispositifs PLV impactants pour marques cosmétique et pharma"
- **Filtre** : bouton "Tous" (semble être le seul filtre visible)
- **CTAs** : "Demander un devis", "Essayer notre simulateur"
- ⚠️ Pas de liste projets/clients exposée publiquement au scrape — contenu potentiellement chargé dynamiquement ou page minimale

---

## Page `/apropos`

- **H1** : "À propos de Multi-Pôles"
- **H2** : Notre histoire, Nos valeurs, Nos certifications, Prêt à collaborer ?
- **Chiffres clés** : fondation 1995, 300+ collaborateurs, plusieurs sites spécialisés
- **Valeurs** : Innovation, Excellence, Éco-responsabilité, Collaboration
- **Certifications** : PEFC, FSC, Imprim Vert, ISO 14001
- **CTAs** : Contact, Simulateur

---

## Page `/contact`

- **Champs formulaire** : Prénom, Nom, Email, Téléphone, Entreprise (optionnel), Message + case consentement RGPD
- **Infos affichées** : tél `+33 (0)1 43 91 17 71`, email `digital@multi-poles.net`, adresse `53 rue des Deux Communes, 93100 Montreuil`
- **Horaires** : lun-ven 9h-18h, sam 9h-13h
- **CTA** : "Envoyer le message"
- **Carte intégrée** : absente

---

## Page `/devis`

- Formulaire **multi-étapes** (4 étapes)
- **Étape 1** : Type de projet (PLV / Packaging / Print / Devis 3D Studio / Autre), Description du projet
- ⚠️ **Bug affichage** : indicateur affiche "Étape 1 sur 425%" au lieu de "Étape 1 sur 4 (25%)" — problème de formatage

---

## Page `/simulateur`

- Iframe vers `studio-3d.html` (simulateur PLV & packaging procédural, Three.js 0.170)
- Tourne en plein écran sous le header

---

## Footer (global)

- Logo + tagline "Depuis 1996" (⚠️ incohérence avec "Depuis 1995" en home/apropos)
- **Nav "Nos Solutions"** : PLV, Packaging, Print, Devis 3D
- **Nav "Liens Utiles"** : Simulateur, Réalisations, Devis, À propos
- **Contact** : adresse, tél, email
- **Légal** : Mentions légales, Politique de confidentialité, Cookies
- **Réseaux sociaux** : LinkedIn, Twitter, Facebook — ⚠️ URLs basiques (`linkedin.com`, `twitter.com`, `facebook.com`), pas de profils réels pointés
- Copyright 2026

---

## SEO

### Meta visibles (home)

- **Title** : `Multi-Pôles | PLV et Packaging pour Cosmétiques & Pharmacie`
- **Meta description** : `Multi-Pôles, expert en PLV et packaging sur-mesure pour les secteurs cosmétique et pharmaceutique. Solutions innovantes et éco-responsables depuis 2005.`
- **OG** : title, description, site_name, locale `fr_FR`, image 1200×630, type website
- **Twitter Card** : ⚠️ non détecté
- **Canonical** : ⚠️ absent sur la home
- **JSON-LD / Schema.org** : ⚠️ non détecté (ni Organization, ni LocalBusiness, ni BreadcrumbList)

### Problèmes critiques détectés

1. 🔴 **`sitemap.xml` pointe vers `http://localhost:3002`** au lieu de `https://www.multi-poles.cloud`. Bug de build Next.js — la variable `NEXT_PUBLIC_SITE_URL` ou équivalent n'est pas injectée en prod. **Impact** : Google Search Console indexe des URLs cassées. À corriger urgemment.
2. 🔴 **`robots.txt` contient `Sitemap: http://localhost:3002/sitemap.xml`** — même racine de bug.
3. 🟡 **Incohérence date de fondation** : 1995 (home/apropos) vs 1996 (footer) vs 2005 (meta description).
4. 🟡 **Pages déclarées sans nav** : `/equipe` et `/blog` sont dans le sitemap mais invisibles depuis le menu principal — soit pages fantômes à supprimer du sitemap, soit pages à exposer.
5. 🟡 **Canonical absent** sur la home.
6. 🟡 **JSON-LD absent** — aucune donnée structurée pour Google (manque Organization, LocalBusiness avec l'adresse Montreuil, FAQPage pour la section FAQ de la home).
7. 🟡 **Liens sociaux footer** : pointent vers les homepages des réseaux au lieu des comptes réels.
8. 🟡 **Bug affichage `/devis`** : "425%" au lieu de "4 (25%)".

---

## Inventaire des composants interactifs

- **Slider carousel** (6 slides produits en home)
- **Galerie 16 images** (`/realisations` + vitrine home)
- **FAQ accordéon** (4 items)
- **Lecteur vidéo** embed (section 9 home)
- **Simulateur 3D** iframe Three.js (`/simulateur`)
- **Formulaire contact** (`/contact`)
- **Formulaire devis multi-étapes** (`/devis`, 4 étapes)

---

## Recommandations prioritaires

1. **Fixer `NEXT_PUBLIC_SITE_URL`** dans la config Next.js pour que sitemap + robots.txt pointent sur `https://www.multi-poles.cloud`. Priorité 1 SEO.
2. **Ajouter canonical + JSON-LD Organization + LocalBusiness** (adresse Montreuil, tél, horaires) en SEO de base.
3. **Uniformiser la date de fondation** partout (probablement 1995).
4. **Décider du sort de `/equipe` et `/blog`** : soit les construire, soit les retirer du sitemap.
5. **Corriger le bug "425%"** dans l'indicateur d'étapes du devis.
6. **Remplir les URLs sociales footer** avec les vrais profils Multi-Pôles.
7. **Ajouter FAQPage schema** sur la section FAQ home pour récupérer des rich snippets Google.
8. **Twitter Card meta** à ajouter (summary_large_image).
