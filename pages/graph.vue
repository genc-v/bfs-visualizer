<template>
  <div class="page-container">
    <div class="controls">
      <button @click="generateRandomGraph">Generate new Graph</button>
      <button v-if="!isBFSRunning" @click="startBFS">Start BFS</button>
      <button v-if="isBFSRunning" @click="pauseBFS">{{ isPaused.value ? 'Resume' : 'Pause' }}</button>
      <div>
        <label for="numNodes">Number of Nodes:</label>
        <input id="numNodes" type="number" v-model="numNodes" min="10" max="100" />
      </div>
      <div>
        <label for="bfsSpeed">BFS Speed (ms):</label>
        <input id="bfsSpeed" type="number" v-model="bfsSpeed" min="100" max="5000" step="100" />
      </div>
      <button @click="saveSettings">Save Settings</button>
    </div>
    <div class="gContainer">
      <div ref="graphContainer" class="graphContainer"></div>
    </div>
  </div>
</template>

<script setup>
import { Network } from 'vis-network/standalone/umd/vis-network.min.js';
import { DataSet } from 'vis-network/standalone/umd/vis-network.min.js';
import { ref, onMounted } from 'vue';

const graphContainer = ref(null);
let network = null;

let nodes = [];
let edges = [];
let edgeList = [];
let bfsQueue = [];
let visitedSet = new Set();
let isPaused = ref(false);
let isBFSRunning = ref(false);

// BFS Debugger state
const startNode = 1;
const steps = ref([]);
const currentStep = ref(0);

// Reactive variables for user inputs
const numNodes = ref(30);
const bfsSpeed = ref(1000);

// Function to generate a random connected graph
function generateRandomGraph() {
  // Reset BFS state
  bfsQueue = [];
  visitedSet.clear();
  steps.value = [];
  currentStep.value = 0;
  isPaused.value = false;
  isBFSRunning.value = false;

  const numEdges = Math.floor(Math.random() * numNodes.value) + numNodes.value;

  nodes = [];
  edges = [];

  for (let i = 0; i < numNodes.value; i++) {
    nodes.push({
      id: i + 1,
      label: `Node ${i + 1}`,
      shape: 'dot',
      size: 15,
      color: { background: 'gray' },
      font: { size: 14 },
    });
  }

  for (let i = 1; i < numNodes.value; i++) {
    edges.push({ id: `${i}-${i + 1}`, from: i, to: i + 1 });
  }

  while (edges.length < numEdges) {
    const from = Math.floor(Math.random() * numNodes.value) + 1;
    let to = Math.floor(Math.random() * numNodes.value) + 1;
    while (from === to || edges.some(edge => (edge.from === from && edge.to === to) || (edge.from === to && edge.to === from))) {
      to = Math.floor(Math.random() * numNodes.value) + 1;
    }
    edges.push({ id: `${from}-${to}`, from, to });
  }

  const data = { nodes: new DataSet(nodes), edges: new DataSet(edges) };

  const options = {
    autoResize: true,
    width: '100%',
    height: '100%',
    interaction: { hover: true },
    physics: {
      enabled: true,
      stabilization: true,
      barnesHut: {
        gravitationalConstant: -8000,
        springLength: 100,
        springConstant: 0.04,
      },
      solver: 'barnesHut',
    },
    edges: { color: '#000000', width: 2, smooth: { type: 'continuous', roundness: 0.5 } },
    nodes: { borderWidth: 3, borderWidthSelected: 4 },
  };

  if (network) {
    network.setData(data);
  } else {
    network = new Network(graphContainer.value, data, options);
  }

  edgeList = edges;
}

// Function to save settings
function saveSettings() {
  generateRandomGraph();
}

