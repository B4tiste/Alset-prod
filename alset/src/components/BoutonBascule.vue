<template>
    <div>
        <div class="goOut" @click="showLeftSlider">
            <span class="line"></span>
            <span class="line"></span>
            <span class="line"></span>
        </div>
        <div class="hide" @click="showLeftSlider"></div>
    </div>
</template>

<script>
// TODO correctif sur l'affichage du btn au clique sur le fond noir
export default {
    methods: {
        showLeftSlider() {
            let btn = document.querySelector('.goOut');
            let borne = document.querySelector('.bornes');
            let hide = document.querySelector('.hide');    
            let menu = document.querySelector('.goOut')

            borne.classList.toggle('active');
            hide.classList.toggle('disable');
            btn.classList.toggle('follow');
            btn.classList.toggle('active');
            menu.classList.toggle('active');
        }
    }
};
</script>
<style scoped>
.goOut {
    visibility: hidden;
    position: absolute;
    top: 20px;
    left: 20px;
    height: 50px;
    width: 50px;
    transform: translateX(0);
    background-color: var(--white-bg);
    border: 2px solid var(--taupe);
    box-shadow: 0px 0px 5px 0px var(--taupe);
    border-radius: 50px;
    z-index: 100;
    overflow: hidden;
    transition-delay: .1s;
    transition: .5s;
}

.goOut.show {
    visibility: visible;
}

.goOut.follow {
    transform: translateX(20vw);
    transition-delay: .1s;
    transition: .5s;
}

.goOut:hover {
    cursor: pointer;
}

.line {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%,-50%);
    width: 60%;
    height: 2px;
    background-color: var(--taupe);
    transition: .5s;
}

.line:nth-child(2) {
    top: 35%;
    width: 20%;
}

.line:nth-child(3) {
    top: 65%;
    width: 40%;
}

.goOut.active .line:nth-child(1) {
    transform: translateX(200px);
}

.goOut.active .line:nth-child(2) {
    transform:translateY(0) rotate(45deg);
    top: 25px;
    left: 10px;
    width: 60%;
}

.goOut.active .line:nth-child(3) {
    transform:translateY(0) rotate(-45deg);    
    top: 25px;
    left: 10px;
    width: 60%;
}

.hide {
    position: absolute;
    top: 0;
    left: 0;
    visibility: hidden;
    width: 100vw;
    height: 100vh;
    background-color:rgba(0, 0, 0, 0.4);
    z-index:80;
    transition: visibility 0.2s linear 0;
}

.hide.disable {
    visibility: visible;
    animation-delay: 0.5s;
    animation: visibility 0.2s linear;
}

</style>