<script setup lang="ts"> //@ts-ignore
import * as d3 from "d3";
import { ref } from 'vue'
interface SolutionApplicationDict {
    [key: string]: boolean
}
interface SensorDatum {
    x: number;
    y: number;
    type: "suburb" | "rural" | "urban";
    value?: number;
}

interface HydratedSensorDatum extends SensorDatum {
  value: number
}

type ColorData = [string, number]
type PValue = 4 | 8 | 16 | 2 | 32;
const chart  = ref(null);
const pValue = ref(4);
const gridSize = ref(10);
const regionalSeed = ref([230, 300, 420])
const solutions = ref({
    whiteRoofs: [0, 8, 18],
    concretePavement: [0, 8, 8],
    maple: [20, 10, 15],
    // heatGoblins: [-10, -15, -30],
    // iceDragon: [15, 20, 50],
    vinylSiding: [0, 18, 18],
    woodSiding: [0, 8, 8],
    });

const colors = ref([
    ['#79CB40', 3],
    ['#FFB840', 2],
    ['#f40000', 3],
    ['#3d0c0a', 3],
    ])
const domain = ref([240, 420]);


const solutionApplicationDict = ref({
    whiteRoofs: false,
    concretePavement: false,
    maple: false,
    vinylSiding: false,
    woodSiding: false,
    // heatGoblins: false,
    // iceDragon: false
});
const initialSensorData: SensorDatum[] = [
        {
          //magnolia manor
          x: 80,
          y: 50,
          type: "suburb",
        },
        {
          //green acres
          x: 240,
          y: 30,
          type: "rural",
        },
        {
          //tulip gardens
          x: 300,
          y: 40,
          type: "rural",
        },
        {
          //totten towers
          x: 93,
          y: 110,
          type: "urban",
        },
        {
          //city center
          x: 160,
          y: 90,
          type: "urban",
        },
        {
          //city center 2
          x: 210,
          y: 75,
          type: "urban",
        },
        {
          //city center 3
          x: 200,
          y: 100,
          type: "urban",
        },
        {
          //putter's
          x: 300,
          y: 125,
          type: "suburb",
        },
        {
          //faraday factory
          x: 40,
          y: 175,
          type: "urban",
        },
        {
          //faraday factory 2
          x: 60,
          y: 220,
          type: "urban",
        },
        {
          //raine's field
          x: 130,
          y: 155,
          type: "urban",
        },
        {
          //lakeview square
          x: 210,
          y: 165,
          type: "suburb",
        },
        {
          //sunny glen
          x: 270,
          y: 240,
          type: "suburb",
        },
        {
          //costly commons
          x: 160,
          y: 210,
          type: "urban",
        },
        {
          //rural 1
          x: 10,
          y: 10,
          type: "rural",
        },
        {
          //rural 2
          x: 120,
          y: 10,
          type: "rural",
        },
        {
          //rural 3
          x: 200,
          y: 10,
          type: "rural",
        },
        {
          //rural 4
          x: 340,
          y: 10,
          type: "rural",
        },
        {
          //rural 5
          x: 10,
          y: 125,
          type: "rural",
        },
        {
          //rural 6
          x: 330,
          y: 170,
          type: "rural",
        },
        {
          //rural 7
          x: 10,
          y: 300,
          type: "rural",
        },
        {
          //rural 8
          x: 370,
          y: 220,
          type: "rural",
        },
        {
          //rural 9
          x: 210,
          y: 290,
          type: "rural",
        },
      ];

const displaySensors = ref(false)

const isFuzzed = ref(false);

    function updateSolutionDict(key: string) {
        const newDict = {
            ...solutionApplicationDict.value,
      };
      //@ts-ignore
      newDict[key] = !solutionApplicationDict.value[key];

      solutionApplicationDict.value = newDict;
    console.log(key, solutionApplicationDict.value)
    }
    function applySolutions() {
      for (let k of Object.keys(solutionApplicationDict.value)) { //@ts-ignore
        if (solutionApplicationDict.value[k]) {
          applySolution(k);
        }
      }
    }
    function applySolution(key: string) { //@ts-ignore
      let vars = solutionApplicationDict.value[key] ? solutions.value[key] : [0, 0, 0];
      console.info("hello", key, vars)
      for (const d of initialSensorData) {
        if (d.type === "suburb" && d.value) {
          d.value = d.value - vars[1];
        }
        if (d.type === "urban" && d.value) {
          d.value = d.value - vars[2];
        }
        if (d.type === "rural" && d.value) {
          d.value = d.value - vars[0];
        }
      }
    }
    function randNumber() {
      return Math.floor(Math.random() * 10);
    }
    function setupData() {
      for (const d of initialSensorData) {
        const randIncrease = Math.random() > 0.5;
        const c = randIncrease ? randNumber() : randNumber() * -1;
        // const c = 0;
        if (d.type === "suburb") {
          d.value = regionalSeed.value[1] + c;
        }
        if (d.type === "urban") {
          d.value = regionalSeed.value[2] + c;
        }
        if (d.type === "rural") {
          d.value = regionalSeed.value[0] + c;
        }
      }
    }

