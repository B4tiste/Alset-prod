<template>
    <div>
        <l-map
            class="map"
            :center="centre"
            :zoom="zoom"
            :options="{ zoomControl: false }"
            ref="map"
            @update:zoom="zoomUpdated"
            @update:center="centerUpdated"
        >
            <!-- Placement du selecteur de map en haut à droite -->
            <l-control-layers position="topright"></l-control-layers>

            <l-tile-layer
                v-for="tileProvider in tileProviders"
                :key="tileProvider.name"
                :name="tileProvider.name"
                :visible="tileProvider.visible"
                :url="tileProvider.url"
                :attribution="tileProvider.attribution"
                layer-type="base"
            />

            <!-- Placement du controlleur de zoom en bas à droite -->
            <!-- Eviter les conflits avec le bouton d'affichage du descriptif de l'itinéraire -->
            <l-control-zoom position="bottomright"></l-control-zoom>

            <l-routing-machine
                :waypoints="waypoints"
                :depart-info="departInfo"
                :arrivee-info="arriveeInfo"
                @update:routeSwitch="onRouteSwitchChange"
            />

            <!-- Bornes de recharge -->
            <l-marker-cluster>
                <l-marker
                    v-for="data in bornesData"
                    v-bind:key="data.id"
                    :lat-lng="[
                        data.AddressInfo.Latitude,
                        data.AddressInfo.Longitude,
                    ]"
                    :icon="icons.greyIcon"
                >
                    <l-tooltip>
                        <div class="chargeur-informations">
                            <p class="chargeur-titre">
                                {{ data.AddressInfo.Title }}
                            </p>
                            <p class="chargeur-adresse">
                                {{ data.AddressInfo.AddressLine1 }}
                            </p>
                            <p
                                v-if="
                                    data.AddressInfo.Town !== null &&
                                    data.AddressInfo.Postcode !== null
                                "
                                class="chargeur-complement"
                            >
                                {{ data.AddressInfo.Town }},
                                {{ data.AddressInfo.Postcode }}
                            </p>
                        </div>
                    </l-tooltip>
                </l-marker>
            </l-marker-cluster>
        </l-map>
    </div>
</template>

<script>
// Import des components
import {
    LMap,
    LTileLayer,
    LMarker,
    LTooltip,
    LControlLayers,
    LControlZoom,
} from "vue2-leaflet";
import Vue2LeafletMarkerCluster from "vue2-leaflet-markercluster";

// Import du component de routing
import LRoutingMachine from "../components/LRoutingMachine";

// Code de telechargement des icones de markers
import { icon } from "leaflet";

// Import du JSON de la BDD des bornes de recharge
import BORNES_COORDS from "../../data/bornesChargement.json";

// URL des maps :
const url_light_theme = "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png";
const url_dark_theme =
    "https://tiles.stadiamaps.com/tiles/alidade_smooth_dark/{z}/{x}/{y}{r}.png";
const url_sat =
    "https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}";

let url = "";

const credits = "Alset | " + '<a href="https://batiste-laloi.com" target="_blank">Batiste Laloi</a>' + " & " + '<a href="https://maxime-battu.fr" target="_blank">Maxime Battu</a>';

//TODO est-ce qu'on peut laisser des const ici ? ou vaut-il mieux les créer dans le beforceCreate() ?
// Centré sur CPE
// const centreParDefaut = [45.78433216234776, 4.869708582722543];
// const tailleIcone = [25, 41];

