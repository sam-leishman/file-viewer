<!-- BreadcrumbNav.svelte -->
<script lang="ts">
    interface BreadcrumbItem {
        name: string;
        handle: FileSystemDirectoryHandle;
    }

    export let breadcrumbs: BreadcrumbItem[];
    export let onNavigate: (index: number) => void;
    export let onBack: () => void;
    export let onChangeDirectory: () => void;
</script>

<div class="mb-4 flex items-center justify-between">
    <div class="flex items-center gap-2">
        <button
            on:click={onBack}
            disabled={breadcrumbs.length <= 1}
            class="rounded-md p-1 hover:bg-gray-100 disabled:opacity-50 disabled:hover:bg-transparent"
            aria-label="Go back to previous directory"
        >
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 text-gray-500" viewBox="0 0 24 24" fill="currentColor" aria-hidden="true">
                <path fill-rule="evenodd" d="M7.72 12.53a.75.75 0 010-1.06l7.5-7.5a.75.75 0 111.06 1.06L9.31 12l6.97 6.97a.75.75 0 11-1.06 1.06l-7.5-7.5z" clip-rule="evenodd" />
            </svg>
        </button>
        <div class="flex items-center gap-1 text-sm">
            {#each breadcrumbs as crumb, index}
                <button
                    on:click={() => onNavigate(index)}
                    class="hover:text-blue-600 {index === breadcrumbs.length - 1 ? 'font-medium text-gray-900' : 'text-gray-500'}"
                    aria-label={`Navigate to ${crumb.name}`}
                >
                    {crumb.name}
                </button>
                {#if index < breadcrumbs.length - 1}
                    <span class="text-gray-400" aria-hidden="true">/</span>
                {/if}
            {/each}
        </div>
    </div>
    <button
        on:click={onChangeDirectory}
        class="rounded-md bg-gray-100 px-3 py-1.5 text-sm text-gray-600 hover:bg-gray-200 focus:outline-none focus:ring-2 focus:ring-gray-500 focus:ring-offset-2"
        aria-label="Change current directory"
    >
        Change Directory
    </button>
</div>
