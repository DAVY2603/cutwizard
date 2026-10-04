# KENI CutWizard — déploiement web

**v140.1 (correctif)** : la v140 utilisait des chemins **absolus** (`/appweb/...`,
`/favicon.ico`, etc.) pour que `en/index.html` puisse charger les fichiers partagés à la racine
sans les dupliquer. Cela cassait l'aperçu Artifact de Claude.ai (page blanche), car ce service ne
sert pas la page à la racine réelle d'un domaine — un chemin absolu y pointait vers
`claude.ai/appweb/...` au lieu de l'espace propre à l'artefact, empêchant le bundle JS de charger.
Corrigé en repassant `index.html` (racine) en chemins **relatifs simples** (`appweb/...`,
`favicon.ico`, ...) et `en/index.html` en chemins relatifs **`../`** (`../appweb/...`,
`../favicon.ico`, ...) vers ces mêmes fichiers partagés à la racine. Ce schéma fonctionne à la
fois dans l'aperçu Artifact et sur l'hébergement OVH à la racine de `cutwizard.com`. Sur GitHub
Pages en sous-chemin de projet (`username.github.io/repo/`), ce schéma fonctionne aussi tel quel
(chemins relatifs, pas de dépendance à la racine du domaine).

**v140 (SEO bilingue)** : ajout d'une vraie page anglaise indexable à `/en/` (dossier `en/`
contenant son propre `index.html`), avec son propre title/description/keywords/Open Graph en
anglais, reliée à la page française par des balises `hreflang` réciproques (fr / en / x-default)
dans les deux `index.html` ET dans `sitemap.xml`. Les deux pages chargent le **même** bundle JS
partagé : seul un petit script inline dans `en/index.html`
(`window.__CUTWIZARD_LANG__ = 'en'`) indique à l'appli de démarrer en anglais sur cette page, pour
que le contenu affiché corresponde dès le chargement à son title/description anglais.

**v139 (SEO)** : mots-clés étendus dans `index.html` — ajout de scies cloches, fraises à carotter,
bimétal, HSS, cobalt, carbure, M42, M51, DIN 338, DIN 345, outils coupants.

**v139** : Scie cloche Carbure/Diamant — ajout des consignes de lubrification/refroidissement
communiquées par KENI, affichées dans la carte de résultat. Diamant : eau obligatoire pour toutes
les matières. Carbure : aucune lubrification (Fontes, Bois & dérivés, Plastiques/PVC), sauf huile
de coupe spéciale non-ferreux pour la matière Aluminium/Non ferreux.

