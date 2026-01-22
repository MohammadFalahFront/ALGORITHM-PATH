<script setup>
import { ref, computed, onMounted, onUnmounted } from "vue";

const algorithms = [
  { name: "Dijkstra's Algorithm", value: "dijkstra" },
  { name: "Depth-First Search (DFS)", value: "dfs" },
  { name: "Breadth-First Search (BFS)", value: "bfs" },
];

const isMouseDown = ref(false);
const speed = ref(50);
const min = 0;
const max = 100;
const showdashboard = ref(true);

const startNodeIndex = ref(null);
const endNodeIndex = ref(null);
const isDraggingStart = ref(false);
const isDraggingEnd = ref(false);
const isDrawingWall = ref(false);
const displayAlgorithmInfo = ref(false);
const selectedAlgorithm = ref(algorithms[0].value);

const restAlltesting = () => {
  for (let i = 1; i <= 35 * 50; i++) {
    const square = document.getElementById(`square-${i}`);
    if (square) {
      square.classList.remove("visited-node");
      square.classList.remove("path-node");
    }
  }
};
const handleMouseDown = (cell) => {
  isMouseDown.value = true;
  
  if (cell === startNodeIndex.value) {
    isDraggingStart.value = true;
  } else if (cell === endNodeIndex.value) {
    isDraggingEnd.value = true;
  } else {
    isDrawingWall.value = true;
    toggleWall(cell);
  }
};

const handleMouseEnter = (cell) => {
  if (!isMouseDown.value) return;

  if (isDraggingStart.value) {
    if (cell !== endNodeIndex.value) {
      startNodeIndex.value = cell;
      removeWall(cell);
    }
  } else if (isDraggingEnd.value) {
    if (cell !== startNodeIndex.value) {
      endNodeIndex.value = cell;
      removeWall(cell);
    }
  } else if (isDrawingWall.value) {
    if (cell !== startNodeIndex.value && cell !== endNodeIndex.value) {
      toggleWall(cell);
    }
  }
};

const handleMouseUp = () => {
  isMouseDown.value = false;
  isDraggingStart.value = false;
  isDraggingEnd.value = false;
  isDrawingWall.value = false;
};

function toggleWall(square_number) {
  const square = document.getElementById(`square-${square_number}`);
  if (square_number !== startNodeIndex.value && square_number !== endNodeIndex.value) {
    square.classList.toggle("wall-node");
  }
}

function removeWall(square_number) {
  const square = document.getElementById(`square-${square_number}`);
  square.classList.remove("wall-node");
}

const sliderBackgroundStyle = computed(() => {
  const percentage = ((speed.value - min) / (max - min)) * 100;
  return {
    background: `linear-gradient(to right, 
      #d946ef 0%, 
      #a855f7 ${percentage}%, 
      rgba(255, 255, 255, 0.1) ${percentage}%, 
      rgba(255, 255, 255, 0.1) 100%
    )`
  };
});

function removeallWalls() {
  for (let i = 1; i <= 35 * 50; i++) {
    const square = document.getElementById(`square-${i}`);
    if (square) {
      square.classList.remove("wall-node");
    }
  }
}

// Generate random Maze
const ROWS = 35;
const COLS = 50;

const getIndex = (row, col) => row * COLS + col;
let wallsToAnimate = [];

const generateMaze = () => {
  removeallWalls();
  
  wallsToAnimate = [];
  
  addOuterWalls();

  recursiveDivision(1, ROWS - 2, 1, COLS - 2);
  
  animateWalls();
};

const addOuterWalls = () => {
  for (let r = 0; r <= ROWS; r++) {
    if(r === 0 || r === ROWS -1) {
        for(let c = 1; c <= COLS; c++) wallsToAnimate.push(getIndex(r, c));
    } else {
        wallsToAnimate.push(getIndex(r, 1));
        wallsToAnimate.push(getIndex(r, COLS));
    }
  }
};

