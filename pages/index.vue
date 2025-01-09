<template>
  <div>
    <input
      type="range"
      min="1"
      max="1000"
      v-model="animationSpeed"
      style="width: 100%; margin-bottom: 10px;"
    />
    <p>Animation Speed: {{ animationSpeed }}ms</p>
    <div
      style="
        padding: 0 16px;
        display: flex;
        min-height: 100vh;
        align-items: flex-end;
        position: relative;
      "
    >
      <div
        v-for="(item, index) in numbers"
        :key="index"
        :style="{
          height: (item / limit) * 100 + 'vh',
          width: barWidth + 'px',
          background: highlighted.includes(index) ? 'blue' : 'red',
          zIndex: highlighted.includes(index) ? '100' : '1',
          position: 'absolute',
          bottom: 0,
          left: index * barWidth + 'px',
          transform: swapping.includes(index) ? 'translateX(' + swapOffsets[index] + 'px)' : '',
          transition: 'transform ' + animationSpeed + 'ms ease',
        }"
      ></div>
    </div>
  </div>
</template>
<script setup>
import { ref, onMounted } from 'vue';

const numbers = ref([]);
const length = 50;
const limit = 3000;
const highlighted = ref([]);
const swapping = ref([]);
const animationSpeed = ref(500);
const barWidth = 20; // Width of each bar in pixels
const swapOffsets = ref(new Array(length).fill(0));

// Initialize the array with random numbers
for (let index = 0; index < length; index++) {
  numbers.value.push(Math.floor(Math.random() * limit));
}

// Helper function to swap two indices in the array
const swap = async (array, i, j) => {
  // Highlight the bars being swapped
  highlighted.value = [i, j];
  swapping.value = [i, j];

  // Calculate the offsets for the animation
  swapOffsets.value[i] = (j - i) * barWidth;
  swapOffsets.value[j] = (i - j) * barWidth;

  // Wait for the animation to complete
  await sleep(animationSpeed.value);

  // Perform the actual swap
  [array[i], array[j]] = [array[j], array[i]];

  // Reset the swapping state
  swapOffsets.value[i] = 0;
  swapOffsets.value[j] = 0;
  swapping.value = [];
  highlighted.value = [];

  await sleep(animationSpeed.value);
};

// Bubble Sort Implementation with Animation
const bubbleSort = async (array) => {
  let n = array.length;

  for (let i = 0; i < n - 1; i++) {
    for (let j = 0; j < n - 1 - i; j++) {
      highlighted.value = [j, j + 1];

      if (array[j] > array[j + 1]) {
        await swap(array, j, j + 1); // Swap bars
      }
    }
  }
};
0
// QuickSort Implementation with Animation
const quickSort = async (array, left = 0, right = array.length - 1) => {
  if (left >= right) return;

  const pivotIndex = await partition(array, left, right);
  await quickSort(array, left, pivotIndex - 1);
  await quickSort(array, pivotIndex + 1, right);
};

// Partition function for QuickSort
const partition = async (array, left, right) => {
  const pivot = array[right];
  let i = left;

  for (let j = left; j < right; j++) {
    highlighted.value = [i, j, right];

    if (array[j] < pivot) {
      await swap(array, i, j);
      i++;
    }
  }

  await swap(array, i, right);
  return i;
};

// Sleep function for animation timing
const sleep = (ms) => new Promise((resolve) => setTimeout(resolve, ms));

// Start sorting when the component mounts
onMounted(() => {
  // Use QuickSort as the default sorting method
  bubbleSort(numbers.value);
});
</script>

