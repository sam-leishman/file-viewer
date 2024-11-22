<!-- FilePreview.svelte -->
<script lang="ts">
    interface FileItem {
        name: string;
        type: 'file' | 'directory';
        handle: FileSystemHandle;
    }

    export let selectedFile: FileItem | null = null;
    export let filePreviewContent: string | ArrayBuffer | null = null;
    export let isLoadingPreview = false;
    export let previewError: string | null = null;
    export let fileMetadata: { size?: number; lastModified?: number } = {};

    function formatFileSize(bytes: number): string {
        const units = ['B', 'KB', 'MB', 'GB'];
        let size = bytes;
        let unitIndex = 0;
        
        while (size >= 1024 && unitIndex < units.length - 1) {
            size /= 1024;
            unitIndex++;
        }
        
        return `${size.toFixed(1)} ${units[unitIndex]}`;
    }

    function formatDate(timestamp: number): string {
        return new Date(timestamp).toLocaleString();
    }
</script>

{#if selectedFile}
    <div class="mb-4">
        <h2 class="text-lg font-medium text-gray-900">{selectedFile.name}</h2>
        {#if fileMetadata.size !== undefined || fileMetadata.lastModified !== undefined}
            <div class="mt-2 text-sm text-gray-500">
                {#if fileMetadata.size !== undefined}
                    <p>Size: {formatFileSize(fileMetadata.size)}</p>
                {/if}
                {#if fileMetadata.lastModified !== undefined}
                    <p>Last modified: {formatDate(fileMetadata.lastModified)}</p>
                {/if}
            </div>
        {/if}
    </div>

    {#if isLoadingPreview}
        <div class="flex justify-center py-12">
            <div class="h-6 w-6 animate-spin rounded-full border-2 border-gray-300 border-t-blue-600"></div>
        </div>
    {:else if previewError}
        <div class="rounded-md bg-red-50 p-4 text-sm text-red-700">
            {previewError}
        </div>
    {:else if filePreviewContent}
        {#if selectedFile.name.toLowerCase().match(/\.(jpg|jpeg|png|gif|webp|svg)$/)}
            <img
                src={filePreviewContent as string}
                alt={selectedFile.name}
                class="max-h-[600px] w-full rounded-md object-contain"
            />
        {:else if selectedFile.name.toLowerCase().match(/\.(mp4|webm|mov|avi|mkv)$/)}
            <video
                src={filePreviewContent as string}
                controls
                class="max-h-[600px] w-full rounded-md"
            >
                <track kind="captions">
                Your browser does not support the video tag.
            </video>
        {:else}
            <pre class="max-h-[600px] overflow-auto rounded-md bg-gray-50 p-4 text-sm">
                <code>{filePreviewContent as string}</code>
            </pre>
        {/if}
    {:else}
        <div class="rounded-md bg-gray-50 p-4 text-center text-sm text-gray-500">
            Preview not available for this file type
        </div>
    {/if}
{:else}
    <div class="flex h-full items-center justify-center text-sm text-gray-500">
        Select a file to preview
    </div>
{/if}
