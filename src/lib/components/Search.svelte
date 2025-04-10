<script>
  import { createEventDispatcher } from 'svelte';

  export let searchTerm;

  const dispatch = createEventDispatcher();

  const handleSearchInput = (event) => {
    dispatch('searchInput', { searchInput: event.target.value })
  }

  const onEnter = (event) => {
    if (event.key === 'Enter') {
      dispatch('enter');
    }
  }

</script>

<div class="relative mb-6">
  <input
    type="search"
    bind:value={searchTerm}
    on:input={handleSearchInput}
    on:keydown={onEnter}
  placeholder="Search for a food (e.g., Apple)"
  class="w-full px-4 py-3 pr-10 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-green-500 focus:border-transparent transition duration-200"
  />
  <svg class="w-5 h-5 text-gray-400 absolute top-1/2 right-3 transform -translate-y-1/2 pointer-events-none" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
  </svg>
</div>
