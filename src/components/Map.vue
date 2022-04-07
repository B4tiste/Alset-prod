<template>
    <div>
        <div class="page-loading" v-if="loading">
            <div class="loader"></div>
        </div>

        <div id="map" />

        <div v-on:click="setDefaultLocalisation" class="localisation">
            <div class="icon">
                <img
                    src="../assets/images/localisation.png"
                    alt="localisation"
                />
            </div>
        </div>

        <div class="options" v-on:click="showOptionsContainer">
            <div class="icon">
                <img src="../assets/images/options.png" alt="options" />
            </div>
        </div>

        <div class="options-container">
            <div class="container">
                <div class="selecteurs">
                    <div class="range-selector">
                        <label for="batterie"
                            >Autonomie de
                            {{ autonomieMaxEnMetres / 1000 }} km</label
                        >
                        <input
                            v-model="autonomieMaxEnMetres"
                            type="range"
                            id="batterie"
                            name="batterie"
                            min="0"
                            max="500000"
                            step="1000"
                        />
                    </div>
                    <div class="filter-vehicules">
                        <div class="voitures-selector">
                            <input
                                v-model="picked"
                                type="radio"
                                id="tesla"
                                name="tesla"
                                value="superChargeur"
                            />
                            <label for="tesla">Tesla</label>
                        </div>
                        <div class="voitures-selector">
                            <input
                                v-model="picked"
                                type="radio"
                                id="tesla"
                                name="tesla"
                                value="type2"
                            />
                            <label for="tesla">Zoé</label>
                        </div>
                        <div class="voitures-selector">
                            <input
                                v-model="picked"
                                type="radio"
                                id="aucun"
                                name="aucun"
                                value="aucun"
                            />
                            <label for="aucun">Aucun</label>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import L /*, { marker }*/ from "leaflet";
import "leaflet.markercluster";
import "leaflet-edgebuffer";
import "leaflet-routing-machine";

/**
 * @typedef Borne
 * @property {Object} AddressInfo
 * @property {number} AddressInfo.Latitude
 * @property {number} AddressInfo.Longitude
 */

/**
 * @typedef Waypoint
 * @property {String} id
 * @property {number} lat
 * @property {number} lon
 * @property {Date} date
 */

