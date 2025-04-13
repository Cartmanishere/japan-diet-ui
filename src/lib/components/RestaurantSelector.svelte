<script>
// @ts-nocheck
  import { createEventDispatcher } from 'svelte';

  export let restaurantsLoading;
  export let restaurantsError;
  export let selectedRestaurant;
  export let restaurants;

  const dispatch = createEventDispatcher();

  const handleRestaurantChange = (event) => {
    dispatch('restaurantChange', { selectedRestaurant: event.target.value })
  }

</script>

<div class="mb-6">
  <label for="restaurant-select" class="block text-sm font-medium text-gray-700 mb-1">Select Restaurant:</label>
  {#if restaurantsLoading}
    <div class="w-full px-4 py-3 border border-gray-200 rounded-lg bg-gray-100 text-gray-500">Loading restaurants...</div>
  {:else if restaurantsError}
    <div class="w-full px-4 py-3 border border-red-300 rounded-lg bg-red-100 text-red-700">{restaurantsError}</div>
  {:else}
    <div class="relative">
      <select
        id="restaurant-select"
        bind:value={selectedRestaurant}
        on:change={handleRestaurantChange}
        class="w-full px-4 py-3 pr-10 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-green-500 focus:border-transparent transition duration-200 appearance-none bg-white"
        disabled={restaurantsLoading || !!restaurantsError}
        >
        <!-- <option value="" disabled>Select a restaurant</option> -->
        {#each restaurants as restaurant (restaurant.id)}
          <option value={restaurant.id}>{restaurant.name}</option>
        {/each}
      </select>
      <svg class="w-5 h-5 text-gray-400 absolute top-1/2 right-3 transform -translate-y-1/2 pointer-events-none" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 20 20">
        <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M6 8l4 4 4-4"/>
      </svg>
    </div>
  {/if}
</div>
