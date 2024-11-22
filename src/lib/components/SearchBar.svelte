<!-- SearchBar.svelte -->
<script lang="ts">
    export let onSearch: (query: string) => void;
    export let isSearching = false;

    let searchQuery = '';

    function handleSubmit(e: Event) {
        e.preventDefault();
        onSearch(searchQuery);
    }
</script>

<form on:submit={handleSubmit} class="relative mb-4 w-full max-w-md">
    <div class="relative">
        <div class="pointer-events-none absolute inset-y-0 left-0 flex items-center pl-3">
            <svg class="h-5 w-5 text-gray-400" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
            </svg>
        </div>
        <input
            type="text"
            bind:value={searchQuery}
            placeholder="Search files..."
            class="block w-full rounded-lg border border-gray-300 bg-white py-2 pl-10 pr-12 text-sm text-gray-900 outline-none transition-colors placeholder:text-gray-500 hover:border-gray-400 focus:border-blue-500 focus:ring-1 focus:ring-blue-500"
        />
        {#if isSearching}
            <div class="absolute inset-y-0 right-0 flex items-center pr-3">
                <div class="h-4 w-4 animate-spin rounded-full border-2 border-gray-300 border-t-blue-600"></div>
            </div>
        {:else if searchQuery}
            <button
                type="button"
                on:click={() => { searchQuery = ''; }}
                class="absolute inset-y-0 right-0 flex items-center pr-3 text-gray-400 hover:text-gray-600"
            >
                <span class="sr-only">Clear search</span>
                <svg class="h-5 w-5" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                </svg>
            </button>
        {/if}
    </div>
</form>

<style>
    /* Prevent the search spinner from affecting layout */
    .animate-spin {
        animation: spin 1s linear infinite;
    }
    @keyframes spin {
        from { transform: rotate(0deg); }
        to { transform: rotate(360deg); }
    }
</style>