// --- Recursive Division ---
const recursiveDivision = (rowStart, rowEnd, colStart, colEnd) => {
  if (rowEnd < rowStart || colEnd < colStart) return;

  let horizontal = rowEnd - rowStart > colEnd - colStart;

  if (horizontal) {
    let possibleRows = [];
    for (let r = rowStart; r <= rowEnd; r += 2) possibleRows.push(r);
    
    if (possibleRows.length === 0) return;
    
    let randomRowIndex = Math.floor(Math.random() * possibleRows.length);
    let currentRow = possibleRows[randomRowIndex];

    let possibleHoles = [];
    for (let c = colStart - 1; c <= colEnd + 1; c += 2) possibleHoles.push(c);
    
    let randomHoleIndex = Math.floor(Math.random() * possibleHoles.length);
    let holeCol = possibleHoles[randomHoleIndex];

    for (let c = colStart - 1; c <= colEnd + 1; c++) {
      if (c !== holeCol) {
        wallsToAnimate.push(getIndex(currentRow, c));
      }
    }

    recursiveDivision(rowStart, currentRow - 2, colStart, colEnd);
    recursiveDivision(currentRow + 2, rowEnd, colStart, colEnd);

  } else {

    let possibleCols = [];
    for (let c = colStart; c <= colEnd; c += 2) possibleCols.push(c);
    
    if (possibleCols.length === 0) return;

    let randomColIndex = Math.floor(Math.random() * possibleCols.length);
    let currentCol = possibleCols[randomColIndex];

    let possibleHoles = [];
    for (let r = rowStart - 1; r <= rowEnd + 1; r += 2) possibleHoles.push(r);
    
    let randomHoleIndex = Math.floor(Math.random() * possibleHoles.length);
    let holeRow = possibleHoles[randomHoleIndex];

    for (let r = rowStart - 1; r <= rowEnd + 1; r++) {
      if (r !== holeRow) {
        wallsToAnimate.push(getIndex(r, currentCol));
      }
    }

    recursiveDivision(rowStart, rowEnd, colStart, currentCol - 2);
    recursiveDivision(rowStart, rowEnd, currentCol + 2, colEnd);
  }
};

const animateWalls = () => {
  for (let i = 0; i < wallsToAnimate.length; i++) {
    setTimeout(() => {
      const index = wallsToAnimate[i];
      if (index !== startNodeIndex.value && index !== endNodeIndex.value) {
        const square = document.getElementById(`square-${index}`);
        if(square) square.classList.add("wall-node");
      }
    }, i * 10); 
  }
};

// --- Algorithm Implementation ---

const startSimulation = () => {
  restAlltesting();
  displayAlgorithmInfo.value = true;
    if(selectedAlgorithm.value === 'bfs') {
        bfsAlgorithm();
    } else if(selectedAlgorithm.value === 'dijkstra') {
        dijkstraAlgorithm();
    } else if(selectedAlgorithm.value === 'dfs') {
        dfsAlgorithm();
    } else {
        displayAlgorithmInfo.value = false;
    }
};

let visitedNodesInOrder = [];

const dfsAlgorithm = async () => {
  let stack = [startNodeIndex.value];
  let visited = new Set();
  let parents = {};
  visitedNodesInOrder = [];

  while (stack.length > 0) {
    let currentNode = stack.pop();

    if (visited.has(currentNode)) continue;

    visited.add(currentNode);
    visitedNodesInOrder.push(currentNode);

    if (currentNode === endNodeIndex.value) {
      await animateVisitedNodes();
      await animatePath(parents);
      displayAlgorithmInfo.value = false;
      return;
    }

    let neighbors = getNeighbors(currentNode);
    
    neighbors.sort(() => Math.random() - 0.5);

    for (let neighbor of neighbors) {
      const isWall = document.getElementById(`square-${neighbor}`).classList.contains('wall-node');
      
      if (!visited.has(neighbor) && !isWall) {
        parents[neighbor] = currentNode;
        stack.push(neighbor);
      }
    }
  }

  await animateVisitedNodes();
  alert("No path found!");
};

