<script lang="ts">
  import { onMount } from 'svelte';
  import { auth } from '$lib/stores/auth';
  import { notesStore, filteredNotes, activeNote } from '$lib/stores/notes';
  import { api, type Note } from '$lib/services/api';
  import NoteList from '$lib/components/NoteList.svelte';
  import NoteEditor from '$lib/components/NoteEditor.svelte';

  onMount(() => {
    // redirect to login if not authenticated
    const s = $auth;
    if (!s.token) {
      const redirect = encodeURIComponent(location.pathname + location.search);
      window.location.href = `/login?redirect=${redirect}`;
      return;
    }
  });

  async function createNote() {
    const s = $auth;
    if (!s.token) return;
    const payload: Partial<Note> = {
      title: 'New note',
      content: '',
      folderId: $notesStore.selectedFolderId || null,
      tags: []
    };
    const note = await api.createNote(s.token, payload);
    notesStore.upsertNote(note);
  }

  async function saveNote() {
    const s = $auth;
    const note = $activeNote;
    if (!s.token || !note) return;
    const updated = await api.updateNote(s.token, note.id, {
      title: note.title,
      content: note.content,
      tags: note.tags
    });
    notesStore.upsertNote(updated);
  }

  async function deleteNote(id: string) {
    const s = $auth;
    if (!s.token) return;
    await api.deleteNote(s.token, id);
    notesStore.removeNote(id);
  }

  function onChangeNote(changes: Partial<Note>) {
    const current = $activeNote;
    if (!current) return;
    notesStore.upsertNote({ ...current, ...changes } as Note);
  }

  // Handle events bubbled from layout for creating folder/note
  function onCreateNoteEvent() {
    createNote();
  }

  async function onCreateFolder() {
    const name = prompt('Folder name');
    if (!name) return;
    const s = $auth;
    if (!s.token) return;
    const folder = await api.createFolder(s.token, name);
    notesStore.setFolders([folder, ...$notesStore.folders]);
  }

  async function onDeleteFolderEvent(e: CustomEvent<string>) {
    const id = e.detail;
    if (!confirm('Delete this folder?')) return;
    const s = $auth;
    if (!s.token) return;
    await api.deleteFolder(s.token, id);
    notesStore.setFolders($notesStore.folders.filter((f) => f.id !== id));
    notesStore.setFilter({ selectedFolderId: null });
  }
</script>

<svelte:head>
  <title>{import.meta.env?.VITE_APP_TITLE || 'Notes'}</title>
</svelte:head>

<div class="grid" on:create-note={() => onCreateNoteEvent()} on:create-folder={() => onCreateFolder()} on:delete-folder={(e) => onDeleteFolderEvent(e as CustomEvent<string>)}>
  <section class="left">
    <NoteList
      notes={$filteredNotes}
      activeId={$notesStore.activeNoteId}
      onSelect={(id) => notesStore.setActiveNote(id)}
      onDelete={(id) => deleteNote(id)}
    />
  </section>
  <section class="right">
    <NoteEditor note={$activeNote} onChange={onChangeNote} onSave={saveNote} />
  </section>
</div>

<style>
  .grid {
    display: grid;
    grid-template-columns: 360px 1fr;
    gap: 16px;
    height: calc(100vh - var(--header-height) - 32px);
  }
  .left, .right {
    min-height: 0;
  }
  .left {
    overflow: auto;
    padding-right: 4px;
  }
  .right {
    overflow: auto;
  }

  @media (max-width: 900px) {
    .grid {
      grid-template-columns: 1fr;
      height: auto;
    }
  }
</style>
