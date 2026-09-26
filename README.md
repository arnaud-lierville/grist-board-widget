# Board — widget kanban pour Grist

Widget personnalisé Grist qui affiche une table de tâches sous forme de tableau kanban : une colonne par étape,
glisser-déposer entre étapes, création et modification des tâches dans une fenêtre, recherche et filtres.

Un seul fichier (`index.html`), sans dépendance ni build. Il doit être hébergé en HTTPS (GitHub Pages, Vercel…)
pour être chargé dans Grist.

---

## 1. Tables Grist

Deux tables : `Etapes` (les colonnes du tableau) et `Taches` (les cartes).
Les **identifiants** de colonnes ci-dessous sont sans accents ; les **libellés** peuvent être librement accentués
(« Échéance » → identifiant `Echeance`).

### Table `Etapes`

| Colonne   | Type       | Obligatoire | Rôle |
|-----------|------------|-------------|------|
| `Nom`     | Texte      | oui         | Nom affiché en tête de colonne |
| `Ordre`   | Numérique  | recommandé  | Ordre des colonnes, de gauche à droite |
| `Couleur` | Texte      | non         | Couleur du bandeau, en hexadécimal (`#16b378`) |
| `Limite`  | Entier     | non         | Nombre maximal de cartes (vide ou 0 = illimité) ; le compteur passe en rouge si dépassé |

Exemple :

| Nom        | Ordre | Couleur | Limite |
|------------|-------|---------|--------|
| À faire    | 1     | #8a8f98 |        |
| En cours   | 2     | #3b82f6 | 5      |
| En attente | 3     | #f59e0b |        |
| Terminé    | 4     | #16b378 |        |

La **dernière étape** (plus grand `Ordre`) est considérée comme « terminée » : ses tâches ne sont jamais signalées en retard.

> Le widget reconnaît les colonnes de cette table par leur nom : `Ordre` (ou `Order`, `Position`, `Rang`),
> `Couleur` (ou `Color`), `Limite` (ou `Limit`, `WIP`, `Max`).

### Table `Taches`

| Colonne       | Type                        | Obligatoire | Affichage sur la carte |
|---------------|-----------------------------|-------------|------------------------|
| `Titre`       | Texte                       | oui         | Titre |
| `Etape`       | Référence → `Etapes`        | oui         | Colonne du tableau |
| `Echeance`    | Date                        | non         | En haut à gauche (rouge si en retard) |
| `Priorite`    | Choix                       | non         | Badge en haut à droite |
| `Description` | Texte                       | non         | Sous le titre (3 lignes max) |
| `Etiquettes`  | Choix multiple              | non         | Badges en bas de la carte |
| `Lien`        | Texte (URL)                 | non         | « 🔗 domaine » en bas à droite, ouvert dans un nouvel onglet |
| `Ordre`       | Numérique                   | non         | Aucun (sert au tri manuel par glisser-déposer) |

D'autres colonnes peuvent être ajoutées librement (ex. `Notes`) : elles apparaissent dans la fenêtre d'édition
et peuvent être affichées sur la carte (voir « Autres champs » plus bas).

**Priorité** — la couleur du badge dépend de l'**ordre des choix** définis dans la colonne :
1er choix rouge, 2e orange, 3e vert, suivants gris. Définir les choix dans l'ordre `Haute`, `Moyenne`, `Basse`.

**Lien** — seules les adresses `http://` et `https://` sont affichées ; `https://` est ajouté si l'adresse est
saisie sans (ex. `github.com/…`).

---

## 2. Configurer la référence `Etape`

1. Dans la table `Taches`, sélectionner la colonne `Etape`.
2. Panneau de droite → **Colonne** → Type : **Référence**.
3. **Données de la table** : `Etapes`.
4. **Afficher la colonne** : `Nom`.

Une colonne `Etape` de type **Choix** fonctionne aussi (les colonnes du tableau sont alors les choix, avec leurs
couleurs Grist), mais on perd l'ordre, les limites et l'ajout/renommage d'étapes depuis le widget.

---

## 3. Importer les données d'exemple (facultatif)

Les fichiers `Etapes.csv` et `Taches.csv` du dépôt correspondent au schéma ci-dessus.