const dijkstraAlgorithm = async () => {
  let distances = {};
  let parents = {};
  let visited = new Set();
  let pq = [{ node: startNodeIndex.value, distance: 0 }];
  
  visitedNodesInOrder = [];

  for (let i = 0; i < ROWS * COLS; i++) {
    distances[i] = Infinity;
  }
  distances[startNodeIndex.value] = 0;

  while (pq.length > 0) {
    pq.sort((a, b) => a.distance - b.distance);
    let currentObj = pq.shift();
    let currentNode = currentObj.node;

    if (visited.has(currentNode)) continue;

    visited.add(currentNode);
    visitedNodesInOrder.push(currentNode);

    if (currentNode === endNodeIndex.value) {
      await animateVisitedNodes();
      await animatePath(parents);
      displayAlgorithmInfo.value = false;
      return;
    }

    let neighbors = getNeighbors(currentNode);
    for (let neighbor of neighbors) {
      const isWall = document.getElementById(`square-${neighbor}`).classList.contains('wall-node');
      
      if (!isWall && !visited.has(neighbor)) {
        let newDist = distances[currentNode] + 1;
        if (newDist < distances[neighbor]) {
          distances[neighbor] = newDist;
          parents[neighbor] = currentNode;
          pq.push({ node: neighbor, distance: newDist });
        }
      }
    }
  }

  await animateVisitedNodes();
  alert("No path found!");
};

const bfsAlgorithm = async () => {
  let queue = [];
  let visited = new Set(); 
  let parents = {};

  queue.push(startNodeIndex.value);
  visited.add(startNodeIndex.value);
  visitedNodesInOrder = [];

  while (queue.length > 0) {
    let currentNode = queue.shift();
    visitedNodesInOrder.push(currentNode);

    if (currentNode === endNodeIndex.value) {
      await animateVisitedNodes();
      await animatePath(parents);
      displayAlgorithmInfo.value = false;
      return;
    }

    let neighbors = getNeighbors(currentNode);

    for (let neighbor of neighbors) {
      const isWall = document.getElementById(`square-${neighbor}`).classList.contains('wall-node');
      
      if (!visited.has(neighbor) && !isWall) {
        visited.add(neighbor);
        parents[neighbor] = currentNode; 
        queue.push(neighbor); 
      }
    }
  }
  
  await animateVisitedNodes();
  alert("No path found!");
};

const getNeighbors = (index) => {
  const neighbors = [];
  const row = Math.floor(index / COLS);
  const col = index % COLS;

  // أعلى
  if (row > 0) neighbors.push(index - COLS);
  // أسفل
  if (row < ROWS - 1) neighbors.push(index + COLS);
  // يسار
  if (col > 0) neighbors.push(index - 1);
  // يمين
  if (col < COLS - 1) neighbors.push(index + 1);

  return neighbors;
};

const animateVisitedNodes = async () => {

  for (let i = 0; i < visitedNodesInOrder.length; i++) {
    const nodeIndex = visitedNodesInOrder[i];
    if (nodeIndex !== startNodeIndex.value && nodeIndex !== endNodeIndex.value) {
      const square = document.getElementById(`square-${nodeIndex}`);
      square.classList.add('visited-node');
    }
    await new Promise(r => setTimeout(r, 100 - speed.value)); 
  }
};

const animatePath = async (parents) => {
  let path = [];
  let current = endNodeIndex.value;

  while (current !== startNodeIndex.value) {
    path.unshift(current);
    current = parents[current];
  }
  
  for (let i = 0; i < path.length; i++) {
    const nodeIndex = path[i];
    if (nodeIndex !== endNodeIndex.value) {
      const square = document.getElementById(`square-${nodeIndex}`);
      square.classList.remove('visited-node');
      square.classList.add('path-node');
    }
    await new Promise(r => setTimeout(r, 50));
  }
};

onMounted(() => {
  window.addEventListener('mouseup', handleMouseUp);
  
  let rand1 = Math.floor(Math.random() * 1750);
  let rand2 = Math.floor(Math.random() * 1750);
  
  while(rand1 === rand2) {
     rand2 = Math.floor(Math.random() * 1750);
  }

  startNodeIndex.value = rand1;
  endNodeIndex.value = rand2;
});

onUnmounted(() => {
  window.removeEventListener('mouseup', handleMouseUp);
});
</script>

<template>
<main class="grid grid-cols-1 xl:grid-cols-5 gap-0 h-[100vh] background-container overflow-hidden relative">
    <div class="stars"></div>
    <div class="stars2"></div>

  <div class="flex w-6 h-5 items-center justify-between flex-col absolute top-6 left-6 z-50 cursor-pointer xl:hidden" 
    @click="showdashboard = !showdashboard">
  <span class="h-0.5 w-full bg-white transition-all duration-300" :class="showdashboard ? 'rotate-45 translate-y-2' : ''"></span>
  <span class="h-0.5 w-full bg-white transition-all duration-300" :class="showdashboard ? 'opacity-0' : ''"></span>
  <span class="h-0.5 w-full bg-white transition-all duration-300" :class="showdashboard ? '-rotate-45 -translate-y-2.5' : ''"></span>
