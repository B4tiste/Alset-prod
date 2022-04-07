<template>
    <div>
        <button type="button" :disabled="isDisabled" @click="setItineraire">
            c'est parti !
        </button>
    </div>
</template>

<script>
import mapboxClient from "../services/MapboxClient";
import debounce from "lodash.debounce";

export default {
    props: {
        departAdresse: {
            type: String,
        },
        arriveeAdresse: {
            type: String,
        },
    },
    computed: {
        isDisabled() {
            let isDisabled;
            if (
                this.departAdresse != null &&
                this.arriveeAdresse != null &&
                this.departAdresse.length > 0 &&
                this.arriveeAdresse.length > 0
            ) {
                isDisabled = false;
            } else {
                isDisabled = true;
            }

            this.$emit("update:renverser", isDisabled);

            return isDisabled;
        },
    },
    watch: {
        departAdresse: debounce(async function (depart) {
            const completions = await (
                await mapboxClient.getCoords(depart)
            ).features;

            const completionsAdresses = completions.map((completion) => {
                return {
                    adresse: completion.place_name,
                };
            });
            this.$emit("update:completionDepart", completionsAdresses);
        }, 50),
        arriveeAdresse: debounce(async function (arrivee) {
            const completions = await (
                await mapboxClient.getCoords(arrivee)
            ).features;

            const completionsAdresses = completions.map((completion) => {
                return {
                    adresse: completion.place_name,
                };
            });

            this.$emit("update:completionArrivee", completionsAdresses);
        }, 50),
    },
    methods: {
        async setItineraire() {
            if (
                this.departAdresse !== null &&
                this.departAdresse.length > 0 &&
                this.arriveeAdresse !== null &&
                this.arriveeAdresse.length > 0
            ) {
                let departCoords;
                let arriveeCoords;
                try {
                    departCoords = await mapboxClient.getCoords(
                        this.departAdresse
                    );
                    arriveeCoords = await mapboxClient.getCoords(
                        this.arriveeAdresse
                    );
                } catch (error) {
                    alert(error);
                    console.error(error);
                    return;
                }

                // Création d'un objet :
                const coordonneesItineraire = {
                    depart: {
                        coords: {
                            lat: departCoords.features[0].center[1],
                            lon: departCoords.features[0].center[0],
                        },
                        adresse: this.departAdresse,
                    },
                    arrivee: {
                        coords: {
                            lat: arriveeCoords.features[0].center[1],
                            lon: arriveeCoords.features[0].center[0],
                        },
                        adresse: this.arriveeAdresse,
                    },
                };
                // Envoie des coordonnées au parent (App.vue)
                this.$emit(
                    "update:coordonneesItineraire",
                    coordonneesItineraire
                );
            }
        },
    },
};
</script>

<style scoped>
button {
    outline: none;
    align-self: center;
    padding: 5px 40px;
    border: 2px solid var(--indigo);
    font-size: var(--font-size-text);
    background-color: var(--indigo);
    color: #fff;
    border-radius: 50px;
    text-transform: uppercase;
    letter-spacing: 2px;
    cursor: pointer;
    margin: 10px 0;
    transition: 0.3s;
}

button:hover {
    color: #fff;
    border-color: var(--indigo);
    background-color: var(--magenta);
    transition: 0.3s;
}

button:disabled {
    border-color: var(--manatee);
    background-color: var(--manatee);
    cursor: not-allowed;
}
</style>