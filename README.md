# r5.06-projet

## Data set :
"Parcoursup 2023 - vœux de poursuite d'études et de réorientation dans l'enseignement supérieur et réponses des établissements"
<br>
Il proviens de data.gouv.fr :
 www.data.gouv.fr/explore/parcoursup-2023-voeux-de-poursuite-detudes-et-de-reorientation-dans-lenseignement-superieur-et-reponses-des-etablissements-1?resource_id=6f622444-6596-424c-bca2-3e2fa763579e 

13 869 formations, 118 variables (capacité, candidats, classés, propositions et admis par type de bac, boursiers, mentions, origine géographique des admis, coordonnées GPS…).

## Problématique

> **À candidature égale, Parcoursup favorise-t-il les candidats dont le bac correspond à la filière visée ? Les quotas (bacs professionnels en BTS, bacs technologiques en BUT, boursiers) modifient-ils ces écarts, et sont-ils appliqués de façon homogène sur le territoire ?**

L'approche est **symétrique** : on ne cherche pas seulement si les bacheliers technologiques et professionnels sont désavantagés, mais aussi si les bacheliers généraux le sont dans les formations techniques et professionnelles. Les **boursiers** forment un axe à part : la bourse n'a aucun lien avec le parcours scolaire, un écart les concernant ne peut donc pas s'expliquer par l'adéquation entre le bac et la filière.

### Sous-questions
1. Existe-t-il un écart significatif entre la part de chaque profil (bac général, technologique, professionnel, boursier) parmi les **candidats** et sa part parmi les **propositions d'admission** ?
2. Les profils dont le bac correspond à la filière sont-ils favorisés, et les autres désavantagés ?
3. Quels facteurs expliquent ces écarts : type de filière, sélectivité, statut public/privé, tension (candidats par place), région ?
4. Les filières soumises à des quotas présentent-elles des écarts différents des autres ?
5. Les écarts varient-ils selon le territoire (Île-de-France, province, outre-mer), et les quotas sont-ils appliqués de la même façon partout ?

### Périmètre
- On compare les candidats aux **propositions**, et non aux admis : l'acceptation d'une proposition dépend du choix du candidat, pas de la sélection.
- On se concentre sur les filières où chaque profil postule réellement (par exemple BTS, IFSI et EFTS pour les bacs professionnels ; BUT, BTS et IFSI pour les bacs technologiques ; BTS et BUT pour étudier le désavantage des bacs généraux). Les boursiers sont étudiés dans toutes les filières.
- On exclut les formations où l'effectif d'un profil est trop faible (moins de 5 candidats) pour que les tests restent valides.
- Pour l'origine géographique, les académies de Paris, Créteil et Versailles sont regroupées (colonne prévue à cet effet dans le jeu de données).

### Premiers constats (descriptifs, sans test de significativité)
Ratio `R = part dans les propositions / part dans les candidats`.

**Par filière**
| Filière | R bac général | R bac techno | R bac pro | Lecture |
|---|---|---|---|---|
| BTS | 0,95 | 0,95 | 1,11 | les bacs pro sont favorisés (quota) |
| BUT | 0,98 | 1,09 | 0,31 | les bacs techno sont favorisés (quota), les bacs pro très désavantagés |
| IFSI | 1,35 | 0,89 | 0,63 | les bacs pro sont désavantagés dans une filière qui leur est ouverte |
| Licence_Las | 1,05 | 0,53 | 0,59 | les bacs techno et pro sont désavantagés |

**Bacheliers généraux désavantagés** dans certaines formations techniques : BTS Prothésiste dentaire (R = 0,32), BTS Métiers de la mode-vêtements (0,64), BTS Maintenance des véhicules (0,68), BUT Information-communication parcours publicité (0,70), BTS CIEL (0,77).

**Par région**
- **Île-de-France** : région la plus défavorable aux bacs techno et pro (R = 0,78 et 0,79 toutes filières), en particulier en licence (R bac techno = 0,63, contre 0,82 à 0,91 dans la plupart des régions). C'est aussi la seule région où les boursiers sont légèrement désavantagés (R = 0,92).
- **BTS** : l'avantage des bacs pro varie fortement selon la région (R = 1,27 en Hauts-de-France, 1,05 en Centre-Val de Loire). Le R des bacs généraux va de 0,74 (La Réunion) et 0,75 (Normandie) à 1,04 (Centre-Val de Loire).
- **BUT** : résultats homogènes dans toutes les régions (R bac techno entre 1,09 et 1,17), cohérent avec un quota national appliqué uniformément. Le BUT peut servir de **groupe témoin**.
- **Recrutement local** : en licence, 77 % des admis néo-bacheliers viennent de la même académie en province, 81 % en Île-de-France (3 académies regroupées) et 88 % en outre-mer. La part atteint 85 à 91 % en PASS et 84 à 95 % en BTS.

### Méthodologie envisagée
- **Indicateur :** ratio de représentation `R` (R = 1 : égalité ; R < 1 : profil désavantagé ; R > 1 : profil favorisé), calculé par formation, filière et région. L'étape intermédiaire des candidats classés permet de suivre l'entonnoir candidats → classés → propositions.
- **Tests :** χ² d'indépendance et tests de proportions, au global, par filière, par région et par formation.
- **Régression linéaire multiple :** expliquer R par les caractéristiques des formations (filière, sélectivité, statut, tension, présence d'un quota, région en variable catégorielle), puis repérer les formations dont l'écart au ratio attendu est significatif (outil de diagnostic).
- **ANOVA / Kruskal-Wallis :** comparer R entre filières, entre établissements publics et privés, et entre régions (ou Île-de-France / province / outre-mer).
- **Visualisation :** carte de France des ratios R par profil et par filière (par région ou à partir des coordonnées GPS).

### Démonstration prévue
On choisit une formation ou un filtre (filière, région, statut). L'outil affiche l'entonnoir par profil, le ratio R, le résultat du test (écart significatif ou non), la position de la formation par rapport au ratio prédit par la régression, et la carte correspondante.

### Limites connues
- Le niveau scolaire des candidats n'est pas disponible (les mentions au bac ne sont connues que pour les admis) : un écart peut venir d'une différence de dossier ou de prérequis et ne prouve pas une discrimination.
- Les données comptent des vœux, pas des individus : un même candidat apparaît dans chaque formation où il postule, et les bacheliers généraux font plus de vœux.
- Les populations comparées diffèrent légèrement (candidats néo-bacheliers en phase principale d'un côté, propositions aux candidats en terminale de l'autre).
- Les DROM et la Corse comptent peu de formations (47 à 231) : leurs ratios sont fragiles et doivent être interprétés avec prudence ou regroupés.
- Une seule session (2023) : on ne peut pas mesurer l'évolution des écarts dans le temps. Les sessions 2018 à 2022, disponibles sur data.gouv.fr, permettraient d'étudier l'effet des quotas.

## Exécution

Prérequis : Python 3.11 ou plus.

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook analyse.ipynb
```

### Contenu du dépôt
- `fr-esr-parcoursup_2023.csv` : jeu de données Parcoursup 2023 (data.gouv.fr).
- `data/regions.geojson` : contours simplifiés des régions de France métropolitaine ([france-geojson](https://github.com/gregoiredavid/france-geojson)), utilisés pour les cartes.
- `analyse.ipynb` : analyse exploratoire (entonnoir de sélection, ratio `R` par filière, tests du χ², bacheliers généraux en BTS/BUT, analyse par région, cartes).
- `requirements.txt` : dépendances Python.
