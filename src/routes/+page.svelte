<script>
  import { fade, slide } from 'svelte/transition'; // Import slide
  import { flip } from 'svelte/animate';
  import { onMount } from 'svelte'; // Import onMount
  import "../app.css";

  // --- Mock API ---
  // Mock Restaurant Data
  const mockRestaurants = [
    { id: 'r1', name: 'The Green Leaf Cafe' },
    { id: 'r2', name: 'Ocean Blue Grill' },
    { id: 'r3', name: 'Mountain Top Diner' },
    { id: 'r4', name: 'City Bistro Express' },
    { id: 'r5', name: 'All Restaurants' }, // Option for no specific restaurant
  ];

  async function fetchRestaurants() {
    console.log('Fetching restaurants...');
    // Simulate network delay
    await new Promise(resolve => setTimeout(resolve, 200));
    console.log('Restaurants fetched.');
    // In a real app, handle potential errors here
    return mockRestaurants;
  }
  // Replace this with your actual API call
  const mockFoodDatabase = [
    { id: 1, name: 'Apple', servingSize: '1 medium', calories: 95, protein: 0.5, carbs: 25, fat: 0.3 },
    { id: 2, name: 'Banana', servingSize: '1 medium', calories: 105, protein: 1.3, carbs: 27, fat: 0.4 },
    { id: 3, name: 'Chicken Breast', servingSize: '100g', calories: 165, protein: 31, carbs: 0, fat: 3.6 },
    { id: 4, name: 'Broccoli', servingSize: '1 cup chopped', calories: 55, protein: 3.7, carbs: 11.2, fat: 0.6 },
    { id: 5, name: 'Salmon', servingSize: '100g fillet', calories: 208, protein: 20, carbs: 0, fat: 13 },
    { id: 6, name: 'Rice', servingSize: '1 cup cooked', calories: 205, protein: 4.3, carbs: 45, fat: 0.4 },
    { id: 7, name: 'Almonds', servingSize: '28g (approx. 23)', calories: 164, protein: 6, carbs: 6, fat: 14 },
    { id: 8, name: 'Egg', servingSize: '1 large', calories: 78, protein: 6, carbs: 0.6, fat: 5 },
    { id: 9, name: 'Spinach', servingSize: '1 cup raw', calories: 7, protein: 0.9, carbs: 1.1, fat: 0.1 },
    { id: 10, name: 'Sweet Potato', servingSize: '1 medium', calories: 86, protein: 1.6, carbs: 20, fat: 0.1 },
    { id: 11, name: 'Avocado', servingSize: '1/2 medium', calories: 160, protein: 2, carbs: 9, fat: 15 },
    { id: 12, name: 'Orange', servingSize: '1 medium', calories: 62, protein: 1.2, carbs: 15, fat: 0.2 },
    { id: 13, name: 'Greek Yogurt', servingSize: '100g', calories: 59, protein: 10, carbs: 3.6, fat: 0.4 },
    { id: 14, name: 'Quinoa', servingSize: '1 cup cooked', calories: 222, protein: 8, carbs: 39, fat: 3.6 },
    { id: 15, name: 'Tofu', servingSize: '100g', calories: 76, protein: 8, carbs: 2.7, fat: 4.8 },
    { id: 16, name: 'Peanut Butter', servingSize: '2 tbsp', calories: 190, protein: 7, carbs: 8, fat: 16 },
    { id: 17, name: 'Milk', servingSize: '1 cup (240ml)', calories: 103, protein: 8, carbs: 12, fat: 2.4 },
    { id: 18, name: 'Oats', servingSize: '1/2 cup dry (40g)', calories: 150, protein: 5, carbs: 27, fat: 3 },
    { id: 19, name: 'Lentils', servingSize: '1 cup cooked', calories: 230, protein: 18, carbs: 40, fat: 0.8 },
    { id: 20, name: 'Beef Steak', servingSize: '100g', calories: 250, protein: 26, carbs: 0, fat: 15 },
  ];

  // Updated fetchFoodData to accept restaurantId (simulation)
  async function fetchFoodData(query = '', page = 1, limit = 5, restaurantId = null) {
    console.log(`Fetching page ${page} for query: "${query}", restaurant: ${restaurantId || 'any'}`);
    // Simulate network delay
    await new Promise(resolve => setTimeout(resolve, 300));

    const lowerCaseQuery = query.toLowerCase();
    // --- Filtering Simulation ---
    // In a real API, the backend would handle filtering by restaurantId
    // Here, we'll just log it and use the same mock data for simplicity.
    // If restaurantId is provided (and not 'all'), you might filter mockFoodDatabase
    // based on some imaginary restaurant-specific menu.
    let baseData = [...mockFoodDatabase];
    if (restaurantId && restaurantId !== 'r5') { // 'r5' is 'All Restaurants'
        // Example: Simulate fewer items or different items for a specific restaurant
        // baseData = mockFoodDatabase.filter(food => food.id % 2 === (restaurantId === 'r1' ? 0 : 1)); // Just an example
        console.log(`Simulating filter for restaurant ${restaurantId}`);
    }

    const filteredData = query
      ? baseData.filter(food => food.name.toLowerCase().includes(lowerCaseQuery))
      : baseData; // Return all (potentially restaurant-filtered) if query is empty

    const totalItems = filteredData.length;
    const totalPages = Math.ceil(totalItems / limit);
    const startIndex = (page - 1) * limit;
    const endIndex = startIndex + limit;
    const paginatedData = filteredData.slice(startIndex, endIndex);

    console.log(`Found ${totalItems} items, returning page ${page}/${totalPages}`);
    return {
      data: paginatedData,
      currentPage: page,
      totalPages: totalPages,
      totalItems: totalItems,
    };
  }
  // --- End Mock API ---

  let searchTerm = '';
  let selectedRestaurant = 'r5'; // Default to 'All Restaurants'
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


  async function searchFoods(page = 1) {
    // Don't search if restaurants are still loading and a specific one is needed (optional check)
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

  function handleSearchInput(event) {
    searchTerm = event.target.value;
    currentPage = 1; // Reset to first page on new text search
    // The reactive block will handle the search call
  }

  function handleRestaurantChange(event) {
    selectedRestaurant = event.target.value;
    currentPage = 1; // Reset to first page when restaurant changes
    // The reactive block will handle the search call
  }

  function toggleExpand(itemId) {
    expandedItemId = expandedItemId === itemId ? null : itemId;
  }

 function goToPage(page) {
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
      Food Nutrition Lookup
    </h1>

    <div class="relative mb-6">
      <input
        type="search"
        bind:value={searchTerm}
        on:input={handleSearchInput}
        on:keydown={(event) => { if (event.key === 'Enter') searchFoods(1); }}
        placeholder="Search for a food (e.g., Apple)"
        class="w-full px-4 py-3 pr-10 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-green-500 focus:border-transparent transition duration-200"
      />
      <svg class="w-5 h-5 text-gray-400 absolute top-1/2 right-3 transform -translate-y-1/2 pointer-events-none" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
     </svg>
   </div>

   <!-- Restaurant Selector -->
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
   <!-- End Restaurant Selector -->


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
          {#each searchResults as food (food.id)}
            <li
              in:fade={{ duration: 200, delay: 100 }}
              animate:flip={{ duration: 300 }}
              class="bg-white rounded-lg border border-gray-200 overflow-hidden transition-shadow duration-200 hover:shadow-md"
            >
              <!-- {@const isExpanded = expandedItemId === food.id} <-- Removed -->

              <!-- Clickable Header -->
              <button
                on:click={() => toggleExpand(food.id)}
                class="w-full flex justify-between items-center p-4 text-left focus:outline-none focus:bg-gray-50 transition duration-150"
                aria-expanded={expandedItemId === food.id}
                aria-controls={`details-${food.id}`}
              >
                <div class="flex-grow mr-4">
                  <h3 class="text-lg font-semibold text-green-800">{food.name}</h3>
                  {#if food.servingSize}
                    <p class="text-sm text-gray-600 mt-1">
                      <span class="font-medium">Serving:</span> {food.servingSize}
                    </p>
                  {/if}
                  <p class="text-sm text-gray-600 mt-1">
                    <span class="font-medium">Calories:</span> {food.calories} kcal
                  </p>
                </div>
                <svg class="w-5 h-5 text-gray-500 transform transition-transform duration-200 flex-shrink-0" class:rotate-180={expandedItemId === food.id} xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7" />
                </svg>
              </button>

              <!-- Expandable Details -->
              {#if expandedItemId === food.id}
                {@const totalMacros = food.protein + food.carbs + food.fat}
                {@const proteinPercent = totalMacros > 0 ? (food.protein / totalMacros) * 100 : 0}
                {@const carbsPercent = totalMacros > 0 ? (food.carbs / totalMacros) * 100 : 0}
                {@const fatPercent = totalMacros > 0 ? (food.fat / totalMacros) * 100 : 0}
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
      <nav class="flex justify-center items-center space-x-2 mt-6" aria-label="Pagination">
        <button
          on:click={() => goToPage(currentPage - 1)}
          disabled={currentPage === 1}
          class="px-3 py-1 rounded-md text-sm font-medium focus:outline-none focus:ring-2 focus:ring-offset-1 focus:ring-green-500 transition disabled:opacity-50 disabled:cursor-not-allowed"
          class:bg-green-600={currentPage !== 1}
          class:text-white={currentPage !== 1}
          class:bg-gray-200={currentPage === 1}
          class:text-gray-500={currentPage === 1}
        >
          Previous
        </button>

        {#each Array(totalPages) as _, i}
          {@const pageNum = i + 1}
          <button
            on:click={() => goToPage(pageNum)}
            class="px-3 py-1 rounded-md text-sm font-medium focus:outline-none focus:ring-2 focus:ring-offset-1 focus:ring-green-500 transition"
            class:bg-green-600={pageNum === currentPage}
            class:text-white={pageNum === currentPage}
            class:bg-gray-100={pageNum !== currentPage}
            class:text-gray-700={pageNum !== currentPage}
            class:hover:bg-gray-200={pageNum !== currentPage}
            aria-current={pageNum === currentPage ? 'page' : undefined}
          >
            {pageNum}
          </button>
        {/each}

        <button
          on:click={() => goToPage(currentPage + 1)}
          disabled={currentPage === totalPages}
          class="px-3 py-1 rounded-md text-sm font-medium focus:outline-none focus:ring-2 focus:ring-offset-1 focus:ring-green-500 transition disabled:opacity-50 disabled:cursor-not-allowed"
          class:bg-green-600={currentPage !== totalPages}
          class:text-white={currentPage !== totalPages}
          class:bg-gray-200={currentPage === totalPages}
          class:text-gray-500={currentPage === totalPages}
        >
          Next
        </button>
      </nav>
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
