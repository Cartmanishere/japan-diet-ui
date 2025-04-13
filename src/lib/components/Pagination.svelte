<script>
// @ts-nocheck

  import { createEventDispatcher } from 'svelte';

  const MAX_PAGES = 5; // Show fewer page numbers directly

  export let currentPage;
  export let totalPages;

  const dispatch = createEventDispatcher();
  export let onPageChange; // Callback function to handle page changes

  // Optional: Add a prop for styling the buttons
  export let buttonClass = "px-3 py-1 rounded-md text-sm font-medium focus:outline-none focus:ring-2 focus:ring-offset-1 focus:ring-green-500 transition";
  export let activeClass = "bg-green-600 text-white";
  export let inactiveClass = "bg-gray-200 text-gray-500";
  export let hoverClass = "hover:bg-gray-200";

  let pageNumbers = [];


  $: {
    // Calculate the start and end page numbers for the visible range
    let startPage = Math.max(1, currentPage - Math.floor(MAX_PAGES / 2));
    let endPage = Math.min(totalPages, startPage + MAX_PAGES - 1);

    // Adjust startPage if we're near the end
    if (endPage - startPage + 1 < MAX_PAGES) {
      startPage = Math.max(1, endPage - MAX_PAGES + 1);
    }

    pageNumbers = Array.from({ length: endPage - startPage + 1 }, (_, i) => startPage + i);
  }

  const goToPage = (page) => {
    if (page >= 1 && page <= totalPages) {
      dispatch('pageChange', { page: page});
    }
  }

</script>

{#if totalPages > 1}
  <nav class="flex justify-center items-center space-x-2 mt-6" aria-label="Pagination">
    <button
      on:click={() => goToPage(currentPage - 1)}
      disabled={currentPage === 1}
      class="{buttonClass} {currentPage === 1 ? inactiveClass : ''} {currentPage !== 1 ? activeClass : ''}"
      >
      Previous
    </button>

    {#each pageNumbers as pageNum, i}
    <button
      on:click={() => goToPage(pageNum)}
      class="{buttonClass} {pageNum === currentPage ? activeClass : ''} {pageNum !== currentPage ? hoverClass : ''}"
      >
      {pageNum}
    </button>
  {/each}

<button
  on:click={() => goToPage(currentPage + 1)}
  disabled={currentPage === totalPages}
  class="{buttonClass} {currentPage === totalPages ? inactiveClass : ''} {currentPage !== totalPages ? activeClass : ''}"
  >
  Next
</button>
  </nav>
{/if}