export default {
    props: {
        coordonneesItineraire: {
            type: Object,
        },
    },
    data() {
        return {
            /** @type Borne[] */
            bornes: [],
            /** @type Borne[] */
            bornesOnMap: [],
            /** @type Waypoint[] */
            waypoints: [],
            /** @type L.map */
            map: null,
            /** @type L.Routing.Control */
            routingLayer: null,
            autonomieMaxEnMetres: 350_000,
            tailleIcone: null,
            zoom: 6,
            /** @type number[] */
            centre: [46.514211736281354, 2.576609403696891],
            icons: {
                blackIcon: L.icon({
                    iconUrl:
                        "https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-black.png",
                    shadowUrl: "",
                    shadowSize: [0, 0],
                    iconSize: this.tailleIcone,
                }),
                redIcon: L.icon({
                    iconUrl:
                        "https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-red.png",
                    shadowUrl: "",
                    shadowSize: [0, 0],
                    iconSize: this.tailleIcone,
                }),
                blueIcon: L.icon({
                    iconUrl:
                        "https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-blue.png",
                    shadowUrl: "",
                    shadowSize: [0, 0],
                    iconSize: this.tailleIcone,
                }),
                purpleIcon: L.icon({
                    iconUrl:
                        "https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-violet.png",
                    shadowUrl: "",
                    shadowSize: [0, 0],
                    iconSize: this.tailleIcone,
                }),
            },
            connexionsCompatible: {
                type2: {
                    connexions: [25, 1036],
                },
                superChargeur: {
                    connexions: [27],
                },
            },
            /** @type L.markerClusterGroup */
            clusterGroup: null,
            mapFiltree: false,
            picked: null,
            loading: true,
        };
    },
    beforeCreate() {
        this.tailleIcone = [25, 41];
    },
    async mounted() {
        this.initGeolocalisation();
        this.initMap();
        this.initRoutingLayer();

        /** chargement asynchrone des bornes : non bloquant */
        await this.chargerBornes();

        this.clusterGroup = L.markerClusterGroup({
            chunkedLoading: true,
        });

        /** markers */
        const markers = this.bornes.map((borne) => {
            return L.marker(
                [borne.AddressInfo.Latitude, borne.AddressInfo.Longitude],
                {
                    icon: this.icons.purpleIcon,
                }
            ).bindTooltip(this.getTooltipHtml(borne));
        });

        /** ajouter TOUS les markers au cluster */
        this.clusterGroup.addLayers(markers);

        /** ajout du cluster à la map */
        this.map.addLayer(this.clusterGroup);

        setTimeout(() => {
            this.loading = false;
        }, 2000);
    },
    methods: {
        initGeolocalisation() {
            if (navigator.geolocation) {
                navigator.geolocation.getCurrentPosition(
                    (success) => {
                        this.centre = [
                            success.coords.latitude,
                            success.coords.longitude,
                        ];
                        this.zoom = 11;
                        const positionActuelle = L.marker(this.centre, {
                            icon: this.icons.blackIcon,
                        }).bindTooltip(
                            `<p class="chargeur-titre">Vous êtes ici</p>`
                        );
                        positionActuelle.addTo(this.map);

                        this.map.setView(this.centre, this.zoom);
                    },
                    (error) => {
                        console.warn(error.message);
                        this.centre = [46.514211736281354, 2.576609403696891];
                        this.zoom = 6;
                        this.map.setView(this.centre, this.zoom);
                    }
                );
            }
        },
        verifierTypeConnexions(idConnexion) {
            return (
                idConnexion !== null &&
                (this.connexionsCompatible.type2.connexions.includes(
                    idConnexion
                ) ||
                    this.connexionsCompatible.superChargeur.connexions.includes(
                        idConnexion
                    ))
            );
        },
        initMap() {
            const credits =
                /* html */
                `Alset |
                <a href="https://batiste-laloi.com" target="_blank">Batiste Laloi</a>
                & 
                <a href="https://maxime-battu.fr" target="_blank">Maxime Battu</a>`;

            this.map = L.map("map", {
                preferCanvas: true,
                zoomControl: false,
            }).setView(this.centre, this.zoom);

            const lightLayer = L.tileLayer(
                "https://api.mapbox.com/styles/v1/mapbox/streets-v11/tiles/{z}/{x}/{y}?access_token=" +
                    process.env.VUE_APP_MAPBOX_ACCESS_TOKEN,
                {
                    attribution: credits,
                    maxZoom: 18,
                    id: "mapbox/streets-v11",
                    tileSize: 512,
                    zoomOffset: -1,
                    edgeBufferTiles: 2,
                    visible: true,
                }
            ).addTo(this.map);

            const darkLayer = L.tileLayer(
                "https://api.mapbox.com/styles/v1/mapbox/dark-v10/tiles/{z}/{x}/{y}?access_token=" +
                    process.env.VUE_APP_MAPBOX_ACCESS_TOKEN,
                {
                    attribution: credits,
                    maxZoom: 18,
                    id: "mapbox/streets-v11",
                    tileSize: 512,
                    zoomOffset: -1,
                    edgeBufferTiles: 2,
                    visible: false,
                }
            );

            L.control
                .layers({
                    "🌑 Dark theme": darkLayer,
                    "💡 Light theme": lightLayer,
                })
                .addTo(this.map);

            L.control
                .scale({
                    imperial: false,
                    maxWidth: 200,
                    metric: true,
                    position: "bottomleft",
                })
                .addTo(this.map);
        },
        initRoutingLayer() {
            this.routingLayer = L.Routing.control({
                fitSelectedRoutes: "smart",
                autoRoute: true,
                routeWhileDragging: false,
                routeDragInterval: 500,
                draggableWaypoints: false,
                waypointMode: "connect",
                useZoomParameter: false,
                showAlternatives: false,
                addWaypoints: false,
                allowUTurn: true,
                lineOptions: {
                    styles: [
                        { color: "black", opacity: 0.8, weight: 2 },
                        { color: "white", opacity: 0.8, weight: 6 },
                        { color: "red", opacity: 0.9, weight: 7 },
                    ],
                    missingRouteTolerance: 0,
                    extendToWaypoints: false,
                },
                altLineOptions: {
                    styles: [
                        { color: "black", opacity: 0.8, weight: 2 },
                        { color: "white", opacity: 0.8, weight: 6 },
                        { color: "darkgrey", opacity: 1, weight: 7 },
                    ],
                    missingRouteTolerance: 0,
                    extendToWaypoints: false,
                },
                show: true,
                language: "fr",
            }).addTo(this.map);

            /** On écoute sur l'évenement routeselected puis on appelle une méthode */
            this.routingLayer.on("routeselected", this.onRouteSelectedEvent);
        },
        /**
         * @param {L.Routing.RouteSelectedEvent} e
         */
        onRouteSelectedEvent(e) {
            const instructions = e.route.instructions.map(
                (instruction, index) => {
                    return {
                        id: index,
                        text: instruction.text,
                        time: instruction.time,
                        type: instruction.type,
                    };
                }
            );
            const totalDistance = e.route.summary.totalDistance;
            const totalTime = e.route.summary.totalTime;
            const name = e.route.name;

            /** tableau des recharges par lesquelles passées */
            const recharges = [];

            let autonomie = this.autonomieMaxEnMetres; // mètres
            let distanceParcourue = 0;
            let i = 0;
            while (distanceParcourue < e.route.summary.totalDistance) {
                distanceParcourue += e.route.instructions[i].distance;
                i++;

                if (i >= e.route.instructions.length) {
                    break;
                }

                let nombrePause = 0;

                // Cas dans lequel on n'a plus de batterie si on suit la prochaine étape
                // Il faut donc ajouter ce point aux points de recherche de bornes de recharge
                if (autonomie - e.route.instructions[i].distance < 0) {
                    nombrePause++;
                    recharges.push({
                        date: new Date().toLocaleString(),
                        lat: e.route.coordinates[e.route.instructions[i].index]
                            .lat,
                        lon: e.route.coordinates[e.route.instructions[i].index]
                            .lng,
                        id: "recharge #" + nombrePause,
                        autonomie_restante: autonomie,
                    });
                    autonomie = this.autonomieMaxEnMetres;
                }
                autonomie -= e.route.instructions[i].distance;
            }

            autonomie = Math.round(this.autonomieMaxEnMetres / 1000);

            this.$emit("update:routeSwitch", {
                instructions,
                recharges,
                totalDistance,
                totalTime,
                name,
                autonomie,
            });

            //Trouver les points les plus proches des éléments du tableau recharges parmis les markers présents sur la carte
            const bornesToUse = [];

            for (let i = 0; i < recharges.length; i++) {
                let distanceEntreBornes = 100_000_000;

                for (let j = 0; j < this.bornesOnMap.length; j++) {
                    let B = this.bornesOnMap[j];

                    const d = this.calculerDistance(
                        recharges[i].lat,
                        recharges[i].lon,
                        B.AddressInfo.Latitude,
                        B.AddressInfo.Longitude
                    );

                    if (d < distanceEntreBornes) {
                        distanceEntreBornes = d;

                        bornesToUse[i] = {
                            id: "Recharge #" + (i + 1),
                            lat: B.AddressInfo.Latitude,
                            lon: B.AddressInfo.Longitude,
                            distance: distanceEntreBornes,
                        };
                    }
                }
            }

            /** On définit les points de départ, d'arrivée et par lesquels on passe */
            const depart = this.waypoints[0];
            const arrivee = this.waypoints[1];

            this.waypoints = [depart, ...bornesToUse, arrivee];

            if (e.route.waypoints.length == 2) {
                this.routingLayer.setWaypoints(this.waypoints);
                this.routingLayer.route();
            }
        },
        loadSpecifiBornes(picked) {
            if (
                this.bornes.length === this.bornesOnMap.length &&
                picked === "aucun"
            ) {
                return;
            }
            /** initiation des cluster */
            this.clusterGroup.clearLayers();
            const markers = [];

            // Reset du tableau des bornes présentes sur la map actuellement
            this.bornesOnMap = [];

            /** markers */
            if (picked === "aucun") {
                this.bornes.forEach((borne) => {
                    markers.push(
                        L.marker(
                            [
                                borne.AddressInfo.Latitude,
                                borne.AddressInfo.Longitude,
                            ],
                            {
                                icon: this.icons.purpleIcon,
                            }
                        ).bindTooltip(this.getTooltipHtml(borne))
                    );

                    this.bornesOnMap.push(borne);
                });

                this.clusterGroup.addLayers(markers);
            } else if (picked === "type2") {
                this.bornes.forEach((borne) => {
                    let isConnectable = false;
                    isConnectable = borne.Connections.some((connection) => {
                        if (
                            this.connexionsCompatible.type2.connexions.includes(
                                connection.ConnectionTypeID
                            )
                        ) {
                            return true;
                        }
                    });

                    if (isConnectable) {
                        markers.push(
                            L.marker(
                                [
                                    borne.AddressInfo.Latitude,
                                    borne.AddressInfo.Longitude,
                                ],
                                {
                                    icon: this.icons.purpleIcon,
                                }
                            ).bindTooltip(this.getTooltipHtml(borne))
                        );

                        this.bornesOnMap.push(borne);
                    }
                });

                this.clusterGroup.addLayers(markers);
            } else if (picked === "superChargeur") {
                this.bornes.forEach((borne) => {
                    let isConnectable = false;
                    isConnectable = borne.Connections.some((connection) => {
                        if (
                            this.connexionsCompatible.superChargeur.connexions.includes(
                                connection.ConnectionTypeID
                            )
                        ) {
                            return true;
                        }
                    });

                    if (isConnectable) {
                        markers.push(
                            L.marker(
                                [
                                    borne.AddressInfo.Latitude,
                                    borne.AddressInfo.Longitude,
                                ],
                                {
                                    icon: this.icons.purpleIcon,
                                }
                            ).bindTooltip(this.getTooltipHtml(borne))
                        );

                        this.bornesOnMap.push(borne);
                    }
                });

                this.clusterGroup.addLayers(markers);
            }

            /** ajout du cluster à la map */
            this.map.addLayer(this.clusterGroup);
        },
        /**
         * @param lat1 {number}
         * @param lon1 {number}
         * @param lat2 {number}
         * @param lon2 {number}
         */
        calculerDistance(lat1, lon1, lat2, lon2) {
            let REnKM = 6371;
            let dLat = ((lat2 - lat1) * Math.PI) / 180;
            let dLon = ((lon2 - lon1) * Math.PI) / 180;
            lat1 = (lat1 * Math.PI) / 180;
            lat2 = (lat2 * Math.PI) / 180;

            let a =
                Math.sin(dLat / 2) * Math.sin(dLat / 2) +
                Math.sin(dLon / 2) *
                    Math.sin(dLon / 2) *
                    Math.cos(lat1) *
                    Math.cos(lat2);

            let c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
            let d = REnKM * c;

            return d;
        },
        async chargerBornes() {
            const response = await fetch("/bornesChargement.json");
            const bornes = await response.json();
            this.bornes = bornes.map((borne) => {
                return {
                    AddressInfo: borne.AddressInfo,
                    Connections: borne.Connections.map((connection) => {
                        return {
                            ConnectionTypeID: connection.ConnectionTypeID,
                        };
                    }),
                };
            });

            this.bornesOnMap = this.bornes;
        },
        getTooltipHtml(borne) {
            let extraInfo = "";
            if (
                borne.AddressInfo.Town !== null &&
                borne.AddressInfo.Postcode !== null
            ) {
                extraInfo = /* html */ `
                <p class="chargeur-complement">
                    ${borne.AddressInfo.Town},
                    ${borne.AddressInfo.Postcode}
                </p>`;
            }

            return /* html */ `
                <div class="chargeur-informations">
                    <p class="chargeur-titre">
                        ${borne.AddressInfo.Title}
                    </p>
                    <p class="chargeur-adresse">
                        ${borne.AddressInfo.AddressLine1}
                    </p>
                    ${extraInfo}
                </div>`;
        },
        setDefaultLocalisation() {
            this.map.setView(this.centre, this.zoom);
        },
        showOptionsContainer() {
            let options = document.querySelector(".options-container");
            options.classList.toggle("active");
        },
    },
    watch: {
        coordonneesItineraire() {
            const date = new Date();

            this.waypoints = [];
            this.waypoints.push(
                {
                    id: "depart",
                    lat: this.coordonneesItineraire.depart.coords.lat,
                    lon: this.coordonneesItineraire.depart.coords.lon,
                    date: date.toLocaleString(),
                },
                {
                    id: "destination",
                    lat: this.coordonneesItineraire.arrivee.coords.lat,
                    lon: this.coordonneesItineraire.arrivee.coords.lon,
                    date: date.toLocaleString(),
                }
            );
            this.routingLayer.setWaypoints(this.waypoints);

            const corner1 = L.latLng(
                this.waypoints[0].lat,
                this.waypoints[0].lon
            );
            const corner2 = L.latLng(
                this.waypoints[1].lat,
                this.waypoints[1].lon
            );

            const bounds = L.latLngBounds(corner1, corner2);
            this.map.fitBounds(bounds, {
                padding: [80, 80],
            });
        },
        picked(newValue) {
            this.picked = newValue;

            this.loadSpecifiBornes(this.picked);

            console.log(this.departAdresse);

            // On recreer l'itinéraire
            if (this.departAdresse !== null && this.arriveeAdresse !== null) {
                const date = new Date();

                this.waypoints = [];
                this.waypoints.push(
                    {
                        id: "depart",
                        lat: this.coordonneesItineraire.depart.coords.lat,
                        lon: this.coordonneesItineraire.depart.coords.lon,
                        date: date.toLocaleString(),
                    },
                    {
                        id: "destination",
                        lat: this.coordonneesItineraire.arrivee.coords.lat,
                        lon: this.coordonneesItineraire.arrivee.coords.lon,
                        date: date.toLocaleString(),
                    }
                );
                this.routingLayer.setWaypoints(this.waypoints);
            }
        },
        autonomieMaxEnMetres(newAutonomie) {
            this.autonomieMaxEnMetres = newAutonomie;
        },
    },
};
</script>

