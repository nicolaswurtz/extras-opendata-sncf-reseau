> **Avertissement !**
>
> Ces données sont publiées _en l'état_ sans garantie de contenu ni de suivi, à utiliser à vos risques et périls. Elles sont intégralement fabriquées, transformées, à partir des jeux de données placés en OpenData entre autres par SNCF et SNCF Réseau (cf https://data.sncf.com) et sous [licence ODbL](https://opendatacommons.org/licenses/odbl/1.0/index.html), ainsi que des données Wikipédia/Wikidata, des Chemins de Fer Corse et de [Trainline](https://github.com/trainline-eu/stations/blob/master/stations.csv).
>
> Ces données concernent exclusivement les voies ferrées situées en France.
>
> Elles sont essentiellement utilisées pour mon projet **personnel** de cartographie du réseau ferré français et du positionnement des trains en temps réel https://carto.graou.info

# Contenu du dépôt

Ce dépôt contient à date :

N° | Nom | Description | GeoJSON | CSV
-- | --- | ----------- | ------- | ---
1 | **PKs** | Liste des points kilométriques à la ligne, avec leur position géographique, leurs altitudes et la vitesse max de la ligne à cet endroit, fichier reconstitué, extrapolé. | | [CSV](pks.csv.zip) |
2 | **VITESSES** | Vitesses des lignes sous forme de vecteurs (`LineString`) | [GeoJSON](lignes-vitesses.geojson.zip) |
3 | **TUNNELS** | Tunnels sous forme de vecteurs (`LineString`) | [GeoJSON](lignes-tunnels.geojson.zip) |
3 | **GARES** | Gares et points remarquables | [CSV](localites.csv.zip) | [GeoJSON](localites.geojson.zip) |

# Détails pour chaque jeu de données

## 1. Liste des PKs

Données CSV des points kilométriques et hectométriques (PK) du réseau ferroviaire français, interpolées grâce aux données opendata de SNCF Réseau.

[Télécharger au format CSV](pks.csv.zip)

### Sources et méthodes

#### PK
Une source unique est le fichier de formes des lignes, en GeoJSON avec PK de début et PK de fin, qui contient l'ensemble des vecteurs (lignes droites) formant une ligne ferroviaire. Ce fichier (version GeoJSON), qui n'existe plus aujourd'hui avec les PK, contenait la liste des coordonnées (latitude, longitude) de chaque vecteur qui forme la ligne (`LineString`) avec un troisième élément qui correspond au point kilométrique (en mètres) lié à cette position — que j'avais interprété comme étant une altitude (comme cela est logiquement prévu dans la norme GeoJSON).
Cette donnée n'est plus disponible dans https://data.sncf.com/explore/dataset/formes-des-lignes-du-rfn/table/

Le fichier de formes des lignes de l'opendata contenait les PK de chaque point (ce n'est plus le cas désormais).

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

On reparcourt l'ensemble des lignes et PKs ainsi trouvés et corrigés pour rechercher les PKs hectométriques par interpolation entre les début et fins de vecteur.  
Par exemple si le premier vecteur va du PK 1.147 au PK 1.967 les PKs nous intéressant sont les PK 1.2 1.3 1.4 1.5 1.6 1.7 1.8 et 1.9. On détermine alors l'orientation du vecteur grâce aux deux coordonnées de début et fin de celui-ci, puis la distance entre le PK de début et le premier PK intéressant (ici 53m pour aller de 1.147 à 1.200), et avec les coordonnées de départ, l'orientation et la distance, on obtient les coordonnées du PK qui nous intéresse.  

#### Vitesses
Pour chaque point hectométrique, on associe la vitesse de la ligne à cet endroit, grâce aux données de https://data.sncf.com/explore/dataset/vitesse-maximale-nominale-sur-ligne/

#### Altimétrie
Trois colonnes sont proposées :

1. Altitude topographique
Grâce à https://www.opentopodata.org et les données EU-DEM de L'European Space Agency https://www.eea.europa.eu/data-and-maps/data/copernicus-land-monitoring-service-eu-dem on peut monter un serveur qui renvoie une altitude à partir du couple latitude/longitude ; on peut envoyer des coordonnées en batch, pour les 366000 points hectométriques du réseau ferroviaire, comptez une heure et demi sur une machine raisonnablement puissante.

2. Altitude topographique, corrigée par les tunnels
Même jeu de données que précédement mais où les zone de tunnels ont été linéarisées entre l'entrée et la sortie pour éviter de tracer l'altitude de la montagne au dessus du tunnel, grâce aux données de https://ressources.data.sncf.com/explore/dataset/liste-des-tunnels

3. Altitude reconstituée avec les déclivités
SNCF Réseau distribue les déclivités en opendata https://ressources.data.sncf.com/explore/dataset/caracteristique-des-voies-et-declivite/information/ à la voie. Ce qui est un peu compliqué puisque les PKs sont à la ligne. J'ai dû ruser pour obtenir les déclivités successives à la ligne.  
Ensuite, j'ai redistribué les déclivités par pas de 100m histoire de coller à mes PKs hectométriques, puis j'ai pris l'altitude du premier point de chaque ligne, altitude trouvée pendant le point 1 et j'ai appliqué les coefficients de rampes et de pente au fil des points.

### Limites constatées

- Les formes ne sont pas toujours précises quant à la réalité du terrain, de plus les points kilométriques ne sont parfois pas justes (relatif souvent à l'erreur de coordonnée) ce qui entraîne des décalages notamment en début de ligne. Les données sont ainsi proposées comme un effort « au mieux », en attendant un fichier de formes corrigé.
- On constate des écarts au niveau de l'altimétrie notamment à haute altitude. Écarts qu'on retrouve parfois sur les lignes, mais pas systématiquement, ce qui ne permet de définir si l'erreur vient de mon algo ou des données sources.


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

## 4. Gares, points remarquables

Il s'agît d'un fichier que je tiens à jour depuis 10 ans, agrégeant un maximum de données opendata, qu'elles viennent de SNCF, SNCF Réseau, Wikipedia/wikidata, Chemins de Fer Corse, ou même Trainline.
Il se veut le plus complet possible mais il n'est pas parfait, n'hésitez pas à proposer des corrections.

[Télécharger au format CSV](localites.csv.zip) [Télécharger au format GeoJSON](localites.geojson.zip)

### Anomalies existantes
- des points avec comme nom de chantier « JON » sont totalement fictifs, ils me servent pour faire les jonctions entre lignes là où il n'y a pas de gare pour la création d'itinéraires topologiques
- des gares n'avaient pas de trigramme, notamment en Corse, donc j'ai dû... inventer
- idem pour certains numéro UIC

