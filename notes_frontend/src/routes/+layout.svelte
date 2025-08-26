<script lang="ts">
  import '../app.css';
  import Header from '$lib/components/Header.svelte';
  import Sidebar from '$lib/components/Sidebar.svelte';
  import { auth } from '$lib/stores/auth';
  import { notesStore } from '$lib/stores/notes';
  import { get } from 'svelte/store';
  import { api } from '$lib/services/api';

  let { children } = $props();

  let sidebarOpen = $state(false);

  // initialize auth from localStorage
  auth.init();

  async function ensureBootstrap() {
    const s = get(auth);
    if (!s.token) return;
    try {
      notesStore.setLoading(true);
      const [folders, notes] = await Promise.all([
        api.listFolders(s.token),
        api.listNotes(s.token, {})
      ]);
      notesStore.setFolders(folders);
      notesStore.setNotes(notes);
    } catch (e: unknown) {
      console.error(e);
    } finally {
      notesStore.setLoading(false);
    }
  }

  $effect(() => {
    const s = get(auth);
    if (s.token) ensureBootstrap();
  });

  function toggleSidebar() {
    sidebarOpen = !sidebarOpen;
  }
</script>

<div class="app">
  <Header
    onSearch={(q) => notesStore.setFilter({ query: q })}
    onCreateNote={() => dispatchEvent(new CustomEvent('create-note', { bubbles: true }))}
    onToggleSidebar={toggleSidebar}
  />
  <div class="content">
    <Sidebar
      open={sidebarOpen}
      folders={$notesStore.folders}
      selectedFolderId={$notesStore.selectedFolderId}
      tags={[...new Set($notesStore.notes.flatMap(n => n.tags || []))]}
      selectedTag={$notesStore.selectedTag}
      onSelectFolder={(id) => notesStore.setFilter({ selectedFolderId: id })}
      onCreateFolder={() => dispatchEvent(new CustomEvent('create-folder', { bubbles: true }))}
      onDeleteFolder={(id) => dispatchEvent(new CustomEvent('delete-folder', { detail: id, bubbles: true }))}
      onSelectTag={(tag) => notesStore.setFilter({ selectedTag: tag })}
    />
    <main>
      {@render children()}
    </main>
  </div>
</div>

<style>
  .app {
    min-height: 100vh;
    display: grid;
    grid-template-rows: var(--header-height) 1fr;
    background: var(--color-surface);
  }
  .content {
    display: grid;
    grid-template-columns: var(--sidebar-width) 1fr;
    gap: 0;
    min-height: 0;
  }
  main {
    min-height: calc(100vh - var(--header-height));
    padding: 16px;
  }

  @media (max-width: 900px) {
    .content {
      grid-template-columns: 1fr;
    }
  }
</style>
