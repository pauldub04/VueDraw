<template>
<div>
    <div class="canvas-container">
        <canvas @click="canvasClicked" class="my-canvas" id="mycanvas" width="600" height="600"></canvas>
        <div class="actions">
            <button @click="clear()">Очистить</button>

            <p class="section">Координаты линии</p>
            <p>Начальная точка (x; y)</p>
            <input type="number" v-model="lineCoords[0]" max="600" min="0" @keyup.enter="drawLine()">
            <input type="number" v-model="lineCoords[1]" max="600" min="0" @keyup.enter="drawLine()">
            <p>Конечная точка (x; y)</p>
            <input type="number" v-model="lineCoords[2]" max="600" min="0" @keyup.enter="drawLine()">
            <input type="number" v-model="lineCoords[3]" max="600" min="0" @keyup.enter="drawLine()">

            <button @click="drawLine()">Нарисовать линию</button>
            <button>очистить</button> 
        </div>
    </div>
</div>
</template>

<script>
export default {
    data(){
        return {
            canvas: null,
            points: [],
            lineCoords: [],
        }
    },
    methods:{
        clear() {
            this.clearCanvas();
            this.points = [];
            this.lineCoords = [];
        },
        clearCanvas() {
            const ctx = this.canvas.getContext("2d");
            ctx.fillStyle = 'white'
            ctx.fillRect(0,0,600,600)
            ctx.fill()
        },
        loadPoints() {
            this.$axios.get('http://188.225.47.187/api/canvas/points.php').then(response=>{
                console.log('response', response)
                this.points = response.data.filter((el) => el.x !== null && el.y !== null)
            })
        },
        drawPoint(point) {
            const {x, y, color} = point
            const ctx = this.canvas.getContext("2d");
            ctx.fillStyle = color;
            const width = 5;
            const height = 5;
            ctx.fillRect(x - width / 2, y - height / 2, width, height);
            ctx.fill();
            // ctx.arc(x, y, radius, 0, 2 * Math.PI, false);
      
        },
        drawLine() {
            let x1 = this.lineCoords[0];
            let y1 = this.lineCoords[1];
            let x2 = this.lineCoords[2];
            let y2 = this.lineCoords[3];
            //console.log(x1, y1, x2, y2);
            let length = Math.sqrt(Math.pow((x1 - x2), 2) + Math.pow((y1 - y2), 2));

            let xCoff = Math.abs(x1 - x2) / length;
            let yCoff = Math.abs(y1 - y2) / length;

            for (let i = 0; i < length; i++) {
                this.points.push({
                                    x: x1 + i * xCoff,
                                    y: y1 + i * yCoff,
                                    color: "blue"
                                });
            }


            //МИША ТЫ ДАУН


            this.lineCoords = [];
        },
        canvasClicked(e) {
            this.$axios.get("http://188.225.47.187/api/canvas/setpoint.php", {
                params:{
                    x: e.pageX,
                    y: e.pageY
                }
            })
            console.log(e.pageX, e.pageY);
            this.points.push({
                                x: e.pageX,
                                y: e.pageY,
                                color: "green"
                            });
        },
    },
    mounted() {
        this.canvas = document.getElementById('mycanvas');
        this.loadPoints();
        /*setInterval(()=>{
            this.loadPoints()
        }, 4000)*/
    },
    watch: {
        points() {
            this.clearCanvas()
            for (const point of this.points) {
                this.drawPoint(point);
            }
        }
    }
}
</script>



<style lang="scss">
.canvas-container {
    display: flex;
}
.my-canvas {
    width: 600px;
    height: 600px;
    margin-left: 100;
}
.actions {
    width: 500px;
    display: inline-block;
    margin-left: 5px;
    margin-top: 2px;
    color: black;
    text-align: left;
    font-family: Roboto;
}
button {
    background-color:#eee;
    color:black;
    text-decoration:none;
    display:block;
    width:150px;
    text-align:center;
    margin-bottom: 3px;
    margin-top: 3px;
    padding:10px 10px;
    transition:all 0.1s;
    border: 0;
    border-radius: 2px;
}
button:hover {
    background-color:#5ef474;
}
button:active {
    background-color:#449c35;
    color:white;
}
p {
    margin: 0px;
    font-size: 14px;
}
.section{
    color: #449c35;
    font-size: 18px;    
}
</style>