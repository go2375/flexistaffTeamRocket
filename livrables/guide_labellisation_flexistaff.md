# FlexiStaff — Guide de labellisation

| | |
|---|---|
| **Version** | 1.0 |
| **Date** | 16/11/2026 |
| **Équipe** | Team Rocket — Carole, Gosia, Khaoula |
| **Outil** | CVAT — organisation *FlexiStaff* |
| **Format d'export** | YOLO |

## 1. Objectif

Constituer un jeu d'images annotées destiné à l'entraînement d'un modèle YOLO capable de localiser les informations utiles dans des photographies de plannings. Les zones détectées sont ensuite transmises à un moteur OCR, puis interprétées pour produire des créneaux structurés et les proposer à l'ajout dans un agenda partagé CalDAV.

La priorité est de **détecter tous les créneaux utiles** : un événement oublié ne peut pas être traité lors des étapes suivantes. Le **rappel** constitue donc la métrique principale. La précision des rectangles reste essentielle pour limiter les zones inutiles soumises à l'OCR.

## 2. Données

- **Source :** photographies de plannings mises à disposition dans CVAT, au sein de l'organisation *FlexiStaff*.
- **Formats :** calendriers mensuels, plannings hebdomadaires, tableaux d'affectation et tableaux à codes horaires.
- **Variabilité :** mises en page, tailles de texte, couleurs, inclinaisons et qualité de prise de vue différentes.
- **Répartition :** annotation répartie entre les trois membres de l'équipe au moyen des jobs CVAT.
- **Préparation :** annotation des images fournies, sans recadrage ni redressement préalable.

## 3. Classes de labellisation

| Classe | Couleur | Définition | Exemples | Contre-exemples |
|---|---|---|---|---|
| `Titre_calendrier` | Jaune moyen | Texte identifiant la période du planning. | « NOVEMBRE 2027 », « PLANNING JUIN 2026 » | « Calendrier familial », note administrative |
| `Date_case` | Rose moyen | Numéro ou en-tête de date permettant de rattacher un créneau à un jour. | `1`, `3`, `29`, date en en-tête de colonne | Heure `15:45`, code `M06` |
| `Ressource` | Bleu vif | Nom de personne, fonction ou métier auquel des créneaux sont associés. | Nom d'agent dans une ligne, « Cariste », « Peintre » | « Inventaire », « Réunion équipe » |
| `Evenement` | Vert vif | Contenu d'un créneau ou code de planning correspondant à une entrée distincte. | « 16:00 Inventaire », « 15:45 Garage », `M06`, `J09`, `RH` | Numéro du jour, titre du mois, légende explicative |

Ces quatre classes couvrent les informations nécessaires à la localisation d'une période, d'une date, d'une ressource et d'un événement. Elles correspondent aux rôles attendus par l'application finale : `month_header`, `date_cell`, `resource_header` et `event`.

L'ordre numérique des classes est celui défini dans la configuration CVAT et repris dans l'export YOLO.

## 4. Règles générales de tracé

1. **Utiliser exclusivement des rectangles indépendants (`Shape`).** Ne pas employer le mode `Track`, qui propage les annotations sur plusieurs images.
2. **Tracer une boîte serrée autour du texte ou de l'information utile**, avec une marge minimale permettant de conserver l'intégralité des caractères.
3. **Ne pas englober toute une bande colorée** lorsque le texte utile n'en occupe qu'une partie.
4. **Une information distincte = un rectangle.** Deux événements dans une même case nécessitent deux annotations `Evenement`.
5. **Conserver une règle de tracé homogène** sur l'ensemble des images et entre les membres de l'équipe.
6. **Pour les images inclinées**, conserver leur orientation et ajuster les rectangles afin d'inclure le texte, sans intégrer de larges espaces inutiles.
7. **Pour un texte partiellement visible**, annoter uniquement la partie exploitable lorsqu'il reste identifiable. Ne pas attribuer de classe à un contenu impossible à reconnaître.
8. **Éviter les doublons** : une même information ne doit pas recevoir deux rectangles de la même classe.

## 5. Règles par classe

### `Titre_calendrier` — jaune moyen

- Encadrer le texte indiquant le mois, l'année ou la période du planning.
- Utiliser un seul rectangle pour un titre formant un bloc continu.
- Exclure les fonds décoratifs et les mentions secondaires situées à proximité.

### `Date_case` — rose moyen

