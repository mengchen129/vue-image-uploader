<template>
    <div class="progress-bar-container">
        <div class="progress-title">{{this.doingTitle}} ({{value}}%)</div>
        <div class="progress-bar">
            <div class="progress-bar-passed" :style="{width: value + '%'}"></div>
        </div>
    </div>
</template>

<script>
    const FULL_VALUE = 100;
    export default {
        name: 'progress-bar',
        props: ['doingTitle', 'speed'],
        data() {
            return {
                value: 0,
                maxStep: 0,
                timer: null
            }
        },
        mounted() {
            switch (this.speed) {
                case 'fast':
                    this.maxStep = 27; break;
                case 'normal':
                    this.maxStep = 17; break;
                case 'slow':
                    this.maxStep = 7; break;
                default:
                    this.maxStep = 17;
            }
            this.randomGo();
        },
        methods: {
            randomGo() {
                this.timer = setInterval(() => {
                    var currentValue = this.value;
                    if (currentValue >= 99) {
                        clearInterval(this.timer);
                        console.log('计时器停止');
                    }
                    var step = parseInt((FULL_VALUE - currentValue) * Math.random());
                    this.value += Math.min(this.maxStep, step);
                }, 300);
            }
        }
    }
</script>

<style lang="sass">
    @import "./ProgressBar.scss";
</style>