**v138** : Scie cloche Carbure/Diamant — les vitesses de rotation viennent maintenant directement
des fichiers KENI `vitesses_scie_cloche_carbure_14_152mm.xlsx` et `vitesses_scie_cloche_diamant_eau.xlsx`
(tr/min par diamètre x matière ; Vc affiché est calculé à partir du tr/min par la formule standard
Vc = π×D×N/1000 — simple conversion d'unité, pas une valeur inventée). Matières Carbure : Fontes,
Aluminium/Non ferreux, Bois & dérivés, Plastiques/PVC (diamètres catalogue 14-152mm). Matières
Diamant : Faïence/Carrelage poreux, Grès/Grès cérame/Verre, Carreaux ciment/Mortier/Béton,
Matières composites (diamètres catalogue 6-127mm — KENI a indiqué une plage 6-133mm, mais le
fichier reçu s'arrête à 127mm ; à ajuster si une ligne manquante pour 130/133mm est fournie). Le
sélecteur de diamètre ne propose que les diamètres catalogue exacts pour ces deux qualités (pas de
palier ni d'extrapolation). Remplace la précédente implémentation v137 basée sur le catalogue K24
p.47, qui utilisait une répartition de matières différente.

Ce dossier contient les fichiers statiques du site. Il est prévu pour être publié sur le nom de
domaine **`cutwizard.com`** (hébergement OVH) — c'est l'URL en dur dans les balises SEO (voir
plus bas), donc utilisez bien ce domaine, ou mettez à jour ces fichiers si le domaine change.

## Publication sur OVH

1. Dans l'espace client OVH, créez l'hébergement web et pointez le nom de domaine `cutwizard.com`
   dessus (zone DNS → enregistrements A/CNAME selon le type d'hébergement).
2. Déposez **tout le contenu de ce dossier** (pas le dossier lui-même) à la racine de l'espace web
   via FTP/SFTP ou le gestionnaire de fichiers OVH : `index.html`, `favicon.ico`, `apple-touch-icon.png`,
   `icon-192.png`, `icon-512.png`, `manifest.json`, `robots.txt`, `sitemap.xml`, `og-image.png`,
   `metadata.json`, le dossier `appweb/` **et le dossier `en/`** (version anglaise, à `cutwizard.com/en/`).
3. Activez le certificat SSL (Let's Encrypt gratuit, généralement proposé automatiquement par OVH)
   pour que le site soit servi en `https://cutwizard.com/` — les moteurs de recherche pénalisent
   fortement un site en http simple.
4. Une fois en ligne, soumettez `https://cutwizard.com/sitemap.xml` dans Google Search Console
   (et Bing Webmaster Tools) pour accélérer l'indexation.

## Alternative : GitHub Pages

Ce dossier peut aussi être publié sur GitHub Pages si besoin (il est déjà initialisé en dépôt Git
local) :
```
git remote add origin https://github.com/<votre-compte>/cutwizard.git
git push -u origin main --force
```
Puis activez Pages dans les paramètres du dépôt (branche `main`, dossier racine `/`). Dans ce cas,
pensez à adapter les URLs en dur (`https://cutwizard.com/...`) dans `index.html`, `robots.txt` et
`sitemap.xml` pour qu'elles correspondent à l'URL réelle du site.

## Référencement (SEO) — ce qui a été mis en place

- `index.html` : balise `<title>` et `<meta name="description">` optimisées, balises Open Graph /
  Twitter Card (pour un aperçu propre quand le lien est partagé), données structurées JSON-LD
  (`schema.org/SoftwareApplication`), `<html lang="fr">`, balise canonique vers
  `https://cutwizard.com/`.
- `robots.txt` et `sitemap.xml` : autorisent l'indexation et indiquent l'unique page du site aux
  moteurs de recherche.
- `og-image.png` : image de partage (1200×630) utilisée par les réseaux sociaux et certains
  moteurs lors du partage du lien.
- `apple-touch-icon.png`, `icon-192.png`, `icon-512.png`, `manifest.json` : icônes et fichier de
  manifeste pour un rendu propre quand le site est ajouté à l'écran d'accueil (iOS/Android) ou
  installé comme application web (PWA).

## Compatibilité navigateurs & responsive — ce qui a été mis en place

- `-webkit-text-size-adjust: 100%` : empêche certains navigateurs mobiles (Safari/Chrome Android)
  de re-proportionner automatiquement le texte, par exemple lors d'une rotation d'écran.
- `height: 100dvh` (avec repli automatique sur `100%` pour les anciens navigateurs) : corrige le
  bug classique de Safari iOS où `100vh` inclut la zone sous la barre d'adresse rétractable, ce qui
  pouvait couper le bas de l'application ou laisser un espace vide.
- Taille de police des champs de saisie relevée à 16px sur le web (`App.js`, style `input`) : en
  dessous de 16px, Safari iOS zoome automatiquement la page quand on touche un champ, ce qui
  cassait la mise en page.
- Zones cliquables agrandies (`hitSlop`) sur les petites icônes (drapeaux de langue, réseaux
  sociaux) pour atteindre la taille minimale recommandée au toucher (44×44px), sans changer leur
  taille visuelle.
- Point de rupture intermédiaire ajouté à 600px (en plus de celui à 900px) pour une transition
  plus progressive entre mobile, tablette et desktop.
- Bandeau bas (3 colonnes) : passage automatique en pile verticale sous 380px de large (petits
  téléphones type iPhone SE), pour rester lisible.
- `<meta name="format-detection" content="telephone=no">` : évite qu'un navigateur mobile
  transforme automatiquement du texte en lien téléphonique, en plus des liens `tel:` déjà gérés
  explicitement dans le bandeau bas.

**Limite à connaître** : cette application est une "single-page app" (React Native Web) — le
contenu réel (textes, résultats) n'existe qu'une fois le JavaScript exécuté par le navigateur.
Google sait généralement exécuter ce JavaScript pour indexer le contenu, mais d'autres moteurs ou
robots plus simples peuvent ne voir que les balises `<head>` décrites ci-dessus. Pour un
référencement plus poussé (apparaître sur des recherches précises comme "calculateur vitesse
scie à ruban inox"), il faudrait à terme une vraie page de contenu statique (texte, FAQ) en plus
de l'application, ou un rendu côté serveur — à envisager si le trafic recherche devient important.

**⚠️ Important pour les prochaines mises à jour** : `index.html` est régénéré automatiquement à
chaque nouvelle version de l'application (commande `expo export`), ce qui réinitialise ce fichier
à sa version brute (sans les balises SEO, sans les correctifs de compatibilité ci-dessus, ni la
mise en page desktop/tablette). Toutes les balises et styles décrits ci-dessus doivent donc être
réinjectés à chaque déploiement, et les fichiers `robots.txt`, `sitemap.xml`, `og-image.png`,
`apple-touch-icon.png`, `icon-192.png`, `icon-512.png` et `manifest.json` doivent être recopiés
dans le nouveau dossier `dist/` (ils ne sont pas générés par `expo export`) — c'est fait
systématiquement par Claude à chaque nouvelle version livrée. Depuis la v140, `dist/en/index.html`
(page anglaise) doit lui aussi être recopié à chaque déploiement, et son `<script src="/appweb/...">`
réécrit avec le nouveau hash de bundle à chaque fois que le code change — c'est fait en même
temps que `index.html` racine par Claude.
