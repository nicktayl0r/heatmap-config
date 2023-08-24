<script setup lang="ts">
//@ts-ignore
import * as d3 from "d3";
declare var require: any
var seedrandom = require('seedrandom');
import { ref, onUpdated} from 'vue'
import { initialSensorData, voronaiData } from '../data';
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
const seed = ref("")
const chart = ref(null);
const pValue = ref(8);
const gridSize = ref(3);
const regionalSeed = ref([230, 360, 420])
const solutions = ref({
  whiteRoofs: [0, 20, 25, "Roof"],
  concretePavement: [0, 13, 19, "Road"],
  maple: [20, 17, 22, "Tree"],
  vinylSiding: [0, 23, 24, "Siding"],
  woodSiding: [0, 13, 14, "Siding"],
});

const colors = ref([
  ['#79CB40', 3],
  ['#FFB840', 1],
  ['#f40000', 1],
  ['#3d0c0a', 1],
])
const domain = ref([256, 418]);

const dataFlag = ref("")
let myrng: any = null;

function applyGratefulDeadConfig() {
  regionalSeed.value = [275, 380, 420];
  domain.value = [256, 418];
  pValue.value = 8;
  gridSize.value = 3;

  solutions.value = {
    whiteRoofs: [0, 20, 25, "Roof"],
    concretePavement: [0, 13, 19, "Road"],
    maple: [20, 17, 22, "Tree"],
    vinylSiding: [0, 23, 24, "Siding"],
    woodSiding: [0, 13, 14, "Siding"],
  };

  colors.value = [
    ['#8cf0e1', 4],
    ['#FDEA00', 3],
    ['#ff8000', 2],
    ['#b70000', 1],
  ]
  dataFlag.value = "initialSensorData"
  redrawChart();
}

function applyGratefulDeadVoronaiConfig() {
  regionalSeed.value = [275, 380, 420];
  domain.value = [256, 410];
  pValue.value = 4;
  gridSize.value = 6;

  solutions.value = {
    whiteRoofs: [0, 20, 25, "Roof"],
    concretePavement: [0, 13, 19, "Road"],
    maple: [20, 17, 22, "Tree"],
    vinylSiding: [0, 23, 24, "Siding"],
    woodSiding: [0, 13, 14, "Siding"],
  };

  colors.value = [
    ['#8cf0e1', 3],
    ['#FDEA00', 2],
    ['#ff8000', 2],
    ['#b70000', 2],
  ]
  dataFlag.value = "voronai"
  redrawChart();
}

function getMeanSAT() {
  const points = getPoints();
  const len = points.length; //@ts-ignore
  const temperatureSum = points.reduce((acc, curr) => {return acc + curr.value}, 0);
  console.info("mean SAT", temperatureSum/len)
}

function applyJosephConradConfig() {
  regionalSeed.value = [230, 360, 420];
  domain.value = [256, 418];
  pValue.value = 8;
  gridSize.value = 3

  colors.value = [
    ['#79CB40', 3],
    ['#FFB840', 1],
    ['#f40000', 1],
    ['#3d0c0a', 1],
  ]

  solutions.value = {
    whiteRoofs: [0, 20, 25, "Roof"],
    concretePavement: [0, 13, 19, "Road"],
    maple: [20, 17, 22, "Tree"],
    vinylSiding: [0, 23, 24, "Siding"],
    woodSiding: [0, 13, 14, "Siding"],
  };
  dataFlag.value = "initialSensorData"
  redrawChart();

}


const solutionApplicationDict = ref({
  whiteRoofs: false,
  concretePavement: false,
  maple: false,
  vinylSiding: false,
  woodSiding: false,

});

const displaySensors = ref(false)

const isFuzzed = ref(false);