<style scoped>
#map {
    height: 100vh;
    z-index: 1;
}

.icon {
    width: 30px;
    height: 30px;
}

.localisation,
.options {
    display: flex;
    position: absolute;
    justify-content: center;
    align-items: center;
    width: 64px;
    height: 64px;
    border-radius: 50px;
    background-color: white;
    z-index: 15;
    cursor: pointer;
    transition: 0.3s background-color;
    right: 15px;
    box-shadow: 0px 0px 8px 0px var(--manatee);
}

.localisation {
    bottom: 20px;
}

.localisation:hover,
.options:hover {
    background-color: #e0e0e0;
    transition: 0.3s background-color;
}

.options {
    bottom: 90px;
}

.options-container {
    display: none;
    justify-content: center;
    align-items: center;
    position: absolute;
    right: 15px;
    bottom: 90px;
    height: 64px;
    min-width: 600px;
    width: 0px;
    z-index: 10;
    animation: reduceWidth 0.5s;
}

.options-container.active {
    display: flex;
    width: 450px;
    animation: growWidth 0.4s;
}

.container {
    display: flex;
    justify-content: flex-start;
    align-items: center;
    height: 100%;
    width: 100%;
    background: white;
    border-radius: 50px;
    box-shadow: 0px 0px 8px 0px var(--manatee);
}

