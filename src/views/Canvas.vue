<template>
<div>
    <div class="canvas-container">
        <canvas @click="canvasClicked" class="my-canvas" id="mycanvas" width="600" height="600"></canvas>
        <div class="actions">
            <button @click="clear()">Очистить</button>

            <p class="section">Линия</p>
            <p>Начальная точка (x; y)</p>
            <input type="number" v-model="line[0]" max="600" min="0" @keyup.enter="drawLine()">
            <input type="number" v-model="line[1]" max="600" min="0" @keyup.enter="drawLine()">
            <p>Конечная точка (x; y)</p>
            <input type="number" v-model="line[2]" max="600" min="0" @keyup.enter="drawLine()">
            <input type="number" v-model="line[3]" max="600" min="0" @keyup.enter="drawLine()">
            <p>Цвет (eng)</p>
            <input type="text" v-model="line[4]" @keyup.enter="drawLine()">
            <button @click="drawLine()">Нарисовать линию</button>

            <p class="section">Шаблон</p>
            <p>Имя</p>
            <input type="text" v-model="templateName" @keyup.enter="drawLine()">
            <button @click="makeTemplate()">{{templateMakeText}}</button>
            <select v-model="curTemplate">
                <option v-for="t in templates" :value="t.template" :key="t.name">{{t.name}}</option>
            </select>
            <p>{{templateSizeText}}</p>
            <button @click="drawTemplate()">{{drawTemplateText}}</button>
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
            line: [],
            templates: [],
            lastTemplate: [],
            isTemplate: false,
            isDrawTemplate: false,
            templateName: '',
            curTemplate: [],
        }
    },
    methods:{
        drawTemplate() {
            this.isDrawTemplate = !this.isDrawTemplate;
            console.log(this.curTemplate);
        },
        makeTemplate() {
            if (this.isTemplate == true) {
                if (this.templateName == "")
                    this.templateName = 'untitled';
                this.templates.push({
                    template: this.lastTemplate,
                    name: this.templateName
                });
                this.templateName = "";

                for (let i = 0; i < this.lastTemplate.length; i++) {
                    for (let j = 0; j < this.points.length; j++) {
                        if (this.points[j].x == this.lastTemplate[i].x &&
                            this.points[j].y == this.lastTemplate[i].y) {
                            this.points.splice(j, 1);
                            console.log("spliced", this.lastTemplate[i]);
                        }
                    }
                }

                this.lastTemplate = [];
                alert("Шаблон создан!");
                console.log(this.templates);
            }

            this.isTemplate = !this.isTemplate;
        },
        clear() {
            this.clearCanvas();
            this.points = [];
            this.line = [];
        },
        clearCanvas() {
            const ctx = this.canvas.getContext("2d");
            ctx.fillStyle = 'white'
            ctx.fillRect(0,0,600,600)
            ctx.fill()
        },
        loadPoints() {
            this.$axios.get('http://188.225.47.187/api/canvas/points.php').then(response=>{
                //console.log('response', response)
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
            let x1 = parseFloat(this.line[0]);
            let y1 = parseFloat(this.line[1]);
            let x2 = parseFloat(this.line[2]);
            let y2 = parseFloat(this.line[3]);
            let color = this.line[4];
            //console.log(x1, y1, x2, y2);
            let length = Math.sqrt(Math.pow((x1 - x2), 2) + Math.pow((y1 - y2), 2));

            let xCoff = Math.abs(x1 - x2) / length;
            let yCoff = Math.abs(y1 - y2) / length;

            xCoff = (x1 > x2) ? -xCoff : xCoff;
            yCoff = (y1 > y2) ? -yCoff : yCoff;

            //console.log('pre', length, xCoff, yCoff);

            for (let i = 0; i < length; i++) {
                let dot = {
                                x: x1 + parseFloat(i * xCoff),
                                y: y1 + parseFloat(i * yCoff),
                                color: color
                            }
                //console.log(dot);
                this.points.push(dot);
            }
            this.line = [];
        },
        canvasClicked(e) {
            if (this.isTemplate == false && this.isDrawTemplate == false) {
                this.$axios.get("http://188.225.47.187/api/canvas/setpoint.php", {
                    params:{
                        x: e.pageX,
                        y: e.pageY
                    }
                })
                //console.log(e.pageX, e.pageY);
                let dot = {
                    x: e.pageX,
                    y: e.pageY,
                    color: "green"
                };
                this.points.push(dot);
            } else if (this.isTemplate == true && this.isDrawTemplate == false) {
                //рисуем шаблон
                let dot = {
                    x: e.pageX,
                    y: e.pageY,
                    color: "blue"
                };
                this.points.push(dot);
                this.lastTemplate.push(dot);
            } else if (this.isTemplate == false && this.isDrawTemplate == true) {
                //asd
                for (let i = 0; i < this.curTemplate.length; i++) {
                    let dot = {
                        x: e.pageX + this.curTemplate[i].x - this.curTemplate[0].x,
                        y: e.pageY + this.curTemplate[i].y - this.curTemplate[0].y,
                        color: "orange"
                    };
                    this.points.push(dot);
                }
            }
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
    },
    computed: {
        templateMakeText() {
            if (this.isTemplate == false)
                return "Создать шаблон";
            else
                return "Закончить";
        },
        templateSizeText() {
            if (this.templates.length > 0)
                return "Шаблонов: " + this.templates.length;
            else
                return "Шаблонов пока нет";
        },
        drawTemplateText() {
            if (this.isDrawTemplate == false)
                return "Нарисовать шаблон";
            else
                return "Закончить рисование";
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
    width: 150px;
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
    width:100%;
    text-align:center;
    margin-bottom: 3px;
    margin-top: 3px;
    padding:10px 10px;
    transition:all 0.1s;
    border: 0;
    border-radius: 2px;
}
button:hover {
    background-color:#57c743;
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
select {
    width: 100%;
}
.input {
    width: 100%;
}
</style>