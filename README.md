# Alset

Projet **Technologies et Langages du Web**, par [Maxime BATTU](https://maxime-battu.fr) et [Batiste LALOI](https://batiste-laloi.com) - CPE Lyon IRC - 2021/2022

## Vidéos de démonstration et d'explication

Démonstration de l'application : [Démonstration ALSET - CPE Lyon 3IRC](https://www.youtube.com/watch?v=pokF4w4si-I)

Explications plus détaillées du fonctionnement du site et des technologie/API utilisées : [Playlist de 3 vidéos explicatives](https://www.youtube.com/playlist?list=PLNbJfH5FAP7pEdBSmqXR-K1bbzsYh1BUT)

## Répartition du travail

[Batiste LALOI](https://batiste-laloi.com) : geocoding, calcul et affichage de l'itinéraire, calcul d'autonomie, déploiement du projet

[Maxime BATTU](https://maxime-battu.fr) : Affichage de la map, mise en place des clusters de markers, optimisation/revu de code, options de filtrage, style

Commun : Alimentation de l'interface utilisateur, descriptif de l'itinéraire

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

    * Affichage de la carte
      * paramétrer pour ne pas pouvoir aller au delà de la France métropolitaine 
      
    * Clustering des bornes sur une carte
      * 3 niveaux de clustering
         * faible 
         * moyen
         * large  
         
    * Informations sur la borne
      * Le nom
      * Son adresse
      * La ville ainsi que le code postal, si ces derniers sont renseignés 
      
    * Par défaut, affichage de toutes les bornes éléctriques présentent en France

* Options de filtre
   * Selection de la compatibiilité des bornes avec deux véhicules :
       * Véhicule 1 : Tesla Model 3
       * Véhicule 2 : Renault Zoé
       * Mise à jour des bornes de carte avec uniquement celles comptatibles avec le véhicule choisit

   * Paramétrer l'autonomie du véhicule
      * L'autonomie, par défaut, est de 200 km  
      * Véhicule 1 : Tesla Model 3, jusqu'à 495 km d'autonomie
      * Véhicule 2 : Renault Zoé, jusqu'à 290 km d'autonomie 
      * Si aucun véhicule n'est sélectionné, l'autonomie maximale sera de 500 km

* Possibilté de rentrer un lieu de départ et d'arrivée dans le menu déroulant en haut de la page
   
   * Mise en place d'un système d'auto-complétion
   * lieux uniquement présent en France métropolitaine

* Prise en compte des bornes de recharges en France dans l'itinéraire 

   * compatibles avec le véhicule sélectionné
   * si aucun véhicules n'est sélectionné, alors le chemin indiqué sera le plus rapide en utilisant toutes les bornes disponibles en France

* Affichage des étapes de la route dans le menu déroulant accessible en haut à gauche de la page

   * les routes principales (autoroutes) par lesquelles passées
   * les étapes du trajet
      * Un clique peut être effectué sur chaque étape pour en voir la durée estimée
      
   * la durée / heure d'arrivée
   * le nombre de km / l'autonomie finale 

* Choix du thème de la carte en haut à droite de la page

    * Choix entre : Clair 💡 / Sombre 🌑
    * Le thème au chargement du site dépend du thème actuelle du naviguateur de l'utilisateur


## Listes des modules npm utilisés :

* Leaflet
    * leaflet.markercluster
        * Gestion des clusters de marqueurs liés aux bornes de charge
        
    * leaflet-edgebuffer
        * Permet de pré-charger des éléments en dehors de la vue actuelle
        
    * leaflet-routing-machine
        * Permet de calculer le chemin le plus court entre deux points
* lodash
   * Permet d'utiliser une méthode pour "debounce" les fonctions et avoir la main sur leurs appels
   * On a notamment utilisé sur l'affichage des autocomplétions d'adresses
      * un nouveau tableau d'adresse prévisionnelle était envoyé toutes les 50ms pour ne pas saturer les envoies entre les components
      * Si on ne mettais pas en place cette méthode là, le tableau aurait été envoyé à chaque fois qu'un caractère aurait été tapé, sur une petite application comme la notre le soucis est moindre mais si on se place à plus grande échelle cela peut devenir plus embettant et créer des problèmes de performances
      
* vue-simple-suggest
    * Permet de créer des suggestions de recherche lors de la saisie du départ / arrivée

## Liste des API utilisées :

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
