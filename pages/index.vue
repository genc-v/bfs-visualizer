<template>
  <div class="page-container">
    <!-- Controls -->
    <div class="controls">
      <div class="control-buttons">
        <button class="primary-button" @click="generateRandomGraph">
          Generate New Graph
        </button>
        <button
          class="primary-button"
          v-if="!isBFSRunning && !isDFSRunning"
          @click="startBFS"
        >
          Start BFS
        </button>
        <button
          class="primary-button"
          v-if="isBFSRunning"
          @click="pauseBFS"
        >
          {{ isPaused.value ? "Resume" : "Pause" }}
        </button>
        <button
          class="primary-button"
          v-if="!isDFSRunning && !isBFSRunning"
          @click="startDFS"
        >
          Start DFS
        </button>
        <button
          class="primary-button"
          v-if="isDFSRunning"
          @click="pauseDFS"
        >
          {{ isPaused.value ? "Resume" : "Pause" }}
        </button>
      </div>

      <!-- Inputs -->
      <div class="control-inputs">
        <div class="input-group">
          <label for="numNodes">Number of Nodes:</label>
          <input
            id="numNodes"
            type="number"
            v-model="numNodes"
            min="10"
            max="100"
            class="input-field"
          />
        </div>
        <div class="input-group">
          <label for="speed">Traversal Speed (ms):</label>
          <input
            id="speed"
            type="number"
            v-model="traversalSpeed"
            min="100"
            max="5000"
            step="100"
            class="input-field"
          />
        </div>
      </div>

      <button class="primary-button primary-button--green" @click="saveSettings">Save Settings</button>
    </div>

    <!-- Graph -->
    <div class="gContainer">
      <div ref="graphContainer" class="graphContainer"></div>
    </div>
  </div>
</template>

<script setup>
// Importojme funksionet dhe klasat 
import { Network, DataSet } from "vis-network/standalone/umd/vis-network.min.js";

// Krijojme nje reference per vendin ku do te shfaqet grafi
const graphContainer = ref(null);
let network = null;

// Variablat per te ruajtur nyjet dhe lidhjet e grafit
let nodes = [];
let edges = [];

// Lista per DFS (kalimi ne thellesi)
let traversalStack = []; 

// Rradhe per BFS (kalimi ne gjersi)
let bfsQueue = []; 

// Nje grup per te shenuar nyjet e vizituara
let visitedSet = new Set();

// Kontrollojme nese algoritmi eshte ne pauze apo jo
let isPaused = ref(false);

// Kontrollojme nese BFS apo DFS eshte duke u ekzekutuar
let isBFSRunning = ref(false);
let isDFSRunning = ref(false);

// Numri i nyjeve ne graf dhe shpejtesia e kalimit
const numNodes = ref(30);
const traversalSpeed = ref(1000);

// Funksion per te krijuar nje grafik randomte
function generateRandomGraph() {
  // Rivendosim te gjitha vlerat e meparshme
  bfsQueue = [];
  traversalStack = [];
  visitedSet.clear();
  isPaused.value = false;
  isBFSRunning.value = false;
  isDFSRunning.value = false;

  // Llogarisim numrin e lidhjeve ne menyre rastesore
  const numEdges = Math.floor(Math.random() * numNodes.value) + numNodes.value;

  // Krijojme nyje te reja me emra unike
  nodes = Array.from({ length: numNodes.value }, (_, i) => ({
    id: i + 1, // Identifikues unik per cdo nyje
    label: `Node ${i + 1}`, // Emri qe shfaqet ne grafik
    shape: "dot", // Forma e nyjes
    size: 15, // Madhesia e nyjes
    color: { background: "gray" }, // Ngjyra fillestare
    font: { size: 14 }, // Madhesia e tekstit
  }));

  // Krijojme lidhje bazuar ne nyjet
  edges = [];
  for (let i = 1; i < numNodes.value; i++) {
    edges.push({ id: `${i}-${i + 1}`, from: i, to: i + 1 });
  }

  // Shto me shume lidhje rastesore per te kompletuar grafin
  while (edges.length < numEdges) {
    const from = Math.floor(Math.random() * numNodes.value) + 1;
    let to = Math.floor(Math.random() * numNodes.value) + 1;
    // Sigurohemi qe lidhjet te jene te vlefshme dhe te mos perseriten
    while (
      from === to ||
      edges.some(
        (edge) =>
          (edge.from === from && edge.to === to) ||
          (edge.from === to && edge.to === from)
      )
    ) {
      to = Math.floor(Math.random() * numNodes.value) + 1;
    }
    edges.push({ id: `${from}-${to}`, from, to });
  }

  // Pergatisim te dhenat dhe opsionet per grafik
  const data = { nodes: new DataSet(nodes), edges: new DataSet(edges) };
  const options = {
    autoResize: true,
    width: "100%",
    height: "100%",
    interaction: { hover: true },
    physics: {
      enabled: true,
      stabilization: true,
      barnesHut: {
        gravitationalConstant: -8000,
        springLength: 100,
        springConstant: 0.04,
      },
    },
    edges: { color: "#000000", width: 2 },
    nodes: { borderWidth: 3, borderWidthSelected: 4 },
  };

  // Nese grafi ekziston, rifreskoje perndryshe, krijoje nga fillimi
  if (network) {
    network.setData(data);
  } else {
    network = new Network(graphContainer.value, data, options);
  }
}

