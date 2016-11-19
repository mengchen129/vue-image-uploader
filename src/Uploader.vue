<template>
    <div class="upload-wrap" @click="triggerUpload" :style="{backgroundImage: 'url(' + bgUrl + ')', backgroundSize: bgSize}">
        <input type="file" ref="fileInput" accept="image/png,image/jpg,image/jpeg" style="display: none;" @change="fileChange">
        <div class="upload-ok" v-if="finished"></div>
        <transition name="fade">
            <progress-bar v-if="compressStatus == 1" speed="fast" doing-title="正在压缩"></progress-bar>
        </transition>
        <transition name="fade">
            <progress-bar v-if="uploadStatus == 1" speed="normal" doing-title="正在上传"></progress-bar>
        </transition>
    </div>
</template>

<script>
    import ProgressBar from './ProgressBar.vue';
    import EXIF from 'exif-js';
    export default {
        name: 'uploader',
        components: {
            [ProgressBar.name]: ProgressBar
        },
        props: ['params'],
        data () {
            return {
                defaultParams: {
                    clipWidth: 200,
                    clipHeight: 200,
                    quality: 1
                },
                localImage: null,
                compressStatus: 0,
                uploadStatus: 0,
                finished: false,
                realImage: null
            }
        },
        mounted: function() {
            if (this.params) {
                this.defaultParams = this.params;
            }
        },
        computed: {
            bgUrl: function() {
                if (this.realImage) {
                    return this.realImage;
                } else if (this.localImage) {
                    return this.localImage;
                } else {
                    return 'http://img.ishequ360.com/images/zg/supply/icon-add-photo.png';
                }
            },
            bgSize: function() {
                if (this.realImage || this.localImage) {
                    return 'cover';
                } else {
                    return '32px 32px';
                }
            }
        },
        methods: {
            triggerUpload: function() {
                if (this.uploadStatus == 1) return;
                this.$refs.fileInput.click();
            },
            fileChange: function(e) {
                var self = this;
                var target = e.target;
                if (!target.value) {
                    return;
                }

                self.finished = false;

                var file = target.files[0];
                EXIF.getData(file, function() {
                    var orientation = EXIF.getTag(this, EXIF.Tags);
                    alert('方向: ' + orientation);
                });

                var fileReader = new FileReader();
                fileReader.onload = function(e) {
                    self.localImage = e.target.result;

                    self.compress(self.localImage, {
                        width: self.defaultParams.clipWidth,
                        height: self.defaultParams.clipHeight
                    }).then((localMinImage) => {
                        self.compressStatus = 2;
                        self.uploadStatus = 1;

                        setTimeout(() => {
                            self.uploadStatus = 2;
                            self.finished = true;
                            self.realImage = localMinImage;
                        }, 5000);
                    });
                };
                fileReader.readAsDataURL(file);
                self.compressStatus = 1;

            },
            compress: function (dataURL, size) {
                var self = this;
                return new Promise(function(resolve) {
                    var img = new window.Image();
                    img.src = dataURL;
                    // onload
                    img.onload = function () {
                        var canvas = document.createElement('canvas');
                        var ctx = canvas.getContext('2d');
                        canvas.width = size.width;
                        canvas.height = size.height;
                        var RATIO = canvas.width / canvas.height;
                        var cutx = 0;
                        var cuty = 0;
                        var cutw = img.width;
                        var cuth = img.height;
                        if (cutw / cuth > RATIO) {
                            // 宽超过比例了]]
                            var realw = cuth * RATIO;
                            cutx = (cutw - realw) / 2;
                            cutw = realw;
                        } else if (cutw / cuth < RATIO) {
                            // 长超过比例了]]
                            var realh = cutw / RATIO;
                            cuty = (cuth - realh) / 2;
                            cuth = realh;
                        }
                        ctx.drawImage(img, cutx, cuty, cutw, cuth, 0, 0, canvas.width, canvas.height);
                        var ndata = canvas.toDataURL('image/jpeg', self.defaultParams.quality);
                        resolve(ndata);
                    };
                });
            }
        }
    }
</script>

<style>
    .upload-wrap {
        width: 100px;
        height: 100px;
        border: 1px dashed #dddddd;
        border-radius: 5px;
        position: relative;
        background-repeat: no-repeat;
        background-position: center;
    }

    .fade-enter-active, .fade-leave-active {
        transition: opacity .5s
    }
    .fade-enter, .fade-leave-active {
        opacity: 0
    }

    .upload-ok {
        position: absolute;
        top: 10px;
        right: 10px;
        width: 25px;
        height: 25px;
        border-radius: 50%;
        box-shadow: 0 1px 4px rgba(0, 0, 0, 0.3);
        background: url('http://img.ishequ360.com/images/zg/supply/icon-upload-ok.png') no-repeat center;
        background-size: 25px 25px;
    }
</style>