function getAvg() {
      return initialSensorData.reduce((a, b) => a + (b.value || 0),0) / initialSensorData.length || 0
}

function showSensors() {
  displaySensors.value = true;
  drawSensors();
}
function hideSensors() {
  displaySensors.value = false;
  removeSensors();
}
function drawSensors() {
    for( const i of initialSensorData) {
        const svg = d3.select('svg');
        svg
        .append("circle")
        .attr('cx', i['x'] )
    .attr('cy', i['y'] )
    .attr('r','2px')
    .style("stroke", 'white')
    .style('fill', 'red');
    }
}
function removeSensors() {
    d3.selectAll("circle").remove();
}
function addColor() {
    colors.value.push(["#ff0000", 1]);
}

function deleteColor(i: number) {
    colors.value.splice(i, 1)
}

function redrawChart() {
    var s = d3.selectAll(".svg-container");
    s.remove();
    setupData();
    applySolutions();
    draw();
}
function draw() {
   const width = 432;
   const height = 324;
   const p = pValue.value

   const colorRange = colors.value.reduce((acc, curr) => { //@ts-ignore
    for(let i = 0; i < curr[1]; i++) {
        acc.push(curr[0]);
    }

    return acc;

   }, [])

    // is it fine grained enough to see solutions?
    // image at coords

    const svg = d3
    .select(chart.value)
    .append("div")
    // Container class to make it responsive.
    .classed("svg-container", true) 
    .append("svg")
                .attr("preserveAspectRatio", "xMinYMin meet")
    .classed("svg-content-responsive", true)
            .attr("viewBox", "0 0 432 324")

    console.info(colorRange, colors.value)

    const colorScale = d3
    .scaleSequential(d3.interpolateRgbBasis(colorRange))
    .domain(domain.value);

    const interpolateValue = (x: number, y: number) => {
    let totalWeight = 0;
    let interpolatedValue = 0;

    initialSensorData.forEach((d) => {
        const distance =
        Math.sqrt((x - d.x) ** p + (y - d.y) ** p) + 10 ** -15;
        const weight = 1 / distance;
        totalWeight += weight; //@ts-ignore
        interpolatedValue += weight * d.value;
    });

    return interpolatedValue / totalWeight;
    };

    for (let x = 0; x < width; x += gridSize.value) {
        for (let y = 0; y < height; y += gridSize.value) {
            const interpolated = interpolateValue(x, y);
            svg
            .append("rect")
            .attr("x", x)
            .attr("y", y)

            .attr("width", gridSize.value)
            .attr("height", gridSize.value)
            .style("opacity", 0.65)
            .style("fill", colorScale(interpolated));
        }
    }
    if(displaySensors.value) {
      drawSensors()
    }
}

</script>