// Funksion per te ruajtur ndryshimet dhe rifreskuar grafin
function saveSettings() {
  generateRandomGraph();
}

// Funksion qe pret per nje kohe te caktuar (per animacione)
function delay(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

// Algoritmi BFS (kalimi ne gjersi)
function startBFS() {
  bfsQueue = [1]; // Nisim nga nyja e pare
  visitedSet.clear();
  isPaused.value = false;
  isBFSRunning.value = true;

  const processQueue = async () => {
    while (bfsQueue.length > 0) {
      if (isPaused.value) {
        await delay(100);
        continue;
      }

      // Marrim nyjen e pare nga rradha dhe e shenojme si te vizituar
      const nodeId = bfsQueue.shift();
      if (!visitedSet.has(nodeId)) {
        visitedSet.add(nodeId);
        network.body.data.nodes.update({
          id: nodeId,
          color: { background: "blue" }, // Shenojme nyjen me ngjyre blu
        });

        // Gjejme nyjet fqinje dhe i shtojme ne radhe
        edges
          .filter((edge) => edge.from === nodeId || edge.to === nodeId)
          .forEach((edge) => {
            const otherNode = edge.from === nodeId ? edge.to : edge.from;
            if (
              !visitedSet.has(otherNode) &&
              !bfsQueue.includes(otherNode)
            ) {
              bfsQueue.push(otherNode);
              network.body.data.nodes.update({
                id: otherNode,
                color: { background: "yellow" },
              });
              network.body.data.edges.update({
                id: edge.id,
                color: { color: "blue" },
              });
            }
          });

        await delay(traversalSpeed.value); // Prisni per animacionin
      }
    }
    isBFSRunning.value = false; // Perfundohet algoritmi
  };

  processQueue();
}

// Nderpret/pauzon BFS
function pauseBFS() {
  isPaused.value = !isPaused.value;
}

// Algoritmi DFS (kalimi ne thellesi)
function startDFS() {
  traversalStack = [1];
  visitedSet.clear();
  isPaused.value = false;
  isDFSRunning.value = true;

  const processStack = async () => {
    while (traversalStack.length > 0) {
      if (isPaused.value) {
        await delay(100);
        continue;
      }

      const nodeId = traversalStack.pop();
      if (!visitedSet.has(nodeId)) {
        visitedSet.add(nodeId);
        network.body.data.nodes.update({
          id: nodeId,
          color: { background: "red" }, // Shenojme nyjen me ngjyre te kuqe
        });

        edges
          .filter((edge) => edge.from === nodeId || edge.to === nodeId)
          .forEach((edge) => {
            const otherNode = edge.from === nodeId ? edge.to : edge.from;
            if (!visitedSet.has(otherNode)) {
              traversalStack.push(otherNode);
              network.body.data.nodes.update({
                id: otherNode,
                color: { background: "orange" },
              });
              network.body.data.edges.update({
                id: edge.id,
                color: { color: "red" },
              });
            }
          });

        await delay(traversalSpeed.value);
      }
    }
    isDFSRunning.value = false;
  };

  processStack();
}

// Nderpret/pauzon DFS
function pauseDFS() {
  isPaused.value = !isPaused.value;
}

// Kur ngarkohet faqja, krijo grafin automatikisht
onMounted(() => {
  generateRandomGraph();
});
</script>


<style>
.page-container {
  display: flex;
  flex-direction: row-reverse;
  gap: 20px;
  padding: 20px;
}

.controls {
  flex: 1;
  max-width: 300px;
  padding: 20px;
  background-color: #ffffff;
  border: 2px solid #cccccc;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.control-buttons {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-bottom: 20px;
}

.control-inputs {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.input-group {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.input-field {
  padding: 8px 10px;
  font-size: 14px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.primary-button {
  padding: 10px 15px;
  font-size: 14px;
  color: #ffffff;
  background-color: #0056b3;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease;
  width: 100%;
  margin-top: 20px;
}

.primary-button--green {
  background-color: #159932;
}

.gContainer {
  flex: 2;
  height: 80vh;
  position: relative;
}

.graphContainer {
  width: 100%;
  height: 100%;
  border: 2px solid #cccccc;
  background-color: #f9f9f9;
  border-radius: 8px;
}
</style>