// BFS Traversal with distinct colors for active nodes and edges
function startBFS() {
  bfsQueue = [startNode];
  visitedSet.clear();
  steps.value = [];
  currentStep.value = 0;
  isPaused.value = false;
  isBFSRunning.value = true;

  const logStep = (message) => {
    steps.value.push({
      message,
      nodes: JSON.parse(JSON.stringify(network.body.data.nodes.get())),
      edges: JSON.parse(JSON.stringify(network.body.data.edges.get())),
    });
  };

  logStep(`Starting BFS from Node ${startNode}`);

  const processQueue = async () => {
    while (bfsQueue.length > 0) {
      if (isPaused.value) {
        await delay(100);
        continue;
      }

      const nodeId = bfsQueue.shift();
      if (!visitedSet.has(nodeId)) {
        logStep(`Processing Node ${nodeId}`);

        visitedSet.add(nodeId);
        // Current node in blue
        network.body.data.nodes.update({ id: nodeId, color: { background: 'blue' } });
        logStep(`Visited Node ${nodeId}`);

        const neighbors = edgeList.filter(edge => edge.from === nodeId || edge.to === nodeId);
        neighbors.forEach(edge => {
          const otherNode = edge.from === nodeId ? edge.to : edge.from;
          if (!visitedSet.has(otherNode) && !bfsQueue.includes(otherNode)) {
            bfsQueue.push(otherNode);
            logStep(`Queued Node ${otherNode} via Edge (${edge.from}-${edge.to})`);
            // Next nodes in yellow
            network.body.data.nodes.update({ id: otherNode, color: { background: 'yellow' } });
            network.body.data.edges.update({ id: edge.id, color: { color: 'blue' } });
            logStep(`Node ${otherNode} is set to yellow`);
          }
        });

        await delay(bfsSpeed.value);
      }
    }
    logStep('BFS Complete');
    isBFSRunning.value = false;
  };

  processQueue();
}

// Pause BFS
function pauseBFS() {
  isPaused.value = !isPaused.value;
}

// Go to the next step
function nextStep() {
  if (isBFSRunning.value) pauseBFS(); // Pause BFS if it is running
  if (currentStep.value < steps.value.length - 1) {
    currentStep.value++;
    applyStep(currentStep.value);
  }
}

// Go to the previous step
function prevStep() {
  if (isBFSRunning.value) pauseBFS(); // Pause BFS if it is running
  if (currentStep.value > 0) {
    currentStep.value--;
    applyStep(currentStep.value);
  }
}

// Apply a specific step
function applyStep(stepIndex) {
  const step = steps.value[stepIndex];
  network.body.data.nodes.clear();
  network.body.data.edges.clear();
  network.body.data.nodes.add(step.nodes);
  network.body.data.edges.add(step.edges);
}

// Utility to create a delay for visualization
function delay(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

onMounted(() => {
  generateRandomGraph();
});
</script>


<style scoped>
.page-container {
  display: flex;
  flex-direction: row-reverse;
  align-items: center;
  justify-content: center;
}
.controls {
  flex: 1;
}
#graphContainer {
  border: 2px solid #ccc;
  background-color: #f9f9f9;
  border-radius: 8px;
}

.bfs-debugger {
  margin-top: 20px;
  padding: 15px;
  border: 2px solid #ccc;
  background-color: #fff;
  border-radius: 8px;
}

.bfs-debugger .controls {
  display: flex;
  gap: 10px;
  margin-bottom: 10px;
}

.bfs-debugger .log ul {
  list-style-type: none;
  padding: 0;
}

.bfs-debugger .log li {
  padding: 5px;
  border-radius: 4px;
  margin: 2px 0;
}

.bfs-debugger .log li.active {
  background-color: #d3f8d3;
  font-weight: bold;
}
.gContainer {
  width: 50%;
  height: 80vh;
  position: relative; /* Ensures child can fill it */
  flex: 1;
}

.graphContainer {
  width: 100%;
  height: 100%;
  border: 2px solid #ccc;
  background-color: #f9f9f9;
  border-radius: 8px;
}
</style>
