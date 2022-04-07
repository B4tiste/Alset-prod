<template>
  <div class="form">
    <label for="depart">{{ label }}</label>
    <vue-simple-suggest 
      v-model="text"
      :list="completions"
      :placeholder="labelPlaceholder"
      class="input"
    />
      
  </div>
</template>
<script>
import VueSimpleSuggest from 'vue-simple-suggest'
import 'vue-simple-suggest/dist/styles.css' // Optional CSS
import mapboxClient from '../services/MapboxClient';

export default {
    components: {
      VueSimpleSuggest
    },
    data() {
        return {
            text: ""
        };
    },
  props: {
    label: {
      required: true,
      type: String,
    },
    labelPlaceholder: {
      required: true,
      type: String,
    },
    completions: {
      type: Array
    }
  },
  watch: {
    text(text) {
      this.$emit("update:adresse", text);
    }
  },
  methods: {
    async onClick() {
      if (this.text !== null && this.text.length > 0) {

        let coords;
        try {
          coords = await mapboxClient.getCoords(this.text);
        } catch (error) {
          alert(error);
          console.error(error);
          return;
        }
          
        // Création d'un objet :
        const departInfo = {
          coords: {
            lat: coords.features[0].center[1],
            lon: coords.features[0].center[0],
          },
          adresse: {
            adresseSaisie: this.text,
          },
        };

        // Envoie des coordonnées au parent (App.vue)
        this.$emit("update:adresse", departInfo);
      }
    }
  }
};
</script>

<style scoped>
.form {
    display: flex;
    flex-direction: column;
    justify-content: space-around;
    padding: 10px 0;
    font-size: var(--font-size-text);
    font-family: var(--font-family-text)!important;
}

.form label {
    color: var(--indigo);
    font-family: var(--font-family-title);
    font-size: var(--font-size-text);
    padding: 10px;
}
</style>