</div>

    <section class="fixed inset-y-0 left-0 z-40 w-[280px] transform transition-transform duration-300 ease-in-out xl:relative xl:translate-x-0 xl:col-span-1 xl:w-auto xl:flex dashboard flex-col items-center py-8 text-white flex"
        :class="showdashboard ? 'translate-x-0' : '-translate-x-full'">
      <div v-if="displayAlgorithmInfo" class="w-full h-full bg-black/10 transition z-10 absolute top-0"></div>
      <span><svg xmlns="http://www.w3.org/2000/svg" width="80" height="80" viewBox="0 0 48 48">
          <path fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"
            d="m23.979 36.55l5.244-2.711v5.685L24.07 42.5zm5.244-2.712L16.924 27.89l.156-8.601l7.105 3.553l12.403-6.327V35.65l-11.936-5.703v-5.75l5.209-2.578l1.707.914v5.19l-6.916-3.526m6.918-1.663l-.037-3.44" />
          <path fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"
            d="M24.07 42.5L6.327 33.54V8.344l5.62-2.843l12.163 6.082l12.158-5.966l5.404 2.639v24.79L36.59 35.65" />
          <path fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"
            d="m6.327 8.343l17.796 8.734l17.55-8.823m-17.55 8.823l.063 5.765m-.207 13.708l-12.383-6.104l.094-13.992l5.391 2.836" />
        </svg></span>
      <h1 class="mt-4 text-[clamp(1rem,1.5vw,2rem)] font-bold">ALGORITHM PATH</h1>
      <p class="mt-4 text-xs">Advanced Pathfinding Visualizer</p>
      <hr class="border-white/20 w-[90%] mt-1" />
      <div class="flex items-center gap-2 mt-6 w-[85%]">
        <span><svg xmlns="http://www.w3.org/2000/svg" width="23" height="23" viewBox="0 0 24 24">
            <g fill="none" stroke="currentColor" stroke-linejoin="round" stroke-width="1.5">
              <path stroke-linecap="round"
                d="M10.5 2v2m3-2v2M8 6.5H6m2 3H6m12-3h-2m2 3h-2M13.333 4h-2.666C9.41 4 8.78 4 8.39 4.39C8 4.782 8 5.41 8 6.668v2.666c0 1.257 0 1.886.39 2.277c.391.39 1.02.39 2.277.39h2.666c1.257 0 1.886 0 2.277-.39c.39-.391.39-1.02.39-2.277V6.667c0-1.257 0-1.886-.39-2.276C15.219 4 14.59 4 13.333 4" />
              <path
                d="M3.617 21.924c.184.076.417.076.883.076s.699 0 .883-.076a1 1 0 0 0 .54-.541C6 21.199 6 20.966 6 20.5s0-.699-.076-.883a1 1 0 0 0-.541-.54C5.199 19 4.966 19 4.5 19s-.699 0-.883.076a1 1 0 0 0-.54.541C3 19.801 3 20.034 3 20.5s0 .699.076.883a1 1 0 0 0 .541.54Zm7.5 0c.184.076.417.076.883.076s.699 0 .883-.076a1 1 0 0 0 .54-.541c.077-.184.077-.417.077-.883s0-.699-.076-.883a1 1 0 0 0-.541-.54C12.699 19 12.466 19 12 19s-.699 0-.883.076a1 1 0 0 0-.54.541c-.077.184-.077.417-.077.883s0 .699.076.883a1 1 0 0 0 .541.54Z" />
              <path stroke-linecap="round"
                d="M12 19v-7m-7.5 7c0-1.404 0-2.107.337-2.611a2 2 0 0 1 .552-.552C5.893 15.5 6.596 15.5 8 15.5h8c1.404 0 2.107 0 2.611.337c.218.146.406.334.552.552c.337.504.337 1.207.337 2.611" />
              <path
                d="M18.617 21.924c.184.076.417.076.883.076s.699 0 .883-.076a1 1 0 0 0 .54-.541c.077-.184.077-.417.077-.883s0-.699-.076-.883a1 1 0 0 0-.541-.54C20.199 19 19.966 19 19.5 19s-.699 0-.883.076a1 1 0 0 0-.54.541c-.077.184-.077.417-.077.883s0 .699.076.883a1 1 0 0 0 .541.54Z" />
            </g>
          </svg></span>
        <h2>Algorithm Select</h2>
      </div>
      <select name="algorithm" id="algorithm" v-model="selectedAlgorithm" :disabled="displayAlgorithmInfo"
        class="mt-2 w-[90%] bg-transparent border border-white/20 rounded-md p-2 text-sm outline-none cursor-pointer">
        <option v-for="value in algorithms" :value="value.value" class="bg-black/80">
          {{ value.name }}
        </option>
      </select>
      <div class="flex items-center gap-2 mt-6 w-[85%]">
        <span><svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 48 48">
            <defs>
              <mask id="SVGPOcIjc5v">
                <g fill="none" stroke="#fff" stroke-linejoin="round" stroke-width="4">
                  <path fill="#555"
                    d="M30.297 18.779s-3.23 9.102-4.764 10.691a4 4 0 0 1-5.754-5.557c1.534-1.59 10.518-5.134 10.518-5.134Z" />
                  <path stroke-linecap="round"
                    d="M38.85 38.85A20.94 20.94 0 0 0 45 24c0-11.598-9.402-21-21-21S3 12.402 3 24c0 5.799 2.35 11.049 6.15 14.85M24 4v4m14.845 3.142l-3.108 2.517m6.785 13.574l-3.897-.9m-33.148.9l3.898-.9m-.22-15.191l3.108 2.517" />
                </g>
              </mask>
            </defs>
            <path fill="currentColor" d="M0 0h48v48H0z" mask="url(#SVGPOcIjc5v)" />
          </svg></span>
        <h2>Simulation Speed</h2>
      </div>
      <p class="w-[90%] text-end text-[.7rem]"><span class="bg-purple-200/10 p-1 rounded-lg"
          :class="speed > 50 ? 'text-red-500' : speed <= 50 && speed >= 30 ? 'text-green-400' : 'text-blue-400'">
          {{ speed }}%
        </span></p>
      <div
        class="relative w-[90%] h-4 mt-2 rounded-full border border-white/10 bg-white/[0.02] flex items-center px-2 shadow-lg backdrop-blur-sm">
        <input :disabled="displayAlgorithmInfo" type="range" :min="min" :max="max" v-model="speed" :style="sliderBackgroundStyle"
          class="w-full h-1.5 rounded-full outline-none z-10 appearance-none cursor-pointer">

        <div class="absolute left-2 right-2 h-1.5 bg-slate-800/50 rounded-full -z-0 pointer-events-none"></div>
      </div>

      <button @click="startSimulation()" :disabled="displayAlgorithmInfo"
        class="mt-6 w-[90%] flex justify-center items-center text-white py-2 px-4 rounded-lg shadow-lg gap-1 start-button">
        <svg xmlns="http://www.w3.org/2000/svg" width="27" height="27"
          viewBox="0 0 512 512"><path fill="currentColor" fill-rule="evenodd"
            d="M421.935 90.066c2.286 2.287 5.356 7.973 4.013 21.398c-1.323 13.221-6.731 30.687-16.845 51.505a306 306 0 0 1-3.77 7.492v42.31c39.823-61.993 54.037-115.44 31.687-137.79c-22.529-22.528-76.652-7.907-139.277 32.646l20.879 12.054c10.535-6.46 20.717-12.073 30.411-16.783c20.817-10.114 38.283-15.522 51.504-16.845c13.425-1.343 19.112 1.727 21.398 4.013M405.333 341.54v-42.31c39.823 61.993 54.037 115.44 31.687 137.79c-22.536 22.536-76.689 7.897-139.341-32.687l20.877-12.053c10.559 6.477 20.763 12.104 30.477 16.823c20.817 10.114 38.283 15.522 51.505 16.845c13.424 1.343 19.111-1.727 21.397-4.013c2.287-2.287 5.357-7.973 4.013-21.398c-1.322-13.221-6.73-30.687-16.845-51.505a306 306 0 0 0-3.77-7.492m-211.888 50.74l20.876 12.053c-62.651 40.584-116.804 55.223-139.34 32.687c-22.35-22.35-8.136-75.795 31.686-137.788v42.31a309 309 0 0 0-3.769 7.49c-10.114 20.818-15.522 38.284-16.845 51.505c-1.343 13.425 1.727 19.111 4.013 21.398c2.287 2.286 7.973 5.356 21.398 4.013c13.221-1.323 30.687-6.731 51.505-16.845c9.713-4.719 19.917-10.346 30.476-16.823M74.981 74.981c-22.35 22.35-8.136 75.795 31.686 137.788v-42.311a308 308 0 0 1-3.769-7.489c-10.114-20.818-15.522-38.284-16.845-51.505c-1.343-13.425 1.727-19.111 4.013-21.398c2.287-2.286 7.973-5.356 21.398-4.013c13.221 1.323 30.687 6.73 51.505 16.845c9.693 4.709 19.874 10.322 30.41 16.782l20.878-12.054C151.633 67.074 97.51 52.453 74.981 74.981M256 108.16l128 73.901v147.801l-128 73.901l-128-73.901V182.06zm85.333 123.132v73.937l-64 36.95v-73.937zm-106.666 36.95v73.937l-64-36.95v-73.937zM256 231.291l-63.969-36.933L256 157.426l63.969 36.932z"
            clip-rule="evenodd" />
        </svg> Start Simulation
      </button>

      <div class="flex justify-center items-center gap-2 w-[90%] mt-6">

        <button @click="removeallWalls()" :disabled="displayAlgorithmInfo"
        class="group flex-1 flex justify-center items-center text-blue-300 py-2 rounded-xl gap-2 text-xs font-medium border border-blue-500/50 bg-purple-900/20 backdrop-blur-md shadow-[0_0_10px_rgba(168,85,247,0.2)] hover:shadow-[0_0_20px_rgba(168,85,247,0.6)] hover:border-blue-400 hover:text-white hover:bg-purple-800/40 transition-all duration-300 active:scale-95">

        <svg class="transition-transform duration-300 group-hover:rotate-[-360deg]" xmlns="http://www.w3.org/2000/svg"
          width="18" height="18" viewBox="0 0 24 24">
          <path fill="currentColor"
            d="M9.825 20.7q-2.575-.725-4.2-2.837T4 13q0-1.425.475-2.713t1.35-2.362q.275-.3.675-.313t.725.313q.275.275.288.675t-.263.75q-.6.775-.925 1.7T6 13q0 2.025 1.188 3.613t3.062 2.162q.325.1.538.375t.212.6q0 .5-.35.788t-.825.162m4.35 0q-.475.125-.825-.175t-.35-.8q0-.3.213-.575t.537-.375q1.875-.6 3.063-2.175T18 13q0-2.5-1.75-4.25T12 7h-.075l.4.4q.275.275.275.7t-.275.7t-.7.275t-.7-.275l-2.1-2.1q-.15-.15-.212-.325T8.55 6t.063-.375t.212-.325l2.1-2.1q.275-.275.7-.275t.7.275t.275.7t-.275.7l-.4.4H12q3.35 0 5.675 2.325T20 13q0 2.725-1.625 4.85t-4.2 2.85" />
        </svg>

        REST PATH
      </button>
      <button @click="restAlltesting()" :disabled="displayAlgorithmInfo"
        class="group flex-1 flex justify-center items-center text-red-300 py-2 rounded-xl gap-2 text-xs font-medium border border-red-500/50 bg-purple-900/20 backdrop-blur-md shadow-[0_0_10px_rgba(168,85,247,0.2)] hover:shadow-[0_0_20px_rgba(168,85,247,0.6)] hover:border-red-400 hover:text-white hover:bg-purple-800/40 transition-all duration-300 active:scale-95">

      <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24"><!-- Icon from Material Symbols by Google - https://github.com/google/material-design-icons/blob/master/LICENSE --><path fill="currentColor" d="M7 21q-.825 0-1.412-.587T5 19V6H4V4h5V3h6v1h5v2h-1v13q0 .825-.587 1.413T17 21zM17 6H7v13h10zM9 17h2V8H9zm4 0h2V8h-2zM7 6v13z"/></svg>

        NEW TESTING
      </button>

      </div>

      <button @click="generateMaze()" :disabled="displayAlgorithmInfo"
        class="group mt-4 w-[90%] flex justify-center items-center text-purple-300 py-2 rounded-xl gap-2 text-xs font-medium border border-purple-500/50 bg-purple-900/20 backdrop-blur-md shadow-[0_0_10px_rgba(168,85,247,0.2)] hover:shadow-[0_0_20px_rgba(168,85,247,0.6)] hover:border-purple-400 hover:text-white hover:bg-purple-800/40 transition-all duration-300 active:scale-95">

        <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18"
          viewBox="0 0 24 24"><g fill="none" stroke="currentColor" stroke-linejoin="round" stroke-width="1.5">
            <path stroke-linecap="round"
              d="M14 2.5V7c0 1.886 0 2.828.586 3.414S16.114 11 18 11m3.5 4H15m-5 0h5m0 0v6.5M2.5 10H5c1.886 0 2.828 0 3.414-.586S9 7.886 9 6m-6.5 9H6" />
            <path
              d="M2.5 12c0-4.478 0-6.718 1.391-8.109S7.521 2.5 12 2.5c4.478 0 6.718 0 8.109 1.391S21.5 7.521 21.5 12c0 4.478 0 6.718-1.391 8.109S16.479 21.5 12 21.5c-4.478 0-6.718 0-8.109-1.391S2.5 16.479 2.5 12Z" />
          </g>
        </svg>
        GENERATE MAZE
      </button>
      <hr class="border-white/20 w-[90%] mt-6" />
      <div class="w-[90%] mt-4">
        <h3 class="text-sm font-semibold">Information System</h3>
        <p class="mt-2 text-[.7rem] text-white/70">Use the controls above to select an algorithm, adjust the simulation
          speed, and start or reset the pathfinding visualization. You can also generate a random maze to see how
          different algorithms perform in complex environments.</p>
      </div>
    </section>

    <section class="xl:col-start-2 xl:col-end-6 col-span-1 grid-area flex justify-center items-center flex-col select-none h-full relative z-0">
      <div class="flex justify-between items-center mb-2 2xl:w-2/4 w-3/4">
        <span class="flex justify-center items-center text-white text-sm">
          <span class="w-3 h-3 bg-green-700 rounded-sm border border-white/20 mr-2"></span>
          Start Node
        </span>
        <span class="flex justify-center items-center text-white text-sm">
          <span class="w-3 h-3 bg-red-700 rounded-sm border border-white/20 mr-2"></span>
          End Node
        </span>
        <span class="flex justify-center items-center text-white text-sm">
          <span class="w-3 h-3 bg-purple-500/30 rounded-sm border border-white/20 mr-2"></span>
          Wall Node
        </span>
        <span class="flex justify-center items-center text-white text-sm">
          <span class="w-3 h-3 bg-[#4299e199] rounded-sm border border-white/20 mr-2"></span>
          Scan Node
        </span>
        <span class="flex justify-center items-center text-white text-sm">
          <span class="w-3 h-3 bg-[#fbbf24] rounded-sm border border-white/20 mr-2"></span>
          Final Path Node
        </span>

      </div>
      <div
        class="w-3/4 h-3/4 border-2 border-[#8b5cf6]/30 rounded-lg grid grid-cols-[repeat(50,1fr)] grid-rows-[repeat(35,1fr)]">
        <span class="border border-r border-b border-white/10 bg-black/20 square" v-for="sq in 35 * 50" :key="sq"
          :id="`square-${sq}`" 
          :class="{
            'start-node': sq === startNodeIndex,
            'end-node': sq === endNodeIndex
          }"
          @mouseenter="handleMouseEnter(sq)" 
          @mousedown.prevent="handleMouseDown(sq)"></span>
      </div>
    </section>
  </main>
