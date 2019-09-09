> **Avertissement !**
>
> Ces données sont publiées _en l'état_ sans garantie de contenu ni de suivi, à utiliser à vos risques et périls. Elles sont intégralement fabriquées, transformées, à partir des jeux de données placés en OpenData par SNCF et SNCF Réseau (cf https://data.sncf.com) et sous [licence ODbL](https://opendatacommons.org/licenses/odbl/1.0/index.html).
>
> Ces données concernent exclusivement le RFN (Réseau Ferroviaire National français), dont SNCF Réseau a la charge.

# Contenu du dépôt

Ce dépôt contient à date :

N° | Nom | Description | GeoJSON | CSV
-- | --- | ----------- | ------- | ---
1 | **PKs** | Liste des points kilométriques avec leur position géographique, fichier reconstitué, extrapolé. | [GeoJSON](liste-des-pks.geojson.zip) | [CSV](liste-des-pks.csv.zip)
2 | **VITESSES** | Vitesses des lignes sous forme de vecteurs (`LineString`) | [GeoJSON](lignes-vitesses.geojson.zip) |
3 | **TUNNELS** | Tunnels sous forme de vecteurs (`LineString`) | [GeoJSON](lignes-tunnels.geojson.zip) |
4 | ~~**QUAIS**~~ | ~~Liste des quais au format geojson **en vecteurs** (transformation du fichier d'origine en points.~~ Les données sont pour l'instant inexploitables (cf détails). | [GeoJSON](liste-des-quais.linestrings.geojson.zip) |

# Détails pour chaque jeu de données

## 1. Liste des PKs

Données GeoJSON &amp; CSV des points kilométriques et hectométriques (PK) du réseau ferroviaire français, interpolées grâce aux données opendata de SNCF Réseau.

[Télécharger au format GeoJSON](liste-des-pks.csv.zip)

[Télécharger au format CSV](liste-des-pks.geojson.zip)

### Sources utilisées

Une source unique est le fichier de formes des lignes, en GeoJSON avec PK de début et PK de fin, qui contient l'ensemble des vecteurs (lignes droites) formant une ligne ferroviaire.

Ce fichier (version GeoJSON) contient la liste des coordonnées (latitude, longitude) de chaque vecteur qui forme la ligne (`LineString`) avec un troisième élément qui correspond au point kilométrique (en mètres) lié à cette position — que j'avais interprété comme étant une altitude (comme cela est logiquement prévu dans la norme GeoJSON).

Version du 22/05/2018 17:51 : https://data.sncf.com/explore/dataset/formes-des-lignes-du-rfn/table/

### Limites constatées

Les formes ne sont pas toujours précises quant à la réalité du terrain, de plus les points kilométriques ne sont parfois pas justes (relatif souvent à l'erreur de coordonnée) ce qui entraîne des décalages notamment en début de ligne.

Les données sont ainsi proposées comme un effort « au mieux », en attendant un fichier de formes corrigé.

### Méthode d'interpolation

1. ~~On transforme le fichier source des formes de ligne en une liste de PK connus, en prenant chaque vecteur à la suite avec ses coordonnées géographiques. Grâce au PK initial connu on calcule les suivants par la distance entre les coordonnées de chaque vecteur, et ainsi de suite.  
Par exemple, si le premier point du fichier de formes pour une ligne donnée est le PK 1.147, on calcule la distance en ligne droite jusqu'au point suivant du fichier pour cette ligne (fin du vecteur) qui est par exemple 820m. Le PK de ce point est donc 1.967, et ainsi de suite.~~  
Devenu obsolète : le fichier de forme contient déjà les PKs de chaque coordonnée stocké en mètres dans la valeur dédiée à l'altitude.  
Par exemple :  
```json
{
    "type": "FeatureCollection",
    "features": [
        {
            "type": "Feature",
            "geometry": {
                "type": "LineString",
                "coordinates": [
                    [
                        5.509821499334702,
                        43.68221085014309,
                        375989
                    ]
                ]
            }
        }
    ]
}                    
```
Ici, nous avons une coordonnée de longitude `5.509821499334702`, latitude `5.509821499334702`, avec comme point kilométrique 375+989 (ou 375,989km suivant la notation utilisée).

2. Devenu obsolète également.
~~Pendant cette phase, on tente de corriger les PK de début de vecteurs ainsi trouvés avec la position des éventuels objets positionnés sur le vecteur en question (donc entre les deux PKs) venant d'autres fichiers (gares et passages à niveau ici).~~
~~Pour cela, on calcule la différence de la distance entre~~
   - ~~le PK de début du vecteur et celui de l'objet contenu dans celui-ci~~
   - ~~la distance réelle entre les coordonnées géographiques de début du vecteur et celles de l'objet~~
~~On obtient ainsi **la valeur de correction**, qu'on soustrait à celle du PK de début avant le calcul du PK de fin de vecteur du point 1.~~  

3. On reparcourt l'ensemble des lignes et PKs ainsi trouvés et corrigés pour rechercher les PKs hectométriques par interpolation entre les début et fins de vecteur.  
Par exemple si le premier vecteur va du PK 1.147 au PK 1.967 les PKs nous intéressant sont les PK 1.2 1.3 1.4 1.5 1.6 1.7 1.8 et 1.9. On détermine alors l'orientation du vecteur grâce aux deux coordonnées de début et fin de celui-ci, puis la distance entre le PK de début et le premier PK intéressant (ici 53m pour aller de 1.147 à 1.200), et avec les coordonnées de départ, l'orientation et la distance, on obtient les coordonnées du PK qui nous intéresse.  
On recommence ensuite pour les autres PKs contenus dans l'intervalle.

4. On exporte le tout en GeoJSON dans un fichiers de points avec ligne, pk et coordonnées géographiques.

## 2. Vitesses des lignes, en vecteurs

Le problème du jeu de donnée fourni par SNCF Réseau est qu'il présente les vitesses sous forme de points (début et fin, voire seulement début dans le GeoJSON). Le fichier proposé ici est calqué sur le fichier de formes de lignes pour représenter les vitesses _avec_ la ligne sur laquelle elle est définie. Cela permet de visuellement représenter les vitesses sous forme de tronçons, par exemple colorés suivant la vitesse.

[Télécharger au format GeoJSON](lignes-vitesses.geojson.zip)

### Sources utilisées

Deux sources, le fichier contenant la liste des vitesses pour chaque tronçon, et le fichier contenu les formes des lignes.

Version du 24/07/2019 12:20 : https://data.sncf.com/explore/dataset/vitesse-maximale-nominale-sur-ligne/

Version du 22/05/2018 17:51 : https://data.sncf.com/explore/dataset/formes-des-lignes-du-rfn

## 3. Tunnels des lignes, en vecteurs

Le problème du jeu de donnée fourni par SNCF Réseau est qu'il présente les tunnels sous forme de points, avec le début (PK & coordonnées) et sa longueur. Le fichier proposé ici est calqué sur le fichier de formes de lignes pour représenter les tunnels le long de la ligne sur laquelle il est situé. Cela permet de visuellement représenter les tunnels sous forme de tronçons.

[Télécharger au format GeoJSON](lignes-tunnels.geojson.zip)

### Sources utilisées

Deux sources, le fichier contenant la liste des vitesses pour chaque tronçon, et le fichier contenu les formes des lignes.

Version du 24/07/2019 15:55 : https://data.sncf.com/explore/dataset/liste-des-tunnels

Version du 22/05/2018 17:51 : https://data.sncf.com/explore/dataset/formes-des-lignes-du-rfn

## 4. Liste des quais

> **Note de publication 01/09/2019** Les données sont hélas trop peu précises et parfois manquantes. Des quais se positionnent à plusieurs centaines de mètres de leur position réelle, voire se croisent. En attente d'un correctif.

Données GeoJSON des quais de gare avec leurs détails, sous forme de vecteurs représentant les quais. En effet, le fichier GeoJSON original propose des points seulement.

[Télécharger au format GeoJSON](liste-des-quais.linestrings.geojson.zip)

### Sources utilisées

Version du 24/07/2019 12:10 : https://data.sncf.com/explore/dataset/liste-des-quais

### Méthodologie

Le fichier d'origine contient deux champs de propriétés c_geo_d et c_geo_f qui représentent les coordonnées de départ et fin de chaque quai. Ces coordonnées sont replacées dans un objet `lineString`, les propriétés sont filtrées et corrigées (notamment les PK sont transformés en nombres flottants et arrondis à une décimale), amputées des éléments non nécessaires.
