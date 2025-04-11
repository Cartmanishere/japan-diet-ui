<script>
  import { fade, slide } from 'svelte/transition'; // Import slide
  import { flip } from 'svelte/animate';
  import { onMount } from 'svelte'; // Import onMount
  import FoodItem from '$lib/components/FoodItem.svelte';
  import Pagination from '$lib/components/Pagination.svelte';
  import RestaurantSelector from '$lib/components/RestaurantSelector.svelte';
  import Search from '$lib/components/Search.svelte';
  import "../app.css";

  const API_BASE = "http://localhost:8080"

  const fetchFoodData = async (query = '', page = 1, limit = 5, restaurantId = null) => {
    console.log(`Fetching page ${page} for query: "${query}", restaurant: ${restaurantId || 'any'}`);

    let url = `/v1/api/foods`
    let requestData = {
      pagination: {
        page: page,
        limit: limit
      },
      name: query,
    }
    if (restaurantId != null && restaurantId != "-1") {
      requestData['restaurant_id'] = restaurantId;
    }
    try {
      const httpResponse = await fetch(API_BASE+url, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(requestData)
      });

      if (!httpResponse.ok) {
        console.log(await httpResponse.json())
        throw new Error(`HTTP error! status: ${httpResponse.status}`);
      }

      const jsonResp = await httpResponse.json();
      const { total, page, limit, items } = jsonResp.data;
      const totalPages = Math.ceil(total / limit);
      console.log(`Found ${total} items, returning page ${page}`);


      return {
        data: items.map(food => {
          return {
            ...food,
            id: food._id.$oid
          }
        }),
        currentPage: page,
        totalPages: totalPages,
        totalItems: total,
      };
    } catch (error) {
      console.error("Error fetching food data:", error);
      throw error; // Re-throw to be handled in the calling function
    }
  }

  async function fetchRestaurants() {
    let url = '/v1/api/restaurants'
    try {
      const httpResponse = await fetch(API_BASE + url)
      if (!httpResponse.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const jsonResp = await httpResponse.json();
      console.log("rest ", jsonResp);
      const { total, page, limit, items } = jsonResp.data;
      return items.map(
        restaurant => {
          return {
            ...restaurant,
            id: restaurant._id.$oid
          }
        }
      )
    } catch (error) {
      console.error("Error fetching food data:", error);
      throw error; // Re-throw to be handled in the calling function
    }
  }

  let searchTerm = '';
  let selectedRestaurant = '-1'; // Default to 'All Restaurants'
  let restaurants = [];
  let restaurantsLoading = true;
  let restaurantsError = null;
  let searchResults = [];
  let isLoading = false;
  let currentPage = 1;
  let totalPages = 1;
  let error = null; // For food search errors
  let expandedItemId = null; // ID of the currently expanded food item

  // Reactive statement to trigger search when searchTerm, currentPage, or selectedRestaurant changes
  $: {
    // Debounce search slightly
    const handler = setTimeout(() => {
      // Ensure restaurants have loaded before attempting search if a specific one is selected
      if (!restaurantsLoading) {
        searchFoods(currentPage);
      }
    }, 300); // Wait 300ms after interaction stops

    // Cleanup function for the timeout
    () => clearTimeout(handler); // Corrected function name
  }

  const searchFoods = async (page = 1) => {
    if (isLoading || (restaurantsLoading && selectedRestaurant !== 'r5')) return;
    isLoading = true;
    error = null;
    currentPage = page; // Update current page for food search

    try {
      // Pass selectedRestaurant to the fetch function
      const result = await fetchFoodData(searchTerm, currentPage, 5, selectedRestaurant);
      searchResults = result.data;
      totalPages = result.totalPages;
      // Ensure currentPage doesn't exceed totalPages after filtering
      if (currentPage > totalPages && totalPages > 0) {
        currentPage = totalPages;
        // Re-fetch if page number was adjusted (optional, depends on desired UX)
        // await searchFoods(currentPage);
      } else if (totalPages === 0) {
        currentPage = 1; // Reset to page 1 if no results
      }

    } catch (err) {
      console.error("Error fetching food data:", err);
      error = "Failed to load data. Please try again.";
      searchResults = [];
      totalPages = 1;
      currentPage = 1;
    } finally {
      isLoading = false;
    }
  }

  const handleSearchInput = ({ detail: { searchInput } }) => {
    searchTerm = searchInput;
    currentPage = 1; // Reset to first page on new text search
    // The reactive block will handle the search call
  }

  const handleRestaurantChange = ({ detail: { selectedRestaurant }}) => {
    selectedRestaurant = selectedRestaurant;
    currentPage = 1; // Reset to first page when restaurant changes
    // The reactive block will handle the search call
  }

  const toggleExpand = (itemId) => {
    expandedItemId = expandedItemId === itemId ? null : itemId;
  }

  const goToPage = ({ detail: { page }}) => {
    if (page >= 1 && page <= totalPages && page !== currentPage) {
      // The reactive block `$: searchFoods(currentPage)` will handle the search call
      // We just need to update currentPage
      currentPage = page;
    }
  }

  // Fetch restaurants when the component mounts
  onMount(async () => {
    try {
      restaurants = await fetchRestaurants();
      restaurants.push({ id: "-1", name: "All Restaurants"})
      console.log(restaurants);
      // Optionally trigger an initial search now that restaurants are loaded
      // searchFoods(1); // Uncomment if you want initial data based on default restaurant
    } catch (err) {
      console.error("Error fetching restaurants:", err);
      restaurantsError = "Could not load restaurants.";
    } finally {
      restaurantsLoading = false;
    }
  });

</script>

<div class="min-h-screen bg-gradient-to-br from-green-50 to-cyan-100 p-4 sm:p-8 font-sans">
  <div class="max-w-2xl mx-auto bg-white rounded-xl shadow-lg p-6">

    <h1 class="text-3xl font-bold text-center text-green-700 mb-6">
      Japan Diet
    </h1>


    <Search
      searchTerm={searchTerm}
      on:searchInput={handleSearchInput}
      on:enter={ () => { searchFoods(1) }}
    />


    <RestaurantSelector
      restaurantsLoading={restaurantsLoading}
      restaurantsError={restaurantsError}
      selectedRestaurant={selectedRestaurant}
      restaurants={restaurants}
      on:restaurantChange={handleRestaurantChange}
    />

    {#if isLoading}
      <div class="text-center py-4 text-gray-500">
        <svg class="animate-spin h-6 w-6 inline-block mr-2 text-green-600" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
          <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
          <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
        </svg>
        Loading food data...
      </div>
    {/if}

{#if error}
  <div class="text-center py-4 text-red-600 bg-red-100 border border-red-300 rounded-lg">
    {error}
  </div>
{/if}

{#if !isLoading && !error}
  {#if searchResults.length > 0}
    <ul class="space-y-3 mb-6">
      {#each searchResults as food, index}
        <FoodItem
          food={searchResults[index]}
          isExpanded={expandedItemId === searchResults[index].id}
          on:toggle={() => toggleExpand(searchResults[index].id)}
          />
        {/each}
      </ul>
    {:else if searchTerm && !isLoading}
      <p class="text-center text-gray-500 py-4">No results found for "{searchTerm}"{selectedRestaurant !== 'r5' ? ` at ${restaurants.find(r=>r.id===selectedRestaurant)?.name || 'selected restaurant'}` : ''}.</p>
    {:else if !searchTerm && !isLoading && !restaurantsLoading}
      <p class="text-center text-gray-500 py-4">Start typing to search for foods{selectedRestaurant !== 'r5' ? ` at ${restaurants.find(r=>r.id===selectedRestaurant)?.name || 'selected restaurant'}` : ''}.</p>
    {:else if restaurantsLoading}
      <!-- Optionally show a message while restaurants load initially -->
    <p class="text-center text-gray-500 py-4">Loading restaurant data...</p>
  {/if}
{/if}


{#if !isLoading && totalPages > 1}
  <Pagination
    currentPage={currentPage}
    totalPages={totalPages}
    on:pageChange={goToPage}
    />
  {/if}

  </div>
</div>

<style>
  /* Optional: Add global styles or Tailwind directives here if not using a separate CSS file */
  /* @tailwind base; */
  /* @tailwind components; */
  /* @tailwind utilities; */

  /* Ensure font is loaded if not default */
  /* @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap'); */
  /* body { font-family: 'Inter', sans-serif; } */

  /* Add custom styles if needed */
  :global(body) {
    font-family: sans-serif; /* Example fallback */
  }

</style>
