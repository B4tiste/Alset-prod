<template>
    <div>
        <div class="popup-trajet active" @click="showItinariesSearching">
            <img class="rotate up" src="../assets/images/icons/chevron-en-haut.png" alt="chevron">
        </div>
    </div>
</template>

<script>
export default {
    methods: {
        showItinariesSearching() {
            let trajet = document.querySelector('.trajet');
            let chevron = document.querySelector('.rotate')

            trajet.classList.toggle('active');
            chevron.classList.toggle('up')
        }  
    }
}
</script>

<style scoped>

.popup-trajet {
    position: absolute;
    top: 0;
    left: 50%;
    transform: translateX(-50%);
    min-width: 100px;
    height: 20px;
    border-bottom-left-radius: 100px;
    border-bottom-right-radius: 100px;
    background-color: var(--white-bg);
    border: 2px solid var(--taupe);
    border-top: none;
    box-shadow: 0px 0px 3px 0px var(--taupe) inset;
    z-index: 150;
    cursor: pointer;
    text-align: center;
}

.rotate {
    height: 15px;
    transition: .3s;
}

.rotate.up {
    transform: rotate(180deg);
    transition: .3s;
}
</style>