.selecteurs {
    display: flex;
    justify-content: space-around;
    align-items: center;
    width: 80%;
}

.range-selector {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    width: 50%;
    color: var(--magenta);
    letter-spacing: 1px;
}

.filter-vehicules {
    display: flex;
    justify-content: space-around;
    flex-direction: row;
    width: 50%;
}

.voitures-selector label {
    padding: 0 5px;
    color: var(--magenta);
    letter-spacing: 1px;
}

.page-loading {
    display: flex;
    justify-content: center;
    align-items: center;
    position: absolute;
    height: 100vh;
    width: 100vw;
    top: 0;
    left: 0;
    background: rgba(0, 0, 0, 0.9);
    z-index: 1000;
    transition: 5s;
    font-size: 36px;
    color: white;
}

.loader {
    display: block;
    position: absolute;
    width: 150px;
    height: 150px;
    border-left: 3px solid rgb(83, 83, 199);
    border-radius: 100px;
    animation: spin 2s linear infinite;
}

.loader:after {
    content: "";
    position: absolute;
    top: 12px;
    left: 12px;
    right: 12px;
    bottom: 12px;
    border-radius: 100px;
    border-top: 3px solid rgb(174, 77, 253);
}

.loader:before {
    content: "";
    position: absolute;
    top: 3px;
    left: 3px;
    right: 3px;
    bottom: 3px;
    border-radius: 100px;
    border-right: 3px solid var(--magenta);
}