<template>
  <div>
        <h1>Configuration</h1>
        <p>This one's for the heat goblins who couldn't be with us today. Gone but not forgotten!</p>
    <div class="configBox">
        <div class="colors">
        <h2>Colors & Weights</h2>
                    <div v-for="(c, i) in colors" :key="i">
            <input type="color" v-model="c[0]">
            <input type="number" max="10" min="1" v-model='c[1]'>
            <button @click="deleteColor(i)">x</button>
        </div>
        <button @click="addColor">Add Color</button>


        </div>
        <div class="solutionToggles">
            <h2>Solution Toggles</h2>
            <div v-for="(s, key) in solutionApplicationDict" :key="key">
        <h3>{{key}}</h3>
        <input type="checkbox" id="checkbox" v-model="solutionApplicationDict[key]" @input="updateSolutionDict(key)">
        </div>

        </div>
        <div class="solutionValues">
                    <h2>Solution Values</h2>
        <div v-for="(s, key) in solutions" :key="key">
            <h3>{{key}}</h3>
            Urban
            <input type="number" v-model='s[2]'>
            <br>
            Suburban
            <input type="number" v-model='s[1]'>
            <br>
            Rural
            <input type="number" v-model='s[0]'>

        </div>
        </div>
        <div class="pval">
            <h2>P-Value</h2>
            <select v-model.number="pValue" style="width:100px">
            <option >2</option>
            <option >4</option>
            <option >8</option>
            <option >16</option>
            <option >32</option>
          </select>
        </div>
        <div class="gridSize">
            <h2>Grid Size</h2>
            <input v-model.number="gridSize" type="number">
        </div>
        <div class="domain">
            <h2>Domain (Ceiling & Floor)</h2>
            <input v-model.number="domain[0]" type="number">
            <br>
            <input v-model.number="domain[1]" type="number">


        </div>
        <div class="stockRegional">
        <h2>Stock Regional Values</h2>
          <label for="urbanAmt">Urban</label>
          <input type="number" id="urbanAmt" v-model.number="regionalSeed[2]">
          <br>
          
          <label for="suburbanAmt">Suburban</label>
          <input type="number" id="suburbanAmt" v-model.number="regionalSeed[1]">
          <br>

          <label for="ruralAmt">Rural</label>
          <input type="number" id="ruralAmt" v-model.number="regionalSeed[0]">


        </div>
    </div>
    <div>
        <h1>Heatmap</h1>
        <button @click="redrawChart">redraw</button>
        <button @click="showSensors">show sensors</button><button @click="hideSensors">hide sensors</button>
        <!-- <span>Average: {{ getAvg() }} </span> -->
        <div class="containerBox">
      <div class="foo" ref="chart">
        <img class="bgCity" src="../assets/bg-city.png" />
        <img class="bgCity top" src="../assets/ui-overlay.png" />
        <div class="svg-container"></div>
      </div>
    </div>
    </div>
  </div>
</template>

<style scoped>

.colors div{
  display: grid;
  grid-template-columns: 1fr 5fr 1fr;
}

.solutionValues {
    grid-area: h
}

.solutionToggles {
    grid-area: g;
}

.sensorData {
    grid-area: f;
}

.colors {
    grid-area: e;
}

.pval {
    grid-area: a
}

.gridSize {
    grid-area: b
}
.solutionToggles h3 {
    display: inline;
}
.domain {
    grid-area: c
}

.stockRegional {
    grid-area: d
}

.configBox {
    overflow: scroll;
    display: grid;;
    width: 100%;
    height: 75vh;
    grid-template-columns: 1fr 1fr 1fr;
    grid-template-rows: 1fr 1fr 1fr 1fr;
    grid-column-gap: 3vh;
    grid-row-gap: 1vh;
    grid-template-areas:
    "d e h"
    "c e h"
    "b g h"
    "a g h";
}

.configBox h3 {
    text-align: left;
}
.svg-container {
  display: inline-block;
  position: relative;
  width: 100%;
  padding-bottom: 100%; /* aspect ratio */
  vertical-align: top;
  overflow: hidden;

}
.svg-content-responsive {
  display: inline-block;
  position: absolute;
  left: 0;
  opacity: 0.3;
}
.foo {
  margin: 0 auto;
  border: 1px solid red;
  height: 60vh;
  width: 80vh;
  position: relative;
}
.bgCity {
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  width: 100%;
  height: 100%;
  z-index: -1;
}
.top {
  z-index: 2;
  /* display: none; */
}

table {
  border-collapse: collapse;
  table-layout: fixed;
  width: 100%;
}

#app > div {
    width: 100%;
    max-width: none;
}

#app {
    max-width: none;
    min-width: 55%;
    margin: 0 auto;
}

th {
  background-color: #eee;
}

tr:first-of-type th {
  width: 100px;
}

tr {
    display: grid;
    grid-template-columns: 1fr 1fr 2fr;
    width: 100%;
}
tr:first-of-type th:first-of-type {
  width: 25px;
}

td {
  border: 1px solid #ccc;
  height: 1.5em;
  overflow: hidden;
}
h1 {
  font-weight: 500;
  font-size: 2.6rem;
  position: relative;
  top: -10px;
}

h3 {
  font-size: 1.2rem;
}

.greetings h1,
.greetings h3 {
  text-align: center;
}

@media (min-width: 1024px) {
  .greetings h1,
  .greetings h3 {
    text-align: left;
  }
}
</style>