- Dans les calendriers mensuels, annoter **tous les numéros de jour visibles**, y compris lorsque la case ne contient aucun événement.
- Encadrer le numéro de jour, sans englober toute la case.
- Dans les tableaux, encadrer l'en-tête de date utile à l'identification de la colonne ou de la ligne ; conserver ensemble le jour et son numéro lorsqu'ils forment un même bloc.
- Ne pas confondre une date avec une heure ou un code de planning.

### `Ressource` — bleu vif

- Encadrer le nom de personne, la fonction ou le métier servant à attribuer les événements d'une ligne ou d'une colonne.
- Ajuster le rectangle au texte, sans inclure l'ensemble de la cellule lorsque celui-ci est court.
- Ne pas classer les activités ou intitulés d'événements comme ressources.

### `Evenement` — vert vif

- Encadrer chaque événement au plus près de son texte : horaires, intitulé et informations associées au même créneau.
- Séparer les événements distincts, même lorsqu'ils figurent dans la même case de calendrier.
- Dans les plannings à codes, annoter chaque occurrence du code présente dans une cellule utile.
- Ne pas regrouper plusieurs cellules ou plusieurs codes dans un rectangle unique.

## 6. Cas particuliers

| Situation | Règle appliquée |
|---|---|
| Date présente dans une case vide | Annoter le numéro avec `Date_case`, sans créer d'`Evenement`. |
| Plusieurs événements dans la même case | Créer un rectangle `Evenement` par événement. |
| Bande colorée nettement plus large que le texte | Encadrer le texte utile uniquement. |
| Tableau à codes (`M06`, `J09`, `RH`, etc.) | Annoter chaque code comme `Evenement`, indépendamment des autres cellules. |
| Nom de personne en tête de ligne | Annoter le nom comme `Ressource`. |
| Photo légèrement inclinée | Conserver l'image et ajuster le rectangle au contenu visible. |
| Texte flou ou illisible | Ne pas inventer le contenu ni attribuer une annotation incertaine. |
| Mention « Calendrier familial » | Ne pas annoter : information descriptive secondaire. |
| Consigne « Toute modification doit être signalée avant 16h la veille » | Ne pas annoter : mention administrative, sans créneau à extraire. |

Les nouveaux cas particuliers sont consignés dans le guide afin de maintenir des règles identiques pour l'ensemble de l'équipe.

## 7. Éléments non annotés

- Fonds colorés, marges, bordures, séparateurs et zones vides.
- Mentions administratives ou descriptives sans lien avec la création d'un créneau.
- Notes de bas de page, consignes de modification et éléments décoratifs.
- Légendes explicatives des codes : elles décrivent la signification des codes, mais ne constituent pas des occurrences de créneaux.
- En-têtes génériques de jours sans date exploitable, lorsqu'ils n'apportent aucune information de rattachement.

## 8. Exemple de référence — `img028.jpg`

**Calendrier mensuel « NOVEMBRE 2027 » :**

- `Titre_calendrier` : un rectangle autour du titre « NOVEMBRE 2027 ».
- `Date_case` : un rectangle autour de chacun des **30 numéros de jour**, y compris sur les cases vides.
- `Evenement` : un rectangle par événement, ajusté au texte et non à l'intégralité de la bande colorée. La case du **3 novembre** comporte deux événements : **deux rectangles distincts**.
- `Ressource` : aucune annotation sur cette image en l'absence de ressource explicitement indiquée.

## 9. Contrôle qualité

Chaque job fait l'objet d'une vérification de cohérence avant l'export.

- [ ] Tous les titres, dates, ressources et événements utiles sont annotés.
- [ ] Les dates de calendrier sont annotées même lorsque les cases sont vides.
- [ ] Chaque événement distinct possède son propre rectangle.
- [ ] Les rectangles entourent le contenu utile sans couper de caractères ni englober des zones excessives.
- [ ] Aucun objet `Track` n'est présent.
- [ ] Aucune annotation en double n'est conservée.
- [ ] Les classes sont utilisées conformément à leurs définitions.
- [ ] Les images sont relues pour détecter les oublis et incohérences.

La relecture croisée entre membres de l'équipe permet d'harmoniser les annotations avant l'entraînement.

## 10. Organisation du travail

L'annotation est réalisée par les trois membres de **Team Rocket** : Carole, Gosia et Khaoula, sur les jobs attribués dans CVAT. Les décisions de labellisation sont communes et appliquées uniformément sur l'ensemble du jeu d'images.

Le dataset est exporté au format YOLO après vérification des annotations. Il est ensuite préparé pour l'entraînement et la validation du modèle.

## 11. Historique des versions

| Version | Date | Modification |
|---|---|---|
| 1.0 | 16/11/2026 | Création du guide de labellisation : classes, règles de tracé, cas particuliers et contrôle qualité. |
