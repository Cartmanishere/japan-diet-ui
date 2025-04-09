<script>
  import { slide } from 'svelte/transition';
  import { createEventDispatcher } from 'svelte';

  export let food; // The food object
  export let isExpanded = false; // Whether this item is currently expanded

  const dispatch = createEventDispatcher();

  // Calculate derived values inside the component if they only depend on the food prop
  $: totalMacros = food.protein + food.carbs + food.fat;
  $: proteinPercent = totalMacros > 0 ? (food.protein / totalMacros) * 100 : 0;
  $: carbsPercent = totalMacros > 0 ? (food.carbs / totalMacros) * 100 : 0;
  $: fatPercent = totalMacros > 0 ? (food.fat / totalMacros) * 100 : 0;

  function handleToggle() {
    // Dispatch an event instead of directly modifying parent state
    dispatch('toggle');
  }
</script>

<li class="bg-white rounded-lg border border-gray-200 overflow-hidden transition-shadow duration-200 hover:shadow-md">
  <button
    on:click={handleToggle}
    class="w-full flex justify-between items-center p-4 text-left focus:outline-none focus:bg-gray-50 transition duration-150"
    aria-expanded={isExpanded}
    aria-controls={`details-${food.id}`}
    >
    <!-- ... (rest of the button content using 'food' prop) ... -->
    <svg class="w-5 h-5 text-gray-500 transform transition-transform duration-200 flex-shrink-0" class:rotate-180={isExpanded}>
      <!-- ... svg path ... -->
    </svg>
  </button>

  {#if isExpanded}
    <div transition:slide={{ duration: 200 }} id={`details-${food.id}`} class="p-4 border-t border-gray-200 bg-gray-50">
      <h4 class="text-md font-semibold text-gray-700 mb-2">Macro Breakdown (grams):</h4>
      <div class="text-sm text-gray-600 space-y-1">
        <div><strong>Protein:</strong> {food.protein}g ({proteinPercent.toFixed(1)}%)</div>
        <div><strong>Carbs:</strong> {food.carbs}g ({carbsPercent.toFixed(1)}%)</div>
        <div><strong>Fat:</strong> {food.fat}g ({fatPercent.toFixed(1)}%)</div>
      </div>
    </div>
  {/if}
</li>
