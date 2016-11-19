<template>
    <div>
        <h4>设置:</h4>
        <p><label>裁剪宽度: <input type="text" v-model="params.clipWidth"></label></p>
        <p><label>裁剪高度: <input type="text" v-model="params.clipHeight"></label></p>
        <p><label>质量(0-1): <input type="text" v-model="params.quality"></label></p>

        <uploader :params="params"></uploader>
    </div>
</template>

<script>
    import Uploader from './Uploader.vue';
    export default {
        name: 'app',
        components: {
            [Uploader.name]: Uploader
        },
        data () {
            return {
                params: {
                    clipWidth: 200,
                    clipHeight: 200,
                    quality: 1
                }
            }
        },
        computed: {
        },
        methods: {
        },
        mounted () {
        }
    }
</script>

<style>
</style>
