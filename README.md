# Avertissement !
Ces données sont publiées _en l'état_ sans garantie de contenu ni de suivi. Elles sont intégralement fabriquées, transformées, à partir des jeux de données placés en OpenData par SNCF et SNCF Réseau (cf https://data.sncf.com) et sous [licence ODbL](https://opendatacommons.org/licenses/odbl/1.0/index.html).

Ces données concernent exclusivement le RFN (Réseau Ferroviaire National français), dont SNCF Réseau a la charge.

# Contenu du dépôt

Ce dépôt contient à date :
- la liste des points kilométriques avec leur position géographique, fichier reconstitué, extrapolé
- la liste des quais au format geojson **en vecteurs** (transformation du fichier d'origine en points

# Détail pour chaque jeu de données

## Liste des quais

Données GeoJSON des quais de gare avec leurs détails, sous forme de vecteurs représentant les quais. En effet, le fichier GeoJSON original propose des points seulement.

### Sources utilisées

Version du 24/07/2019 12:10 : https://data.sncf.com/explore/dataset/liste-des-quais

### Méthodologie

Le fichier d'origine contient deux champs de propriétés c_geo_d et c_geo_f qui représentent les coordonnées de départ et fin de chaque quai. Ces coordonnées sont replacées dans un objet `lineString`, les propriétés sont filtrées et corrigées (notamment les PK sont transformés en nombres flottants et arrondis à une décimale), amputées des éléments non nécessaires.

## Liste des PKs

Données GeoJSON &amp; CSV des points kilométriques et hectométriques (PK) du réseau ferroviaire français, interpolées grâce aux données opendata de SNCF Réseau.

[Télécharger au format GeoJSON](liste-des-pks.csv.zip)

[Télécharger au format CSV](liste-des-pks.geojson.zip)

### Sources utilisées

Une source unique est le fichier de formes des lignes, en GeoJSON avec PK de début et PK de fin, qui contient l'ensemble des vecteurs (lignes droites) formant une ligne ferroviaire.

Version du 22/05/2018 17:51 : https://data.sncf.com/explore/dataset/formes-des-lignes-du-rfn/table/


### Sources de « correction »

Ces sources sont utiles à la correction de positionnement de PK par rapport à des objets placés sur la ligne.

Version du 13/04/2018 07:54 : https://data.sncf.com/explore/dataset/liste-des-passages-a-niveau

Version du 24/07/2019 15:07 : https://data.sncf.com/explore/dataset/liste-des-gares

_Bien qu'étant nettement plus nombreux, la liste des signaux a dû être écartée en raison du trop grand nombre de corrections entraînant des comportements aléatoires._

### Limites constatées

Les formes ne sont pas toujours précises quant à la réalité du terrain, de plus les points kilométriques de début ne sont parfois pas justes (relatif notamment à l'erreur de coordonnée) ce qui entraîne des décalages notamment en début de ligne.
De façon à pallier ces erreurs, les coordonnées sont corrigées avec des points appartenant à ces lignes (gares, pn, cf plus bas). Si le résultat devient plus satisfaisant des erreurs restent visibles, ne dépassant pas 100m à 200m en général, mais pouvant atteindre 800m dans certains cas, rares.

Les données sont ainsi proposées comme un effort « au mieux », en attendant un fichier de formes corrigé, voire un fichier des PKs émanant directement de SNCF Réseau.

### Méthode d'interpolation

1. On transforme le fichier source des formes de ligne en une liste de PK connus, en prenant chaque vecteur à la suite avec ses coordonnées géographiques. Grâce au PK initial connu on calcule les suivants par la distance entre les coordonnées de chaque vecteur, et ainsi de suite.
Par exemple, si le premier point du fichier de formes pour une ligne donnée est le PK 1.147, on calcule la distance en ligne droite jusqu'au point suivant du fichier pour cette ligne (fin du vecteur) qui est par exemple 820m. Le PK de ce point est donc 1.967, et ainsi de suite.
2. Pendant cette phase, on tente de corriger les PK de début de vecteurs ainsi trouvés avec la position des éventuels objets positionnés sur le vecteur en question (donc entre les deux PKs) venant d'autres fichiers (gares et passages à niveau ici).
Pour cela, on calcule la différence de la distance entre
  - le PK de début du vecteur et celui de l'objet contenu dans celui-ci
  - la distance réelle entre les coordonnées géographiques de début du vecteur et celles de l'objet
 On obtient ainsi **la valeur de correction**, qu'on soustrait à celle du PK de début avant le calcul du PK de fin de vecteur du point 1.
3. On reparcourt l'ensemble des lignes et PKs ainsi trouvés et corrigés pour rechercher les PKs hectométriques par interpolation entre les début et fins de vecteur.
Par exemple si le premier vecteur va du PK 1.147 au PK 1.967 les PKs nous intéressant sont les PK 1.2 1.3 1.4 1.5 1.6 1.7 1.8 et 1.9. On détermine alors l'orientation du vecteur grâce aux deux coordonnées de début et fin de celui-ci, puis la distance entre le PK de début et le premier PK intéressant (ici 53m pour aller de 1.147 à 1.200), et avec les coordonnées de départ, l'orientation et la distance, on obtient les coordonnées du PK qui nous intéresse.
On recommence ensuite pour les autres PKs contenus dans l'intervalle.
4. On exporte le tout en GeoJSON dans un fichiers de points avec ligne, pk et coordonnées.
