# KENI CutWizard — déploiement web

**v144.7 (logo MK Morse sur chaque carte de résultat)** : l'utilisateur a demandé d'afficher le
logo du fabricant (catalogue MK Morse) sur chaque résultat de lame, aligné à droite sur la même
ligne que le nom de la famille. Recherche effectuée dans `Catalogue_KENI_-_K24.pdf` (pages 9 à 18,
couvrant les 7 familles bimétal et les 4 familles carbure) : chaque page produit ne porte pas une
icône distincte par famille, seulement le nom du produit en typographie stylisée (déjà affiché en
texte dans l'app) et, de façon constante sur toutes les pages, la marque du fabricant "MORSE" en
haut de page. Ce logo MK Morse a donc été extrait à haute résolution (600 dpi) depuis le catalogue,
détouré (fond transparent), recadré et converti en PNG léger (~1.7 Ko) encodé en base64
(`MORSE_LOGO_B64`), sur le modèle des icônes `SOCIAL_*_B64` déjà présentes dans `App.js`. Il est
désormais affiché dans `CandidateCard` (toutes les variantes : résultat complet, lame non adaptée,
case vide) aligné à droite, sur la même ligne que le nom de la famille de lame (nouveau style
`famNameRow` + `famLogo`), pour chacune des 5 cartes de résultat (bimétal ×2, carbure, options
supplémentaires). Le même logo est utilisé pour toutes les familles car toutes les lames du
calculateur (bimétal et carbure) sont des produits de la marque MK Morse.

**v144.6 (correction du vrai bug : liens sans effet dans l'aperçu Artifact)** : l'utilisateur a
signalé qu'aucun lien externe (référence KENI, mentions légales, téléphone, adresse, réseaux
sociaux) ne produisait d'effet visible dans l'aperçu Artifact de Claude.ai. Cause identifiée :
tous ces liens utilisaient `window.open(url, '_blank')`, un appel scripté que l'iframe en bac à
sable de l'aperçu Artifact (et le bloqueur de popups de nombreux navigateurs) bloque souvent
silencieusement — sans erreur visible, juste aucune action. Corrigé en s'appuyant sur le mécanisme
natif de react-native-web : chaque `TouchableOpacity` concerné reçoit maintenant aussi les props
`href`/`hrefAttrs`, que react-native-web transforme en un vrai élément HTML `<a href target="_blank">`
sur le web — une navigation d'ancre authentique gérée nativement par le navigateur, qui fonctionne
de façon fiable y compris dans l'iframe en bac à sable (contrairement à un `window.open()`
scripté). Le comportement sur mobile (Snack/natif), qui utilisait déjà `Linking.openURL`, est
inchangé. Les liens concernés : référence KENI de chaque carte de résultat, logo d'en-tête,
mentions légales/CGV/confidentialité, téléphone, adresse, et les 4 réseaux sociaux du bandeau bas.

**v144.5 (annule la v144.4 — lien de recherche confirmé par KENI)** : la v144.4 avait remplacé le
lien vers `https://keni-sa.com/recherche?q=...` par un lien vers la page d'accueil, en pensant
cette URL de recherche inexistante (le site keni-sa.com est une appli Next.js dont la recherche
semblait, de l'extérieur, fonctionner uniquement côté client via ⌘K/Ctrl+K). KENI a confirmé que
`https://keni-sa.com/recherche?q=` suivi de la référence encodée est bien le bon format d'URL de
recherche du site. Revenu à ce lien d'origine : `https://keni-sa.com/recherche?q=` concaténé avec
la référence complète (référence catalogue + longueur de lame), et au texte de bouton d'origine
("Cliquez ici pour commander cette référence" / "Click here to order this reference").

**v144.4 (tentative de correction, annulée en v144.5)** : le bouton de chaque carte de résultat
pointait vers `https://keni-sa.com/recherche?q=...`, une URL qui n'existe pas sur le site réel —
confirmé en consultant keni-sa.com : c'est un site Next.js dont la recherche (raccourci clavier
⌘K / Ctrl+K) fonctionne uniquement côté client, sans URL de résultats dédiée à laquelle on
pourrait faire pointer un lien, d'où le lien cassé (en français comme en anglais, signalé par
l'utilisateur). Corrigé en pointant vers la page d'accueil du site (`https://keni-sa.com`, vérifiée
fonctionnelle) et, sur le web, en copiant automatiquement la référence dans le presse-papiers au
clic (`navigator.clipboard`, sans dépendance supplémentaire) pour que l'utilisateur n'ait plus qu'à
la coller dans la recherche du site une fois dessus. Sur mobile (Snack), la copie est silencieusement
ignorée si l'API n'existe pas — la référence reste de toute façon affichée à l'écran. Texte du
bouton mis à jour en conséquence ("Cliquez pour rechercher cette référence sur keni-sa.com" /
"Click to search this reference on keni-sa.com").

**v144.3 (lubrification déplacée dans chaque carte de résultat)** : la consigne de lubrification,
qui était affichée une seule fois dans le bloc "Informations matière & risque", est maintenant
répétée dans chaque carte de résultat (une par lame recommandée, bimétal et carbure) — c'est au
moment de choisir et monter la lame qu'on veut l'avoir sous les yeux, pas seulement en lisant les
propriétés générales de la matière. Le bloc matière ne garde que le risque d'écrouissage comme
point de vigilance mis en valeur (propriété intrinsèque à la matière, indépendante de la lame
choisie).

