# Liste des PKs RFN (Réseau Ferroviaire Français)

Données GeoJSON &amp; CSV des points kilométriques et hectométriques (PK) du réseau ferroviaire français, interpolées grâce aux données opendata de SNCF Réseau.

Télécharger au format GeoJSON
Télécharger au format CSV

## Sources utilisées

Une source unique est le fichier de formes des lignes, en GeoJSON avec PK de début et PK de fin, qui contient l'ensemble des vecteurs (lignes droites) formant une ligne ferroviaire.
https://data.sncf.com/explore/dataset/formes-des-lignes-du-rfn/table/

### Sources de « correction »

Ces sources sont utiles à la correction de positionnement de PK par rapport à des objets placés sur la ligne.

https://data.sncf.com/explore/dataset/liste-des-passages-a-niveau
https://data.sncf.com/explore/dataset/liste-des-gares

_Bien qu'étant nettement plus nombreux, la liste des signaux a dû être écartée en raison du trop grand nombre de corrections entraînant des comportements aléatoires._

## Méthode d'interpolation

1. On transforme le fichier source des formes de ligne en une liste de PK connus, en prenant chaque vecteur à la suite avec ses coordonnées géographiques. Grâce au PK initial connu on calcule les suivants par la distance entre les coordonnées de chaque vecteur, et ainsi de suite.
Par exemple, si le premier point du fichier de formes pour une ligne donnée est le PK 001+147, on calcule la distance en ligne droite jusqu'au point suivant du fichier pour cette ligne (fin du vecteur) qui est par exemple 820m. Le PK de ce point est donc 001+967, et ainsi de suite.
      
    - PENDANT CETTE PHASE On tente de corriger les PK de début de vecteurs ainsi trouvé avec la position des éventuels signaux contenus dans le vecteur.
      On calcule la différence de la distance entre
       1. les PKs du début de vecteur et celui du signal contenu dans celui-ci
       2. la distance réelle entre les coordonnées géographiques du début de vecteur et la position du signal
      On obtient ainsi le différentiel, qu'on ajoute au calcul des PKs successifs
    - On cherche les PKs hectométriques par interpolation entre les début et fins de vecteur avec calcul de la distance, avec l'orientation du vecteur

    Et on exporte le tout en geojson
