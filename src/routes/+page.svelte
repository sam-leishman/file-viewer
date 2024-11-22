<script lang="ts">
    import BreadcrumbNav from '$lib/components/BreadcrumbNav.svelte';
    import FileList from '$lib/components/FileList.svelte';
    import FilePreview from '$lib/components/FilePreview.svelte';
    import SearchBar from '$lib/components/SearchBar.svelte';

    interface FileItem {
        name: string;
        type: 'file' | 'directory';
        handle: FileSystemHandle;
        path?: string; // For search results
    }

    interface BreadcrumbItem {
        name: string;
        handle: FileSystemDirectoryHandle;
    }

    let selectedDirectory: FileSystemDirectoryHandle | null = null;
    let contents: FileItem[] = [];
    let isLoading = false;
    let error: string | null = null;
    let breadcrumbs: BreadcrumbItem[] = [];
    
    // Search state
    let searchResults: FileItem[] = [];
    let isSearching = false;
    let showingSearchResults = false;

    // File preview state
    let selectedFile: FileItem | null = null;
    let filePreviewContent: string | ArrayBuffer | null = null;
    let isLoadingPreview = false;
    let previewError: string | null = null;
    let fileMetadata: { size?: number; lastModified?: number } = {};

    async function handleDirectorySelect() {
        try {
            const dirHandle = await window.showDirectoryPicker();
            selectedDirectory = dirHandle;
            breadcrumbs = [{ name: dirHandle.name, handle: dirHandle }];
            selectedFile = null;
            filePreviewContent = null;
            showingSearchResults = false;
            await loadDirectoryContents(dirHandle);
        } catch (err) {
            console.error('Error selecting directory:', err);
            error = 'Failed to open directory';
        }
    }

    async function loadDirectoryContents(dirHandle: FileSystemDirectoryHandle) {
        isLoading = true;
        error = null;
        contents = [];

        try {
            const items: FileItem[] = [];
            for await (const entry of dirHandle.values()) {
                items.push({
                    name: entry.name,
                    type: entry.kind,
                    handle: entry
                });
            }
            // Sort directories first, then files, both alphabetically
            contents = items.sort((a, b) => {
                if (a.type !== b.type) {
                    return a.type === 'directory' ? -1 : 1;
                }
                return a.name.localeCompare(b.name);
            });
        } catch (err) {
            console.error('Error reading directory contents:', err);
            error = 'Failed to read directory contents';
        } finally {
            isLoading = false;
        }
    }

    async function searchFiles(query: string) {
        if (!query?.trim() || !selectedDirectory) return;
        
        isSearching = true;
        searchResults = [];
        error = null;
        
        try {
            const normalizedQuery = String(query).trim().toLowerCase();
            searchResults = await recursiveSearch(selectedDirectory, normalizedQuery, selectedDirectory.name);
            showingSearchResults = true;
        } catch (err) {
            console.error('Error searching files:', err);
            error = 'Failed to search files';
        } finally {
            isSearching = false;
        }
    }

    async function recursiveSearch(
        dirHandle: FileSystemDirectoryHandle,
        query: string,
        currentPath: string
    ): Promise<FileItem[]> {
        if (!query) return [];

        const results: FileItem[] = [];
        
        try {
            for await (const entry of dirHandle.values()) {
                if (!entry) continue;

                const name = entry.name;
                const kind = entry.kind;

                if (!name || !kind) continue;
                
                const path = `${currentPath}/${name}`;
                const entryName = String(name).toLowerCase();
                
                if (entryName.includes(query)) {
                    results.push({
                        name,
                        type: kind,
                        handle: entry,
                        path
                    });
                }
                
                if (kind === 'directory') {
                    try {
                        const dirHandle = entry as FileSystemDirectoryHandle;
                        const subdirResults = await recursiveSearch(dirHandle, query, path);
                        results.push(...subdirResults);
                    } catch (err) {
                        console.error(`Error searching subdirectory ${path}:`, err);
                    }
                }
            }
        } catch (err) {
            console.error(`Error searching directory ${currentPath}:`, err);
        }
        
        return results;
    }

    async function handleItemClick(item: FileItem) {
        if (item.type === 'directory') {
            try {
                const dirHandle = item.handle as FileSystemDirectoryHandle;
                breadcrumbs = [...breadcrumbs, { name: item.name, handle: dirHandle }];
                selectedFile = null;
                filePreviewContent = null;
                showingSearchResults = false;
                await loadDirectoryContents(dirHandle);
            } catch (err) {
                console.error('Error opening directory:', err);
                error = 'Failed to open directory';
            }
        } else {
            await previewFile(item);
        }
    }

    function clearSearch() {
        showingSearchResults = false;
        searchResults = [];
        selectedFile = null;
        filePreviewContent = null;
    }

    async function previewFile(file: FileItem) {
        selectedFile = file;
        isLoadingPreview = true;
        previewError = null;
        filePreviewContent = null;
        fileMetadata = {};

        try {
            const fileHandle = file.handle as FileSystemFileHandle;
            const file_ = await fileHandle.getFile();
            fileMetadata = {
                size: file_.size,
                lastModified: file_.lastModified
            };

            const fileName = file.name.toLowerCase();
            if (fileName.match(/\.(jpg|jpeg|png|gif|webp|svg)$/)) {
                // Image preview
                filePreviewContent = URL.createObjectURL(file_);
            } else if (fileName.match(/\.(mp4|webm|mov|avi|mkv)$/)) {
                // Video preview
                filePreviewContent = URL.createObjectURL(file_);
            } else if (fileName.match(/\.(txt|md|js|ts|json|css|html|svelte|py|java|cpp|h|c|rs|go|rb|php|xml|yaml|yml|ini|conf|sh|bash|zsh)$/)) {
                // Text preview
                filePreviewContent = await file_.text();
            } else {
                // No preview available
                filePreviewContent = null;
            }
        } catch (err) {
            console.error('Error previewing file:', err);
            previewError = 'Failed to preview file';
        } finally {
            isLoadingPreview = false;
        }
    }

    async function navigateToBreadcrumb(index: number) {
        if (index >= 0 && index < breadcrumbs.length) {
            const targetDir = breadcrumbs[index];
            breadcrumbs = breadcrumbs.slice(0, index + 1);
            selectedFile = null;
            filePreviewContent = null;
            showingSearchResults = false;
            await loadDirectoryContents(targetDir.handle);
        }
    }

    async function handleBackClick() {
        if (breadcrumbs.length > 1) {
            const previousIndex = breadcrumbs.length - 2;
            await navigateToBreadcrumb(previousIndex);
        }
    }