**v144.2 (déplacement + refonte visuelle du bloc surface/volume)** : la surface de la section
coupée et le volume de copeaux enlevé (ajoutés en v144) sont désormais affichés dans le bloc
"Informations matière & risque" plutôt que dans le bloc "Dimensions de la pièce", puisqu'il s'agit
d'un résultat de calcul au même titre que les recommandations de lame, pas d'une donnée de saisie.
Profité de ce déplacement pour améliorer la présentation de l'ensemble du bloc matière :
- Les propriétés neutres (dureté, usinabilité, traitement thermique) sont maintenant dans une
  grille de petites fiches, plus lisible qu'une liste de lignes "label : valeur".
- Les deux points de vigilance opérationnelle (risque d'écrouissage, lubrification) sont mis en
  valeur à part, dans des encadrés dédiés (fond teinté), pour bien les distinguer des propriétés
  neutres ci-dessus.
- La surface et le volume sont affichés en bas du bloc, sous un séparateur, avec le même style de
  "tuile" (StatTile) que les résultats de lame — plus un rappel explicite : ces deux valeurs
  dépendent de la lame choisie (épaisseur) et des dimensions de la pièce, et se recalculent donc
  automatiquement à chaque changement de l'un ou l'autre (déjà le cas techniquement depuis la v144
  — un seul calcul réactif commun à tous les résultats affichés — mais rendu explicite à l'écran
  à la demande de l'utilisateur).

**v144 (surface de la section coupée + volume de copeaux enlevé)** : ajout d'un nouveau résultat
dans le calculateur scie à ruban — pour chaque calcul, l'appli affiche désormais (unités en cm²/cm³
pour rester lisibles, calcul interne en mm puis conversion finale) :
1. **Surface de la section coupée (cm²)** : l'aire de la section de la pièce au plan de coupe,
   avec une formule dédiée selon la forme :
   - Ronde pleine : S = π×D²/4.
   - Carrée/rectangulaire pleine : S = côté1 × côté2.
   - Tube rond (creux) : S = π/4 × (D²ext − D²int), D_int = D_ext − 2×épaisseur de paroi.
   - Tube rectangulaire (creux) : S = (A×B) − (A−2t)×(B−2t), t = épaisseur de paroi uniforme.
   - Profilé cornière (L) à ailes égales : S = t×(2A − t) (A = longueur d'aile, t = épaisseur),
     formule standard de section d'angle à coin vif.
   - Profilé U et H/I : S = A×B − (A−2D)×(B−C) (A = largeur, B = hauteur, C = épaisseur âme,
     D = épaisseur aile) — la même formule nette s'applique aux deux, seule la position de l'âme
     (en haut pour U, au centre pour H/I) diffère, sans changer l'aire totale retirée.
2. **Volume de copeaux enlevé en un passage (cm³)** = Surface de la section × nombre total de
   pièces engagées (nbH × nbV) × épaisseur de la lame (kerf, prise directement dans les
   dimensions de lame du catalogue KENI — onglet "Lames disponibles"). Représente le volume de
   matière transformé en copeaux lorsque la lame traverse entièrement le paquet de pièces en un
   seul passage.
   Toutes les formules ont été vérifiées analytiquement (calcul à la main vs. moteur JS réel) pour
   chaque forme avant publication.

**v143 (correction Vc inox 316/316L + bug table diamètre)** : en comparant les résultats de
CutWizard au "Lenox Guide to Band Sawing" (le document source officiel fourni en pièce jointe au
début du projet, table BI-METAL SPEED CHART p.20-21, référence Ø100mm matière recuite, lame
bimétal, arrosage), deux problèmes ont été identifiés :
1. **vcBim du 316/316L était trop haut** : Lenox indique 25 m/min pour le 316 à 100mm (contre 37
   dans la base KENI, puis 35 après un 1er ajustement) — ramené à **27 m/min** (base, avant
   correction diamètre), qui donne ~30 m/min à Ø50mm, cohérent avec la fourchette KENI (25-35
   m/min) et avec la source Lenox.
2. **Bug dans la table de correction par diamètre (`facteurTailleBim`)** : les paliers Lenox
   officiels (6mm:+15%, 19mm:+12%, 32mm:+10%, 64mm:+5%, 100mm:référence, 200mm:-12%) étaient
   mal indexés — le palier "64mm:+5%" était enregistré avec un facteur de +0% au lieu de +5%, et
   le palier "100mm:référence" avait un facteur de -3% au lieu de 0%. Ce bug sous-estimait
   systématiquement la vitesse de coupe de ~3 à 7% pour toute pièce entre 50 et 150mm de diamètre.
   Corrigé en réalignant la table exactement sur les seuils et facteurs publiés par Lenox. Seul le
   calcul bimétal est concerné (le carbure n'applique pas de correction diamètre dans le modèle
   KENI). Le reste du catalogue acier (1018, A36, 1045, 4140, 4340, 12L14, 304, A2, D2) a été
   vérifié contre le même document Lenox et est fidèle à ±10% — aucune autre correction nécessaire.

**v142 (correction Vc inox 316/316L, 1er ajustement)** : valeurs précédentes (37/62 m/min,
identiques pour 304/316/316L) dépassaient les fourchettes fournisseur communiquées par KENI pour
le 316 et le 316L ; ajustées une 1ère fois à 35/50 m/min (voir v143 ci-dessus pour l'ajustement
final du bimétal après analyse de la source Lenox).

**v141** : renommage "Trepanning cutter" → "Hole Cutter" (page anglaise).

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
