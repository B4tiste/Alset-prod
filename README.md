# Dépot clone de déploiement de l'application Alset via Netlify

# alset

Projet **Technologies et Langages du Web**, par [Maxime BATTU](https://maxime-battu.fr) et [Batiste LALOI](https://batiste-laloi.com).

## Technologies utilisées

* VueJS 
* NodeJS
* HTML/CSS

## Récupération

Dupliquer le dépôt git :

```
git clone https://github.com/cpe-lyon/groupe-3-alset.git
```

## Setup du projet

### **⚠ Disclaimer ⚠**

Notre projet, utilisant des API, possède un token d'accès qui est gardé secret dans une variable d'envrionnement locale à nos machines. De ce fait, la compilation du projet sera fonctionnel mais pas son utilisation. 

Pour l'utiliser et voir le produit final, rendez-vous à cette addresse de déploiement de notre application, hébérgée par le biais d'un [dépot clone](https://github.com/B4tiste/Alset-prod) : [Alset](https://alset.netlify.app/)

Liste des fonctionnalités :

* Affichage des bornes de recharge électrique en France

    * Clustering des bornes sur une carte
    * Informations sur la borne
    * FIltrage des bornes en fonction du véhicule

* Selection de la compatibiilité des bornes avec deux véhicules :

    * Véhicule 1 : Tesla Model 3
    * Véhicule 2 : Renault Zoé

* Possibilté de rentrer un lieu de départ et d'arrivée dans le menu déroulant en haut de la page

* Prise en compte des bornes de recharges en France compatibles avec le véhicule sélectionné

* Affichage des étapes de la route dans le menu déroulant accessible en haut à gauche de la page, avec de plus amples informations sur le trajet

* Choix du thème de la carte

    * Clair 💡 / Sombre 🌑


Listes des modules npm utilisés :

* Leaflet
    * leaflet.markercluster
        * Gestion des clusters de marqueurs liés aux bornes de charge
    * leaflet-edgebuffer
        * Permet de pré-charger des éléments en dehors de la vue actuelle
    * leaflet-routing-machine
        * Permet de calculer le chemin le plus court entre deux points

* vue-simple-suggest
    * Permet de créer des suggestions de recherche lors de la saisie du départ / arrivée

Liste des API utilisées :

* MapBox
    * Récupération de la carte
    * Géocoding
* OpenChargeMap
    * Récupération des stations
    * Récupération des informations sur les stations
* Google Fonts
    * Récupération des polices


### Installer les modules

```
npm install
```