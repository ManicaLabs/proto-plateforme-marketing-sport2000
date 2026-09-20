# CDC - Prototype PWA « Plateforme collaborative marketing » (Sport 2000 / Céraclès)

> Document de reprise pour une prochaine session. La source de vérité du code est le dépôt git.

## 1. Contexte et objectif

Prototype interactif présenté par Manica Labs dans sa réponse à l'appel d'offres Céraclès Coopérative
« Plateforme collaborative marketing » (remplacement du portail OPECOM, 740 magasins, 8 enseignes).
Il illustre, avec des données fictives, le parcours d'un magasin (Grenoble Échirolles) :
tableau de bord à frise unifiée (opérations nationales + locales), commande d'une opération nationale,
personnalisation locale sur gabarit verrouillé avec envoi en BAT, suivi de projet multi-intervenants.

Contraintes : site statique, zéro backend, aucune donnée réelle, écrans non contractuels.
Cible : jury de la soutenance (3 septembre 2026) et équipes marketing Sport 2000, sur ordinateur et smartphone.
Le prototype doit être installable (PWA) et fonctionner hors ligne après la première visite.

## 2. Liens, accès et déploiement

- **App (prod)** : https://manicalabs.github.io/proto-plateforme-marketing-sport2000/
- **Dépôt** : https://github.com/ManicaLabs/proto-plateforme-marketing-sport2000 - GitHub Pages servi depuis `main`, racine
- **Déploiement = `git push origin main`** (build automatique, ~1 à 2 min ; vérifier Settings > Pages)
- **Auth** : push effectué par Cédric depuis son poste (`gh auth`) ; aucun token ne transite par les sessions Claude.
- **Cache Pages** : ~10 min ; rafraîchissement forcé parfois nécessaire. Le service worker versionne son cache
  (`VERSION` dans `sw.js`) : **incrémenter `pm-proto-vN` à chaque déploiement** pour forcer la mise à jour côté client.

### Fichiers du dépôt
| Fichier | Rôle |
|---|---|
| `index.html` | Application complète (CSS + JS inline), 4 écrans, barre d'onglets basse mobile, barre commande collante |
| `manifest.webmanifest` | Nom, couleurs, `display: standalone`, raccourcis `#cmd` et `#perso` |
| `sw.js` | Service worker : app shell en cache, réseau d'abord avec repli cache |
| `icons/` | Badge rond rouge 192 / 512 / maskable 512 / apple-touch 180 |
| `.nojekyll` | Désactive Jekyll sur Pages (indispensable) |
| `docs/CDC.md` | Ce document |

## 3. Architecture actuelle (v1, en prod)

- Écrans : `#boutique` (objets pub : `totB()` calcule les sous-totaux par circuit `data-circ` et la jauge franco), `#dash` (frise 2027, KPI, bandeau J-12), `#cmd` (Rentrée des classes 2027 : kit obligatoire, PLV, questionnaire, total au fil de l'eau), `#perso` (Anniversaire 10 ans : champs à gauche, aperçu temps réel à droite, envoi BAT), `#projet` (Dépliant Noël : 5 jalons, relecture BAT v3, intervenants).
- Navigation : `go(id)` bascule les sections `.screen` et synchronise les trois menus. Desktop : barre haute. Mobile (< 760 px) : bouton ☰ dans l'en-tête ouvrant un tiroir latéral (`.drawer`, 4 écrans avec descriptions, bouton d'installation, fermeture par ×, fond ou Échap), titre de l'écran courant dans l'en-tête (`#mob-title`), barre d'onglets basse `.bnav` en accès rapide, barre `#cmd-bar` collante sur l'écran commande.
- État : en mémoire uniquement (aucun `localStorage`), tout est réinitialisé au rechargement. C'est volontaire.
- PWA : enregistrement `sw.js` au `load` ; bouton `#btn-install` affiché sur `beforeinstallprompt` (Android / Chrome / Edge) ; iOS passe par Partager > Sur l'écran d'accueil.
- Polices : Barlow Condensed + Inter via Google Fonts (réseau ; mises en cache par le SW après première visite).

## 4. Détails à ne pas régresser

- Extracteur : jeu de données `LIGNES` généré à l'initialisation (8 magasins × 3 opérations × 5 articles, avec trous) ; le modèle « transport » agrège par imprimeur + magasin + adresse + article. L'export Excel dépend de `window.XLSX` (script cdnjs, `defer`, mis en cache par le SW après la première visite) ; repli CSV automatique.
- Gabarits : `ZONES_DEF` par format, un gabarit créé porte ses zones dans `data-zones` ; `gabarit()` masque champs et zones d'aperçu non ouverts.

