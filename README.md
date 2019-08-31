# Liste des PKs RFN (Réseau Ferroviaire Français)

Données GeoJSON &amp; CSV des points kilométriques et hectométriques (PK) du réseau ferroviaire français, interpolées grâce aux données opendata de SNCF Réseau.

[Télécharger au format GeoJSON](liste-des-pks.csv.zip)

[Télécharger au format CSV](liste-des-pks.geojson.zip)

## Sources utilisées

Une source unique est le fichier de formes des lignes, en GeoJSON avec PK de début et PK de fin, qui contient l'ensemble des vecteurs (lignes droites) formant une ligne ferroviaire.

Version du 22/05/2018 17:51 : https://data.sncf.com/explore/dataset/formes-des-lignes-du-rfn/table/


### Sources de « correction »

Ces sources sont utiles à la correction de positionnement de PK par rapport à des objets placés sur la ligne.

Version du 13/04/2018 07:54 : https://data.sncf.com/explore/dataset/liste-des-passages-a-niveau

Version du 24/07/2019 15:07 : https://data.sncf.com/explore/dataset/liste-des-gares

_Bien qu'étant nettement plus nombreux, la liste des signaux a dû être écartée en raison du trop grand nombre de corrections entraînant des comportements aléatoires._

## Limites


## Méthode d'interpolation

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