1. Importer **`Etapes.csv` en premier**, puis `Taches.csv`.
2. L'import crée des colonnes de type Texte : convertir ensuite dans `Taches` :
   - `Etape` → Référence vers `Etapes`, colonne affichée `Nom` (section 2) ;
   - `Echeance` → Date ;
   - `Priorite` → Choix, puis ranger les choix dans l'ordre `Haute`, `Moyenne`, `Basse` ;
   - `Etiquettes` → Choix multiple ;
   - `Ordre` → Numérique.
3. Dans `Etapes` : `Ordre` → Numérique, `Limite` → Entier.

---

## 4. Ajouter et configurer le widget

### Ajout

1. Sur la page voulue : **Nouveau** → **Ajouter une vue à la page**.
2. Sélectionner le widget : **Personnalisé** ; sélectionner les données : table **`Taches`**.
3. Dans le panneau de droite (onglet **Widget**) : **URL personnalisée**, puis coller l'adresse où `index.html`
   est hébergé.
4. **Niveau d'accès** : choisir **Accès complet au document**. Il est nécessaire pour lire la table `Etapes` et
   la structure des colonnes, et pour enregistrer les modifications.

### Associations des colonnes

Toujours dans l'onglet **Widget**, associer chaque champ à une colonne de `Taches` :

| Champ du widget                       | Colonne à associer | Obligatoire |
|---------------------------------------|--------------------|-------------|
| Titre de la tâche                     | `Titre`            | oui |
| Étape                                 | `Etape`            | oui |
| Échéance                              | `Echeance`         | non |
| Priorité                              | `Priorite`         | non |
| Étiquettes                            | `Etiquettes`       | non |
| Description                           | `Description`      | non |
| Lien                                  | `Lien`             | non |
| Ordre manuel                          | `Ordre`            | non (active le tri « Manuel ») |
| Autres champs affichés sur la carte   | plusieurs colonnes au choix (ex. `Notes`) | non |

Un champ non associé est simplement absent de la carte (et le filtre ou le tri correspondant disparaît).
Les noms de colonnes sont libres : c'est l'association qui compte, pas le nom.

---

## 5. Utilisation

- **Créer une tâche** : bouton `+` en tête de colonne.
- **Modifier / supprimer** : clic sur la carte (la suppression demande un second clic de confirmation).
  `Ctrl`/`Cmd` + `Entrée` enregistre, `Échap` ferme.
- **Changer d'étape** : glisser la carte dans une autre colonne.
- **Réordonner** : glisser la carte dans sa colonne (nécessite le champ **Ordre manuel** ; le tri passe en « Manuel »).
- **Étapes** : **+ Étape** dans la barre supérieure pour en ajouter une ; double-clic sur un nom de colonne pour le
  renommer. Suppression et réordonnancement se font dans la table `Etapes`.
- **Barre supérieure** : recherche plein texte, filtres Priorité / Étiquette / En retard, choix du tri
  (Échéance, Priorité, Titre, Manuel). **Réinitialiser** (visible en tri Manuel) revient au tri par échéance
  sans effacer l'ordre manuel. **↻** relit les étapes et la structure des tables.

Le tri choisi est mémorisé dans la configuration du widget.

---

## 6. Dépannage

| Symptôme | Cause probable |
|----------|----------------|
| « Associe au minimum Titre de la tâche et Étape » | Champs obligatoires non associés dans l'onglet Widget. |
| Tout est dans « Sans étape » | `Etape` n'est pas une Référence vers `Etapes` (ou un Choix), ou les valeurs ne correspondent à aucune étape. |
| Colonnes nommées `#1`, `#2`… | Aucune colonne affichée définie sur la référence `Etape` et pas de colonne `Nom` dans `Etapes`. |
| Badge priorité gris | La colonne `Priorite` n'est pas de type Choix, ou la valeur n'est pas parmi les 3 premiers choix. |
| Pas de bouton « + Étape » | `Etape` est de type Choix (et non Référence). |
| Modifications refusées | Niveau d'accès du widget différent de « Accès complet au document ». |
| Nouvelle colonne ou étape absente | Cliquer sur **↻** (les étapes sont aussi relues automatiquement toutes les 15 s). |

Le widget est prévu pour un usage sur ordinateur (pas de glisser-déposer tactile).
