<script>
  import { slide } from 'svelte/transition';
  import { createEventDispatcher } from 'svelte';
  import { onDestroy, afterUpdate } from 'svelte';
  import { Chart } from 'chart.js/auto'; // Import Chart.js

  export let food; // The food object
  export let isExpanded = false; // Whether this item is currently expanded

  const dispatch = createEventDispatcher();

  let canvasEl; // To bind to the canvas element
  let chartInstance = null; // To hold the Chart.js instance

  const handleToggle = () => {
    // Dispatch an event instead of directly modifying parent state
    dispatch('toggle');
  }

  afterUpdate(() => {
    // Ensure canvas element exists and isExpanded is true before creating
    if (isExpanded && canvasEl) {
      // Destroy previous chart if it exists (e.g., if food prop changed while expanded)
      if (chartInstance) {
        chartInstance.destroy();
      }
      // Create new chart
      chartInstance = new Chart(canvasEl.getContext('2d'), {
        type: 'pie',
        data: {
          labels: ['Protein (g)', 'Carbs (g)', 'Fat (g)'],
          datasets: [{
            label: 'Macros',
            data: [food.protein, food.carbs, food.fat],
            backgroundColor: [
              'rgb(255, 205, 86)',
              'rgb(54, 162, 235)',
              'rgb(255, 99, 132)'
            ],
            hoverOffset: 4
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false, // Adjust as needed for layout
          plugins: {
            legend: {
              position: 'top',
            },
            tooltip: {
              callbacks: {
                label: function(context) {
                  let label = context.label || '';
                  if (label) {
                    label += ': ';
                  }
                  if (context.parsed !== null) {
                    // Calculate percentage for tooltip
                    const total = context.dataset.data.reduce((a, b) => a + b, 0);
                    const value = context.parsed;
                    const percentage = total > 0 ? ((value / total) * 100).toFixed(1) : 0;
                    label += `${value}g (${percentage}%)`;
                  }
                  return label;
                }
              }
            }
          }
        }
      });
    } else {
      // If not expanded or canvas doesn't exist, destroy any existing chart
      if (chartInstance) {
        chartInstance.destroy();
        chartInstance = null;
      }
    }
  });

  onDestroy(() => {
    // Ensure chart is destroyed when component is removed
    if (chartInstance) {
      chartInstance.destroy();
    }
  });
</script>

<li class="bg-white rounded-lg border border-gray-200 overflow-hidden transition-shadow duration-200 hover:shadow-md">
  <button
    on:click={handleToggle}
    class="w-full flex justify-between items-center p-4 text-left focus:outline-none focus:bg-gray-50 transition duration-150"
    aria-expanded={isExpanded}
    aria-controls={`details-${food.id}`}
    >
    <!-- ... (rest of the button content using 'food' prop) ... -->
    <div class="flex-grow mr-4">
      <h3 class="text-lg font-semibold text-green-800">{food.name}</h3>
      {#if food.serving}
        <p class="text-sm text-gray-600 mt-1">
          <span class="font-medium">Serving:</span> {food.serving.size} {food.serving.unit}
        </p>
      {/if}
      <p class="text-sm text-gray-600 mt-1">
        <span class="font-medium">Calories:</span> {food.calories} kcal
      </p>
    </div>
    <svg class="w-5 h-5 text-gray-500 transform transition-transform duration-200 flex-shrink-0" class:rotate-180={isExpanded}>
      <!-- ... svg path ... -->
    </svg>
  </button>

  {#if isExpanded}
    <div transition:slide={{ duration: 200 }} id={`details-${food.id}`} class="p-4 border-t border-gray-200 bg-gray-50">
      <div class="relative h-64 w-full"> <!-- Adjust height/width as needed -->
        <canvas bind:this={canvasEl}></canvas>
      </div>
    </div>
  {/if}
</li>