@keyframes spin {
    from {
        transform: rotate(0deg);
    }
    to {
        transform: rotate(360deg);
    }
}

@keyframes growWidth {
    0% {
        min-width: 480px;
    }
    100% {
        min-width: 600px;
    }
}

@keyframes reduceWidth {
    0% {
        min-width: 500px;
        display: flex;
    }
    100% {
        min-width: 100px;
        display: none;
    }
}
</style>

<style>
.leaflet-routing-container {
    visibility: hidden; /* Cacher le carré vide de la liste de étapes de l'itinéraire */
}

.marker-cluster-small {
    background-color: rgba(38, 201, 129, 0.6) !important;
}

.marker-cluster-small div {
    background-color: #1ba367 !important;
    color: #fff;
}

.marker-cluster-medium {
    background-color: rgba(246, 180, 59, 0.6) !important;
}

.marker-cluster-medium div {
    background-color: #de9000 !important;
    color: #fff;
}

.marker-cluster-large {
    background-color: rgba(126, 0, 230, 0.6) !important;
}

.marker-cluster-large div {
    background-color: var(--magenta) !important;
    color: #fff;
    font-weight: bold !important;
}

.tooltip {
    padding: 0;
    color: none;
    border: none;
    border-radius: 25px;
    background-color: red;
}

.chargeur-titre {
    font-family: "Roboto", sans-serif;
    font-size: 1.1rem;
    font-weight: 900;
    color: var(--indigo);
}

.chargeur-adresse,
.chargeur-complement {
    font-family: "Montserrat", sans-serif;
    font-size: 0.9rem;
    font-weight: 500;
}
</style>