export default {
    props: {
        departInfo: {
            type: Object,
        },
        arriveeInfo: {
            type: Object,
        },
    },
    components: {
        LMap,
        LTileLayer,
        LMarker,
        LTooltip,
        LRoutingMachine,
        "l-marker-cluster": Vue2LeafletMarkerCluster,
        LControlLayers,
        LControlZoom,
    },
    data() {
        return {
            // Map
            url: url,
            tileProviders: [
                {
                    name: "💡 Light theme",
                    visible: true,
                    attribution: credits,
                    url: url_light_theme,
                },
                {
                    name: "🌑 Dark theme",
                    visible: false,
                    attribution: credits,
                    url: url_dark_theme,
                },
                {
                    name: "🌍 Satellite",
                    visible: false,
                    attribution: credits,
                    url: url_sat,
                },
            ],
            centre: this.centreParDefaut,
            zoom: 11,
            waypoints: [],
            w_recharges: [],
            icons: {
                greyIcon: icon({
                    iconUrl:
                        "https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-grey.png",
                    shadowUrl: "",
                    shadowSize: [0, 0],
                    iconSize: this.tailleIcone,
                }),
                redIcon: icon({
                    iconUrl:
                        "https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-red.png",
                    shadowUrl: "",
                    shadowSize: [0, 0],
                    iconSize: this.tailleIcone,
                }),
                blueIcon: icon({
                    iconUrl:
                        "https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-blue.png",
                    shadowUrl: "",
                    shadowSize: [0, 0],
                    iconSize: this.tailleIcone,
                }),
                purpleIcon: icon({
                    iconUrl:
                        "https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-violet.png",
                    shadowUrl: "",
                    shadowSize: [0, 0],
                    iconSize: this.tailleIcone,
                }),
                orangeIcon: icon({
                    iconUrl:
                        "https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-orange.png",
                    shadowUrl: "",
                    shadowSize: [0, 0],
                    iconSize: this.tailleIcone,
                }),
            },
            bornesData: this.chargerBornes(this.centreParDefaut),
            routeSwitch: null,
            threshold: 0.3,
            connexionsCompatible: [25, 1036, 27],
        };
    },
    beforeCreate() {
        this.centreParDefaut = [45.78433216234776, 4.869708582722543];
        this.tailleIcone = [25, 41];
        if (navigator.geolocation) {
            navigator.geolocation.getCurrentPosition(
                (success) => {
                    this.centre = [
                        success.coords.latitude,
                        success.coords.longitude,
                    ];
                },
                (error) => {
                    alert(error);
                    console.error(error.message);
                }
            );
        }
    },
    methods: {
        calculerDistance(lat1, lon1, lat2, lon2) {
            var R = 6371; // km
            var dLat = ((lat2 - lat1) * Math.PI) / 180;
            var dLon = ((lon2 - lon1) * Math.PI) / 180;
            lat1 = (lat1 * Math.PI) / 180;
            lat2 = (lat2 * Math.PI) / 180;

            var a =
                Math.sin(dLat / 2) * Math.sin(dLat / 2) +
                Math.sin(dLon / 2) *
                    Math.sin(dLon / 2) *
                    Math.cos(lat1) *
                    Math.cos(lat2); // haut
            var c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
            var d = R * c;

            return d;
        },

        /**
         * Charges les bornes pour un centre donné
         * @param {[number, number]} centre
         */
        chargerBornes(centre) {
            // Store in a array all points from BORNES_COORDS inside the current map view
            // item.AddressInfo.Latitude
            // item.AddressInfo.Longitude
            if (!centre) {
                return [];
            }
            let bornesData = [];
            for (let i = 0; i < BORNES_COORDS.length; i++) {
                let item = BORNES_COORDS[i];
                if (
                    // On ne prend que les bornes qui sont aux alentours du centre de  la carte
                    item.AddressInfo.Latitude > centre[0] - this.threshold &&
                    item.AddressInfo.Latitude < centre[0] + this.threshold &&
                    item.AddressInfo.Longitude > centre[1] - this.threshold &&
                    item.AddressInfo.Longitude < centre[1] + this.threshold
                ) {
                    // On ne garde que les bornes de recharges compatibles avec les véhicules
                    for (let j = 0; j < item.Connections.length; j++) {
                        let connexion = item.Connections[j];
                        if (
                            this.connexionsCompatible.includes(
                                connexion.ConnectionTypeID
                            )
                        ) {
                            bornesData.push(item);
                        }
                    }
                }
            }
            return bornesData;
        },
        /**
         * Mets à jour les bornes en fonction du zoom
         *
         * @param {HTMLElement}
         */
        zoomUpdated(zoom) {
            this.zoom = zoom;
            // this.bornesData = this.chargerBornes(this.zoom);
        },
        /**
         * Mets à jour les bornes lors du déplacement sur la map
         *
         * @param {[number, number]} : coordonnées du nouveau centre
         */
        centerUpdated(centre) {
            this.centre = [centre.lat, centre.lng];
            // this.bornesData = this.chargerBornes(this.centre);
        },
        onRouteSwitchChange(routeSwitch) {
            this.routeSwitch = routeSwitch[0];
            this.$emit("update:routeSwitch", this.routeSwitch);

            this.w_recharges = routeSwitch[1];

            // console.log(this.w_recharges);

            // Actualiser le tableau bornesData pour afficher les bornes proches des points de recharges
            this.bornesData = [];

            for (let i = 0; i < this.w_recharges.length; i++) {
                let re = this.w_recharges[i];
                for (let j = 0; j < BORNES_COORDS.length; j++) {
                    let item = BORNES_COORDS[j];

                    if (
                        // On ne prend que les bornes qui sont aux alentours du centre de  la carte
                        item.AddressInfo.Latitude > re.lat - 0.2 &&
                        item.AddressInfo.Latitude < re.lat + 0.2 &&
                        item.AddressInfo.Longitude > re.lon - 0.2 &&
                        item.AddressInfo.Longitude < re.lon + 0.2
                    ) {
                        // On ne garde que les bornes de recharges compatibles avec les véhicules
                        for (let j = 0; j < item.Connections.length; j++) {
                            let connexion = item.Connections[j];
                            if (
                                this.connexionsCompatible.includes(
                                    connexion.ConnectionTypeID
                                )
                            ) {
                                this.bornesData.push(item);
                            }
                        }
                    }
                }
            }
        },
    },
};
</script>

<style scoped>
.map {
    height: 100vh;
    width: 100%;
    z-index: 1;
}

.leaflet-tooltip {
    border: none;
    border-radius: 15px;
    padding: 10px;
}

.leaflet-pane .leaflet-marker-pane > img {
    box-shadow: none;
}

.chargeur-titre {
    font-family: "Roboto", sans-serif;
    font-size: 1.1rem;
    font-weight: 900;
}

.chargeur-adresse,
.chargeur-complement {
    font-family: "Montserrat", sans-serif;
    font-size: 0.9rem;
    font-weight: 500;
}
</style>