- Assistant : 100 % local et scripté (aucun appel réseau, fonctionne hors ligne), étiqueté « réponses simulées ». Moteur `answer(q)` par expressions régulières sur le texte normalisé (sans accents) ; `creerLocale(q)` extrait la remise (`NN %`) et la date (`JJ mois`) pour pré-remplir `#f-promo` / `#f-date`, appelle `maj()` et ajoute une opération `.op.loc.plan[data-ia]` sur la dernière ligne de la frise. Une seule opération créée par session (garde `data-ia`).

- Charte : rouge `#D8121F`, noir `#17181C`, bandeau hachuré `.stripe` sous l'en-tête ; opérations locales en bleu `#1E5AA8` (pointillé = en préparation).
- Frise : grille 12 colonnes (`grid-column: N/span M`), `min-width: 920px` en mobile pour défilement horizontal.
- Commande : `tot()` recalcule kit / PLV / total (desktop et barre mobile `#r-total-m`) ; avertissement d'attribution automatique du kit obligatoire.
- Personnalisation : `maj()` recopie les champs dans l'aperçu ; zone créative verrouillée non éditable (principe du CDC client).
- Aucun tiret cadratin dans les textes (règle Manica) : utiliser « - ».

## 5. État d'avancement

- ✅ v1.0 : 4 écrans, responsive, PWA installable, hors ligne, icônes, manifest, README.
- ✅ v1.1 : URL de publication dans README et CDC.
- ✅ v1.5 : écran Espace centrale (`#admin`, 3 onglets) : création de fiches produits (support d'opération nationale → apparaît dans Commander ; objet pub → apparaît dans la boutique), création de gabarits d'impression locale (zones ouvertes cochables → gabarit sélectionnable dans Personnaliser), extracteur de rapports (modèles Répartition PLV / Plan de transport / Commandes / libre, champs cochables, filtres, aperçu, export Excel via SheetJS cdnjs ou CSV en repli). Commander : adresse de livraison par ligne (principale / réserve / entrepôt groupement) reprise dans le plan de transport. Personnaliser : choix de gabarit, zones affichées selon le gabarit, accroche locale 40 caractères, photo locale (FileReader), export du fichier prêt à imprimer (canvas PNG, fonds perdus, traits de coupe). SW `pm-proto-v5`.
- ✅ v1.4 : écran Objets publicitaires (boutique : 6 produits, circuits entrepôt / fournisseur, conditionnements, repiquage, alerte stock, panier par circuit avec franco fournisseur 150 €) ; 5e onglet dans les trois menus ; SW `pm-proto-v4`.
- ✅ v1.3 : assistant IA simulé (bouton ✦, panneau de conversation, 9 intentions par mots-clés, actions dans l'app : navigation, pré-remplissage de la personnalisation, création d'une opération locale sur la frise, lecture du total) ; SW `pm-proto-v3`.
- ✅ v1.2 : navigation mobile robuste (menu ☰ + tiroir, titre d'écran, barre basse conservée) ; SW `pm-proto-v2`.
- 🚧 Rien en cours.
- ⚠️ Aucune anomalie connue. Pas de capture de l'écran suivi de projet dans les documents d'offre (seulement dans l'app).

## 6. Prochain périmètre (si retenu)

- Assistant : brancher un vrai modèle (API) avec le contexte du magasin (opérations, commandes, échéances) ; création guidée de projets et de commandes par dialogue.

- Écran centrale (back-office) : liste des magasins n'ayant pas commandé, extraction par imprimeur.
- Mode « Voir en tant que » (assistance centrale).
- Notifications push réelles (nécessite un backend : hors périmètre prototype).

## 7. Backlog

- Capture mobile du tableau de bord pour les supports.
- Variante de charte par enseigne (Mondovelo, Espace Montagne...).

## 8. Pièges et conventions

- Sur mobile, ne jamais compter sur la seule barre basse fixée : Safari iOS et Chrome Android peuvent la recouvrir de leur barre d'outils. Le menu ☰ dans l'en-tête est le point d'entrée garanti.

- Changer `VERSION` dans `sw.js` à chaque déploiement, sinon les clients gardent l'ancienne version en cache.
- `.nojekyll` obligatoire (sinon Pages ignore certains fichiers).
- Validation avant push : `node --check` sur les blocs `<script>` extraits, `python -c "json.load(...)"` sur le manifest.
- Commits versionnés : `v1.1: <changement>`.

## 9. Checklist de démarrage de session

1. Lire ce CDC puis `index.html`.
2. Demander à Cédric l'URL Pages si la section 2 n'est pas complétée.
3. Implémenter, valider, remettre l'archive ou le diff ; Cédric pousse depuis son poste.
4. Mettre à jour ce CDC avant de clore.