</template>

<style>
.background-container {
  background: radial-gradient(circle at top right, #3b1f54, #000000);
  position: relative;
  overflow: hidden;
}

.dashboard {
  background: linear-gradient(180deg, rgba(20, 30, 48, 0.7) 0%, rgba(36, 59, 85, 0.4) 100%);
  backdrop-filter: blur(16px) saturate(120%);
  -webkit-backdrop-filter: blur(16px) saturate(120%);
  border-right: 1px solid rgba(255, 255, 255, 0.1);
  box-shadow: 5px 0 25px rgba(0, 0, 0, 0.2);
}

.stars,
.stars2 {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: transparent;
  z-index: 0;
}

.stars::after {
  content: " ";
  position: absolute;
  top: 0;
  left: 0;
  width: 1px;
  height: 1px;
  background: transparent;
  box-shadow:
    10vw 10vh #fff,
    20vw 50vh #fff,
    30vw 80vh #fff,
    80vw 10vh #fff,
    95vw 40vh #fff,
    40vw 20vh #fff,
    50vw 80vh #fff,
    10vw 90vh #fff,
    60vw 30vh #fff,
    75vw 60vh #fff,
    25vw 25vh #fff,
    85vw 85vh #fff,
    15vw 65vh #fff,
    55vw 15vh #fff,
    35vw 95vh #fff,
    70vw 50vh #fff,
    90vw 70vh #fff,
    05vw 35vh #fff;

  animation: twinkle 5s infinite alternate;
}

.stars2::after {
  content: " ";
  position: absolute;
  top: 0;
  left: 0;
  width: 2px;
  height: 2px;
  background: transparent;
  box-shadow:
    15vw 15vh rgba(255, 255, 255, 0.8),
    35vw 55vh rgba(255, 255, 255, 0.8),
    65vw 25vh rgba(255, 255, 255, 0.8),
    85vw 75vh rgba(255, 255, 255, 0.8);

  animation: twinkle 7s infinite alternate-reverse;
}

@keyframes twinkle {
  0% {
    opacity: 0.3;
    transform: translateY(0);
  }

  100% {
    opacity: 1;
    transform: translateY(-10px);
  }
}

.grid-area {
  position: relative;
  z-index: 1;
}

.dashboard svg {
  filter: drop-shadow(0 0 5px rgba(255, 255, 255, 0.5));
}

.dashboard h1 {
  filter: drop-shadow(0 0 5px rgba(255, 255, 255, 0.5));
  background: radial-gradient(circle at top right, #8b5cf6, #d946ef);
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  color: transparent;
}

.start-button {
  background: linear-gradient(90deg, #8b5cf6, #d946ef);
  box-shadow: 0 2px 10px rgba(217, 70, 239, 0.4);
  transition: background 0.3s ease, box-shadow 0.3s ease;
}

.start-button:hover {
  background: linear-gradient(90deg, #d946ef, #8b5cf6);
  box-shadow: 0 4px 15px rgba(217, 70, 239, 0.6);
}

span.square:hover {
  background-color: rgba(139, 92, 246, 0.3);
  cursor: pointer;
  transition: background-color 0.1s ease;
}

.wall-node {
  background-color: rgba(139, 92, 246, 0.3) !important;
}

.start-node {
  background-color: rgb(22 163 74) !important;
  transform: scale(1.1);
  box-shadow: 0 0 10px #22c55e; 
  border: none;
  cursor: grab;
}

.start-node:active {
  cursor: grabbing;
  transform: scale(0.9); 
}

.end-node {
  background-color: rgb(220 38 38) !important;
  transform: scale(1.1);
  box-shadow: 0 0 10px #ef4444; 
  border: none;
  cursor: grab;
}

.end-node:active {
  cursor: grabbing;
  transform: scale(0.9); 
}

input[type=range]::-webkit-slider-thumb {
  -webkit-appearance: none;
  height: 20px;
  width: 20px;
  border-radius: 50%;
  background: linear-gradient(to bottom right, #e879f9, #a855f7);
  margin-top: 0px; 
  border: 2px solid rgba(255, 255, 255, 0.5);
  box-shadow: 0 0 15px rgba(217, 70, 239, 0.8);
  transition: transform 0.1s;
}

input[type=range]:active::-webkit-slider-thumb {
  transform: scale(1.1);
  box-shadow: 0 0 20px rgba(217, 70, 239, 1);
}

input[type=range] {
  -webkit-appearance: none; 
}

.visited-node {
  background-color: rgba(66, 153, 225, 0.6) !important; 
  animation: pop 0.3s ease-out;
}

.path-node {
  background-color: #fbbf24 !important; 
  animation: pop 0.3s ease-out;
  box-shadow: 0 0 10px #fbbf24;
}

@keyframes pop {
  0% { transform: scale(0.3); }
  50% { transform: scale(1.2); }
  100% { transform: scale(1); }
}
</style>