</script>

<div class="min-h-screen bg-gray-50 p-8">
    <div class="mx-auto max-w-6xl">
        <h1 class="mb-8 text-4xl font-bold text-gray-900">File Viewer</h1>
        
        <div class="rounded-lg bg-white shadow-sm">
            {#if !selectedDirectory}
                <div class="p-6 text-center">
                    <p class="mb-4 text-gray-600">Select a directory to view its contents</p>
                    <button
                        on:click={() => handleDirectorySelect()}
                        class="rounded-md bg-blue-600 px-4 py-2 text-white hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2"
                    >
                        Choose Directory
                    </button>
                </div>
            {:else}
                <div class="grid grid-cols-[minmax(0,1fr),1.5fr] divide-x divide-gray-200">
                    <!-- File Browser -->
                    <div class="p-6">
                        <BreadcrumbNav
                            {breadcrumbs}
                            onNavigate={navigateToBreadcrumb}
                            onBack={handleBackClick}
                            onChangeDirectory={handleDirectorySelect}
                        />
                        
                        <SearchBar
                            onSearch={searchFiles}
                            {isSearching}
                        />

                        {#if showingSearchResults}
                            <div class="mb-4 flex items-center justify-between">
                                <p class="text-sm text-gray-600">
                                    Found {searchResults.length} results
                                </p>
                                <button
                                    on:click={clearSearch}
                                    class="text-sm text-blue-600 hover:text-blue-700"
                                >
                                    Clear Search
                                </button>
                            </div>
                            <FileList
                                contents={searchResults}
                                {selectedFile}
                                {isLoading}
                                {error}
                                onItemClick={handleItemClick}
                                showPaths={true}
                            />
                        {:else}
                            <FileList
                                {contents}
                                {selectedFile}
                                {isLoading}
                                {error}
                                onItemClick={handleItemClick}
                            />
                        {/if}
                    </div>

                    <!-- File Preview -->
                    <div class="p-6">
                        <FilePreview
                            {selectedFile}
                            {filePreviewContent}
                            {isLoadingPreview}
                            {previewError}
                            {fileMetadata}
                        />
                    </div>
                </div>
            {/if}
        </div>
    </div>
</div>
