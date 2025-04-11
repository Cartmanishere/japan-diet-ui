<script>
  import { slide } from 'svelte/transition';
  import { createEventDispatcher } from 'svelte';
  import { onDestroy, afterUpdate } from 'svelte';
  import { Chart } from 'chart.js/auto'; // Import Chart.js
  import ChartDataLabels from 'chartjs-plugin-datalabels'; // Import the datalabels plugin

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
      // Register the plugin locally for this chart instance
      Chart.register(ChartDataLabels);
      // Create new chart
      chartInstance = new Chart(canvasEl.getContext('2d'), {
        type: 'pie',
        data: {
          labels: ['Protein (g)', 'Carbs (g)', 'Fat (g)'],
          datasets: [{
            label: 'Macros',
            data: [food.protein, food.carbs, food.fat],
            backgroundColor: [
              'rgb(255, 205, 86)', // Yellow
              'rgb(54, 162, 235)', // Blue
              'rgb(255, 99, 132)'  // Red
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
            },
            // Configure the datalabels plugin
            datalabels: {
              formatter: (value, ctx) => {
                // Display the raw value (grams)
                // Ensure value is not null/undefined before appending 'g'
                return value != null ? value + 'g' : '';
              },
              color: '#fff', // Color of the labels
              font: {
                weight: 'bold'
              },
              // Optional: Add background/border for better visibility on slices
              // backgroundColor: function(context) {
              //   return context.dataset.backgroundColor;
              // },
              // borderRadius: 4,
              // padding: 6
            }
          }
        }
        // Note: We registered the plugin above, so no need to list it in a separate 'plugins' array here
        // unless specifically overriding global registration for this instance.
      });
    } else {
      // If not expanded or canvas doesn't exist, destroy any existing chart
      if (chartInstance) {
        chartInstance.destroy();
        chartInstance = null;
        // Optional: Unregister if dynamically registering/unregistering is complex.
        // Chart.js usually handles cleanup with destroy, but explicit unregister
        // can prevent potential issues in complex scenarios.
        // Chart.unregister(ChartDataLabels);
      }
    }
  });

  onDestroy(() => {
    // Ensure chart is destroyed when component is removed
    if (chartInstance) {
      chartInstance.destroy();
    }
    // Optional: Explicit unregistration on component destroy if needed,
    // especially if registration issues were encountered.
    // Chart.unregister(ChartDataLabels);
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
