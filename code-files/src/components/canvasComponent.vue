<template>
    <div class="container">
        <div class="controls-container">
            <h1>Options</h1>
            <ControlsContainer name="Iterations" :value="iterations" :min="0" :max="15" @updateValue="updateEmittedValue" />
            <ControlsContainer name="Angle" :value="angle" :min="0" :max="360" @updateValue="updateEmittedValue" />
            <ControlsContainer name="Length" :value="length" :min="0" :max="1000" @updateValue="updateEmittedValue" />
            <ControlsContainer name="Length Scalar" :value="length_scalar" :min="0" :max="2" :step="0.01" @updateValue="updateEmittedValue" />
            <ControlsContainer name="Thickness" :value="thickness" :min="0" :max="100" @updateValue="updateEmittedValue" />
            <ControlsContainer name="Angle Scalar" :value="angle_scalar" :min="0" :max="2" :step="0.01" @updateValue="updateEmittedValue" />
            <ControlsContainer name="Thickness Scalar" :value="thickness_scalar" :min="0" :max="2" :step="0.01" @updateValue="updateEmittedValue" />
            <br />
            <WebGPUCheck />
        </div>
        <div class="canvas-container">
            <canvas ref="canvas"></canvas>
        </div>
    </div>
</template>
<script>
import init, { draw_fractal, clear, to_lower_case, to_float } from '../../public/pkg/rust_backend.js';
import ControlsContainer from './ControlsContainer.vue';
import WebGPUCheck from './WebGPUCheck.vue';

export default {
    components: { WebGPUCheck, ControlsContainer },
    data() {
        return {
            iterations: 10,
            length: 750,
            length_scalar: 0.49, //change to 50 to recreate bug on Friday, add the /100 too
            angle: 45,
            angle_scalar: 0.99,
            angle_constant: 0,
            thickness: 20,
            thickness_scalar: 0.75,
            start_color: "#DE493E",
        };
    },
    methods: {
        drawFractal() {
            const canvas = this.$refs.canvas;
            clear(canvas);
            draw_fractal(10, 0, this.length, this.length_scalar, this.angle, this.angle_scalar, this.iterations, canvas, this.thickness, this.thickness_scalar, this.start_color);
            draw_fractal(10, 0, this.length, this.length_scalar, -this.angle, this.angle_scalar, this.iterations, canvas, this.thickness, this.thickness_scalar, this.start_color);
        },
        update(element, value) {
            element.innerHTML = value;
        },
        updateEmittedValue(name, value) { 
            name = to_lower_case(name);
            this[name] = to_float(value); //doesn't update?
            this.drawFractal();
        },
    },
    async mounted() {
        await init();
        const canvas = this.$refs.canvas;
        const context = canvas.getContext('2d');

        context.canvas.width = window.innerWidth * 0.8;
        context.canvas.height = window.innerHeight * 0.975; // this doesn't feel great but not sure what alternative look like

        context.translate(canvas.width / 2, canvas.height);
        context.rotate(-Math.PI / 2);

        this.drawFractal();
    },
};
</script>

<style scoped>
    .h1 {
        color: #ffd700;
    }

    body {
        margin: 0;
        padding: 0;
    }

    canvas {
        border-left: 1px solid #f4f4f4;
        margin: 0;
        padding: 0;
    }
    .container {
        display: flex;
        flex-direction: row;
        margin: 0;
        padding: 0;
        width: 100%;
        height: 100%;
    }
    .canvas-container {
        background: #f4f4f4;
        flex: 80%;
    }
    .controls-container {
        color: #f4f4f4;
        background: #333333;
        flex: 20%;
    }

    .controls-container h1 {
        color: #ffd700;
        font-family: 'Open Sans', sans-serif;
    }
</style>