function updateSolutionDict(key: string) {
  const newDict = {
    ...solutionApplicationDict.value,
  };
  //@ts-ignore
  newDict[key] = !solutionApplicationDict.value[key];

  solutionApplicationDict.value = newDict;
}
function applySolutions() { //@ts-ignore
  myrng = seedrandom(seed.value);
  console.info("apply-solutions", solutionApplicationDict.value)
  for (let k of Object.keys(solutionApplicationDict.value)) { 
    applySolution(k);
  }
}
function applySolution(key: string) { //@ts-ignore
  let vars = solutionApplicationDict.value[key] ? solutions.value[key] : [0, 0, 0, "*"];
  
  for (const d of getPoints()) {
    const r = randNumber(); //@ts-ignore
    // const r = 0 //@ts-ignore
    d.value = d.value - r
    //@ts-ignore
    if (d.zone === "suburb" && d.type === vars[3] && d.value) { //@ts-ignore
      d.value = d.value - vars[1];
    } //@ts-ignore
    if (d.zone === "urban" && d.type === vars[3] && d.value) { //@ts-ignore
      d.value = d.value - vars[2];
    } //@ts-ignore
    if (d.zone === "rural" && d.type === vars[3] && d.value) { //@ts-ignore
      d.value = d.value - vars[0];
    } //@ts-ignore
    if (d.zone === "suburb" && d.type === "*" && d.value) { //@ts-ignore
      d.value = d.value - (vars[1]/2);
    } //@ts-ignore
    if (d.zone === "urban" && d.type === "*" && d.value) { //@ts-ignore
      d.value = d.value - (vars[2]/2);
    } //@ts-ignore
    if (d.zone === "rural" && d.type === "*" && d.value) { //@ts-ignore
      d.value = d.value - (vars[0]/2);
    } //@ts-ignore
    console.info(d.value)
  }
}
function randNumber() {
  let alpha = 0;
  if(myrng !== null && seed.value !== "") {
    alpha = Math.round(myrng() * 100)/10
  }
  
  console.info(alpha, seed.value)
  return alpha;
}
function setupData() {
  for (const d of getPoints()) {
    const randIncrease = Math.random() > 0.5;
    // const c = randIncrease ? randNumber() : randNumber() * -1;
    const c = 0;
    if (d.zone === "suburb") { //@ts-ignore
      d.value = regionalSeed.value[1] + c;
    }
    if (d.zone === "urban") { //@ts-ignore
      d.value = regionalSeed.value[2] + c;
    }
    if (d.zone === "rural") { //@ts-ignore
      d.value = regionalSeed.value[0] + c;
    }
    if (d.zone === "water") { //@ts-ignore
      d.value = 0
    }
  }
}

function getAvg() { //@ts-ignore
  return getPoints().reduce((a, b) => a + (b.value || 0), 0) / getPoints().length || 0
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
  for (const i of getPoints()) {
    const svg = d3.select('svg');
    svg
      .append("circle")
      .attr('cx', i['x'])
      .attr('cy', i['y'])
      .attr('r', '2px')
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
  getMeanSAT()
  draw();
}

function getPoints() {
  return dataFlag.value === "voronai"
    ? voronaiData
    : initialSensorData

}
function draw() {
  const width = 432;
  const height = 324;
  const p = pValue.value

  const colorRange = colors.value.reduce((acc, curr) => { //@ts-ignore
    for (let i = 0; i < curr[1]; i++) {
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

  // console.info(colorRange, colors.value)

  const colorScale = d3
    .scaleSequential(d3.interpolateRgbBasis(colorRange))
    .domain(domain.value);

  const interpolateValue = (x: number, y: number) => {
    let totalWeight = 0;
    let interpolatedValue = 0;


    const points = getPoints()

    points.forEach((d) => {
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
      const scale = colorScale(interpolated);

      svg
        .append("rect")
        .attr("x", x)
        .attr("y", y)

        .attr("width", gridSize.value)
        .attr("height", gridSize.value)
        .style("opacity", 0.65)
        .style("fill", scale);
    }
  }
  if (displaySensors.value) {
    drawSensors()
  }
}

</script>

<template>
  <div>
    <h1>Configuration</h1>
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
          <h3>{{ key }}</h3>
          <input type="checkbox" id="checkbox" v-model="solutionApplicationDict[key]" @input="updateSolutionDict(key)">
        </div>

      </div>
      <div class="solutionValues">
        <h2>Solution Values</h2>
        <div v-for="(s, key) in solutions" :key="key">
          <h3>{{ key }}</h3>
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
          <option>2</option>
          <option>4</option>
          <option>8</option>
          <option>16</option>
          <option>32</option>
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
      <button @click="showSensors">show sensors</button>
      <button @click="hideSensors">hide sensors</button>
      <br>
      <h2>presets</h2>
      <button @click="applyGratefulDeadConfig">Grateful Dead</button>
      <button @click="applyGratefulDeadVoronaiConfig">Grateful Dead + Voronai Points</button>
      <button @click="applyJosephConradConfig">Joseph Conrad</button>
      <hr>
      <h2>Seed</h2>
      <input type="text" v-model="seed">
      <span>Average: {{ getAvg() }} </span>
      <div class="containerBox">
        <div class="foo" ref="chart">
          <img class="bgCity" src="../assets/bg-city-v3.png" />
          <img class="bgCity top" src="../assets/ui-overlay.png" />
          <div class="svg-container"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.colors div {
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
  display: grid;
  ;
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
  padding-bottom: 100%;
  /* aspect ratio */
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

#app>div {
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
}</style>
