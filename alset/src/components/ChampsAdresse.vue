<template>
  <div class="form">
    <label for="depart">{{ label }}</label>
    <input
      v-model="text"
      :placeholder="labelPlaceholder"
      id="depart"
    />
    <button @click="onClick">{{ texteBtn }}</button>
  </div>
</template>
<script>
import mapboxClient from '../services/MapboxClient';

export default {
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
    texteBtn: {
      required: true,
      type: String,
    },
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
}

.form label {
    color: var(--jungle);
    font-family: var(--font-family-title);
    font-size: var(--font-size-text);
    padding: 0 10px;
}

.form input {
    width: 100%;
    height: 30px;
    border: none;
    outline: none;
    font-size: var(--font-size-text);
    border-bottom: 2px solid var(--taupe);
    border-top-left-radius: 5px;
    border-top-right-radius: 5px;
    padding: 0 10px;
    transition: .5s;
}

.form input:focus {
    border: none;
    border-bottom: 2px solid var(--jungle);
    background-color: var(--white-bg);
    transform: scaleX(1) translateY(0px);
    transition: .5s;
}

.form button {
    outline: none;
    align-self: center;
    width: 80%;
    height: 30px;
    border:2px solid var(--taupe) ;
    font-size: var(--font-size-text);
    color: var(--taupe);
    border-radius: 50px;
    text-transform: uppercase;
    letter-spacing: 2px;
    background-color: transparent;
    cursor: pointer;
    margin: 10px 0;
    transition: .2s;
}

.form button:hover {
    color: #fff;
    border-color: var(--jungle) ;
    background-color: var(--jungle);
    transition: .3s;
}
</style>
