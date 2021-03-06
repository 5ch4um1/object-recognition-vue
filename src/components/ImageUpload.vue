<template>
    <div>
        <div class="my-5">

            <canvas width="240" height="120" ref="canvas"></canvas>

            <image-uploader :preview="true" :className="['fileinput', { 'fileinput--loaded': hasImage }]"
                            capture="environment" :debug="1" doNotResize="gif" :autoRotate="true" outputFormat="verbose"
                            @input="setImage">
                <label for="fileInput" slot="upload-label" class="upload-label">
                    <figure>
                        <svg class="upload-icon mt-5" xmlns="http://www.w3.org/2000/svg" width="32" height="32"
                             viewBox="0 0 32 32">
                            <path class="path1"
                                  d="M9.5 19c0 3.59 2.91 6.5 6.5 6.5s6.5-2.91 6.5-6.5-2.91-6.5-6.5-6.5-6.5 2.91-6.5 6.5zM30 8h-7c-0.5-2-1-4-3-4h-8c-2 0-2.5 2-3 4h-7c-1.1 0-2 0.9-2 2v18c0 1.1 0.9 2 2 2h28c1.1 0 2-0.9 2-2v-18c0-1.1-0.9-2-2-2zM16 27.875c-4.902 0-8.875-3.973-8.875-8.875s3.973-8.875 8.875-8.875c4.902 0 8.875 3.973 8.875 8.875s-3.973 8.875-8.875 8.875zM30 14h-4v-2h4v2z"></path>
                        </svg>
                    </figure>
                    <span class="upload-caption">{{hasImage ? "Replace" : "Click to upload"}}</span>
                </label>
            </image-uploader>
        </div>
    </div>
</template>

<script lang="ts">
    import {Component, Prop, Vue} from "vue-property-decorator";
    import * as cocoSSD from '@tensorflow-models/coco-ssd';
    import PredictionRenderer from "./PredictionRenderer";


    @Component
    export default class ImageUpload extends Vue {

        @Prop() private model: cocoSSD.ObjectDetection;

        private canvas: HTMLCanvasElement;

        private hasImage = false;
        private image = null;


        setImage(output: any) {

            this.hasImage = true;
            this.image = output;

            this.canvas = <HTMLCanvasElement>this.$refs.canvas;
            const ctx = <CanvasRenderingContext2D> this.canvas.getContext("2d");

            var image = new Image();
            image.onload = (event: any) => {

                var path = event.path || (event.composedPath && event.composedPath());
                
                ctx.clearRect(0, 0, ctx.canvas.width, ctx.canvas.height);

                ctx.canvas.width = Math.min(800, path[0].width);
                ctx.canvas.height = Math.min(600, path[0].height);

                ctx.drawImage(image, 0, 0);

                this.model.detect(this.canvas).then((predictions: Array<any>) => {
                    new PredictionRenderer().render(predictions, ctx);
                });
            };
            image.src = output.dataUrl;
        }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss">
    #fileInput {
        display: none;
    }

    .upload-icon {
        fill: #e1e1e1;
    }

    .upload-label {
        display: block;
    }

    .img-preview {
        display: none;
    }

</style>
