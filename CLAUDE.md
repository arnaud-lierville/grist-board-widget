# Board widget pour Grist

Widget personnalisé Grist (« Custom widget → URL personnalisée ») affichant une table de tâches en tableau kanban.
Un seul fichier : `index.html` (HTML/CSS/JS vanilla, aucune dépendance, aucun build). Hébergé via GitHub Pages / Vercel.
Instance Grist cible : grist.numerique.gouv.fr. Interface et commentaires en français.

## Fichiers
- `index.html` — le widget.
- `Statuts.csv` — table d'exemple des statuts : `Nom`, `Ordre`, `Couleur` (#hex), `Limite` (nb max de cartes, vide = illimité).
- `Taches.csv` — table d'exemple des tâches : `Titre`, `Statut` (Référence → Statuts), `Description`, `Echeance` (Date),
  `Priorite` (Choix), `Etiquettes` (Choix multiple), `Notes`, `Ordre` (Numérique).

## Choix de conception validés avec l'utilisateur
- Widget générique : colonnes associées dans le panneau Grist (mappings) :
  `Titre` (requis), `Statut` (requis ; Référence vers une table de statuts, ou à défaut colonne Choix),
  `Echeance`, `Priorite`, `Etiquettes`, `Ordre` (facultatifs), `Carte` (allowMultiple : champs affichés sur la carte ;
  défaut = échéance, priorité, étiquettes).
- Colonnes du board = lignes de la table Statuts, triées par `Ordre`, couleur `Couleur`, compteur rouge si `Limite` dépassée
  (dépôt quand même autorisé). Colonnes de la table statuts détectées par nom (ordre/order, couleur/color, limite/limit),
  nom affiché = colonne d'affichage (visibleCol) de la référence.
- Colonne « Sans statut » affichée seulement s'il existe des tâches sans statut valide.
- Statuts : renommage par double-clic sur l'en-tête, ajout via « + Statut » ; suppression/réordonnancement dans Grist.
  Table statuts relue toutes les 15 s (onRecords ne notifie pas les autres tables).
- CRUD dans une fenêtre modale générée depuis les métadonnées (`_grist_Tables_column`) : le formulaire s'adapte
  aux colonnes ajoutées/supprimées dans Grist (Text, Numeric/Int, Bool, Date, DateTime, Choice, ChoiceList, Ref ;
  formules et autres types en lecture seule). Suppression avec double clic de confirmation.
- Tri : Échéance (défaut) | Priorité (ordre des choix) | Titre | Manuel (colonne `Ordre`, pas de 10).
  Glisser-déposer dans une même colonne ⇒ passage en Manuel à partir de l'ordre affiché ; « Réinitialiser » revient
  au tri par échéance. Entre colonnes : change le statut (et la position si mode Manuel). Mode stocké via grist.setOption('tri').
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
