# Board widget pour Grist

Widget personnalisé Grist (« Custom widget → URL personnalisée ») affichant une table de tâches en tableau kanban.
Un seul fichier : `index.html` (HTML/CSS/JS vanilla, aucune dépendance, aucun build). Hébergé via GitHub Pages / Vercel.
Instance Grist cible : grist.numerique.gouv.fr. Interface et commentaires en français.

## Fichiers
- `index.html` — le widget.
- `Etapes.csv`, `Taches.csv` — données d'exemple conformes au schéma ci-dessous.

## Schéma Grist recommandé (identifiants sans accents ; libellés libres)
Table `Taches` : `Titre` (Texte, requis), `Etape` (Référence → `Etapes`, colonne affichée `Nom`, requis),
`Echeance` (Date), `Priorite` (Choix, dans l'ordre `Haute`, `Moyenne`, `Basse`), `Description` (Texte),
`Etiquettes` (Choix multiple), `Lien` (Texte, URL), `Ordre` (Numérique, tri manuel). Autres colonnes libres (ex. `Notes`).
Table `Etapes` : `Nom` (Texte), `Ordre` (Numérique), `Couleur` (Texte #hex), `Limite` (Entier, vide/0 = illimité).
La dernière étape (plus grand `Ordre`) = « terminé » (jamais en retard).
Après import CSV : vérifier/convertir les types (Référence, Choix, Choix multiple, Date) dans Grist.

## Choix de conception validés avec l'utilisateur
- Widget générique : colonnes associées dans le panneau Grist (mappings) :
  `Titre` (requis), `Etape` (requis ; Référence vers une table d'étapes, ou à défaut colonne Choix),
  `Echeance`, `Priorite`, `Etiquettes`, `Description`, `Lien`, `Ordre` (facultatifs), `Carte` (allowMultiple :
  autres champs affichés sur la carte, en plus des emplacements fixes ; défaut = aucun).
- Carte : en haut échéance (gauche) + badge priorité (droite ; couleur selon l'index du choix : 1er rouge, 2e orange,
  3e vert, suivants gris) ; titre ; description (3 lignes max) ; autres champs ; en bas étiquettes + lien « 🔗 domaine »
  (target _blank, http/https seulement, https:// ajouté si absent).
- Colonnes du board = lignes de la table Etapes, triées par `Ordre`, couleur `Couleur`, compteur rouge si `Limite` dépassée
  (dépôt quand même autorisé). Colonnes de la table des étapes détectées par nom (ordre/order, couleur/color, limite/limit),
  nom affiché = colonne d'affichage (visibleCol) de la référence.
- Colonne « Sans étape » affichée seulement s'il existe des tâches sans étape valide.
- Étapes : renommage par double-clic sur l'en-tête, ajout via « + Étape » (barre supérieure) ; suppression/réordonnancement
  dans Grist. Table des étapes relue toutes les 15 s (onRecords ne notifie pas les autres tables).
- CRUD dans une fenêtre modale générée depuis les métadonnées (`_grist_Tables_column`) : le formulaire s'adapte
  aux colonnes ajoutées/supprimées dans Grist (Text, Numeric/Int, Bool, Date, DateTime, Choice, ChoiceList, Ref ;
  formules et autres types en lecture seule). Suppression avec double clic de confirmation.
- Tri : Échéance (défaut) | Priorité (ordre des choix) | Titre | Manuel (colonne `Ordre`, pas de 10).
  Glisser-déposer dans une même colonne ⇒ passage en Manuel à partir de l'ordre affiché ; « Réinitialiser » revient
  au tri par échéance (ne modifie pas `Ordre`). Entre colonnes : change l'étape (et la position si mode Manuel). Mode stocké via grist.setOption('tri').
- Recherche plein texte + filtres Priorité, Étiquette, « En retard » (échéance passée, sauf dernière colonne = terminé).
- Ordinateur uniquement (pas de glisser-déposer tactile).

## Points techniques
- L'API Grist est chargée dynamiquement depuis l'origine de la page parente (`document.referrer`), puis
  grist.numerique.gouv.fr, puis docs.getgrist.com (docs.getgrist.com était bloqué chez l'utilisateur).
- `onRecords` sur cette instance ne renvoie que les colonnes associées : le widget relit la table complète avec
  `docApi.fetchTable` (`loadFullRows` / `fullRow`) pour le formulaire, les cartes et la recherche.
- Écritures via `docApi.applyUserActions` (UpdateRecord / AddRecord / RemoveRecord). Dates écrites en secondes UTC,
  ChoiceList en `['L', ...]`, Ref en id (0 = vide).
- Valeurs lues tolérantes : Date en objet Date, nombre (s) ou `['d', n]` ; ChoiceList en tableau avec ou sans `'L'`.

## Tests
Pas de suite de tests dans le dépôt. Les tests ont été faits avec Playwright et un faux objet `window.grist`
(injecté avant le chargement, avec `window.__GRIST_MOCK__ = true`) simulant fetchTable / applyUserActions / onRecords.
