<!-- FileList.svelte -->
<script lang="ts">
    import FileIcon from './FileIcon.svelte';

    interface FileItem {
        name: string;
        type: 'file' | 'directory';
        handle: FileSystemHandle;
        path?: string;
    }

    export let contents: FileItem[] = [];
    export let selectedFile: FileItem | null = null;
    export let isLoading = false;
    export let error: string | null = null;
    export let showPaths = false;
    export let onItemClick: (item: FileItem) => void;
</script>

{#if isLoading}
    <div class="flex justify-center py-12">
        <div class="h-6 w-6 animate-spin rounded-full border-2 border-gray-300 border-t-blue-600"></div>
    </div>
{:else if error}
    <div class="rounded-md bg-red-50 p-4 text-sm text-red-700">
        {error}
    </div>
{:else if contents.length === 0}
    <div class="py-12 text-center text-sm text-gray-500">
        No files found
    </div>
{:else}
    <div class="space-y-1">
        {#each contents as item}
            <button
                on:click={() => onItemClick(item)}
                class="flex w-full items-center rounded-md px-2 py-1.5 text-sm hover:bg-gray-100 {selectedFile?.name === item.name ? 'bg-gray-50' : ''}"
            >
                <FileIcon fileName={item.name} type={item.type} />
                <div class="ml-2 overflow-hidden text-left">
                    <div class="truncate font-medium text-gray-900">
                        {item.name}
                    </div>
                    {#if showPaths && item.path}
                        <div class="truncate text-xs text-gray-500">
                            {item.path}
                        </div>
                    {/if}
                </div>
            </button>
        {/each}
    </div>
{/if}
