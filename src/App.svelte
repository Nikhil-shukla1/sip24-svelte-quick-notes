<script>
  import Dexie from 'dexie';
  import { onMount } from 'svelte';

  let db;
  let pages = [];
  let currentPageIndex = 0;
  let title = '';
  let note = '';

  onMount(() => {
    setupDatabase();
    loadPages();
  });

  function setupDatabase() {
    db = new Dexie("notesApp");
    db.version(1).stores({
      pages: '++id,title',
      notes: 'title'
    });
  }

  async function loadPages() {
    pages = await db.pages.toArray();
    if (pages.length > 0) {
      selectPage(0);
    } else {
      addPage();
    }
  }

  async function saveNote() {
    if (pages[currentPageIndex].title !== title) {
      await db.notes.delete(pages[currentPageIndex].title);
      pages[currentPageIndex].title = title;
      await db.pages.put(pages[currentPageIndex]);
    }

    await db.notes.put({ title, note });
    await db.pages.put(pages[currentPageIndex]);
  }

  async function addPage() {
    const newPage = { title: "New Page" };
    const id = await db.pages.add(newPage);
    pages = await db.pages.toArray();
    selectPage(pages.length - 1);
  }

  async function selectPage(index) {
    currentPageIndex = index;
    title = pages[currentPageIndex].title;
    const noteData = await db.notes.get(title);
    note = noteData ? noteData.note : '';
  }

  async function deletePage() {
    const titleToDelete = pages[currentPageIndex].title;
    await db.notes.delete(titleToDelete);
    await db.pages.delete(pages[currentPageIndex].id);
    pages = await db.pages.toArray();
    if (pages.length > 0) {
      selectPage(0);
    } else {
      addPage();
    }
  }
</script>

<aside class="fixed top-0 left-0 z-40 w-60 h-screen">
  <div class="bg-light-gray overflow-y-auto py-5 px-3 h-full border-r border-gray-200">
    <ul class="space-y-2">
      {#each pages as page, index}
        <li>
          <button on:click={() => selectPage(index)} class="{index == currentPageIndex ? 'bg-dark-gray' : ''} py-2 px-3 text-gray-900 rounded-lg">{page.title}</button>
        </li>
      {/each}
      <li class="text-center"><button on:click={addPage} class="font-medium">+ Add page</button></li>
    </ul>
  </div>
</aside>

<main class="p-4 ml-60 h-auto">
  <div class="grid grid-cols-2 items-center mb-3">
    <h1 class="text-3xl font-bold" contenteditable bind:textContent={title}></h1>
    <div class="ml-auto flex gap-2">
      <button class="bg-gray-800 text-white px-5 py-2.5 rounded-lg font-medium text-sm mt-3 hover:bg-gray-900" on:click={saveNote}>Save</button>
      <button class="bg-red-600 text-white px-5 py-2.5 rounded-lg font-medium text-sm mt-3 hover:bg-red-700" on:click={deletePage}>Delete</button>
    </div>
  </div>
  <hr />
  <textarea class="mt-3 block w-full bg-gray-50 border border-gray-300 rounded-lg text-gray-900 p-2.5" bind:value={note}></textarea>
</main>

<style>
.bg-light-gray {
  background: #FBFBFB;
}

.bg-dark-gray {
  background: #EFEFEF;
}
</style>
