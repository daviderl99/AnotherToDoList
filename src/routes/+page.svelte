<script>
  import Section from '$lib/Section.svelte';
  import { onMount } from 'svelte';

  let tasks = [];
  let darkMode = false;
  let initialized = false;

  onMount(() => {
    // Load tasks from localStorage
    const savedTasks = localStorage.getItem('tasks');
    if (savedTasks) {
      tasks = JSON.parse(savedTasks);
    }

    // Load theme preference from localStorage
    const savedTheme = localStorage.getItem('darkMode');
    if (savedTheme !== null) {
      darkMode = JSON.parse(savedTheme);
      document.documentElement.setAttribute('data-theme', darkMode ? 'dark' : 'light');
    } else if (typeof window !== 'undefined' && window.matchMedia) {
      // Fall back to system preference if no saved theme
      darkMode = window.matchMedia('(prefers-color-scheme: dark)').matches;
      document.documentElement.setAttribute('data-theme', darkMode ? 'dark' : 'light');
    }
    
    initialized = true;
  });

  // Save tasks whenever they change
  $: {
    if (typeof window !== 'undefined' && tasks && initialized) {
      localStorage.setItem('tasks', JSON.stringify(tasks));
    }
  }

  function toggleTheme() {
    darkMode = !darkMode;
    document.documentElement.setAttribute('data-theme', darkMode ? 'dark' : 'light');
    localStorage.setItem('darkMode', JSON.stringify(darkMode));
  }
</script>

<main>
  <button on:click={toggleTheme} class="theme-toggle" title={darkMode ? 'Switch to light theme' : 'Switch to dark theme'}>
    {#if darkMode}
      ☀️
    {:else}
      🌙
    {/if}
  </button>
  <div class="sections-container">
    <Section title="ToDo Tasks" bind:tasks={tasks} />
  </div>
</main>

<style>
  :global(:root) {
    --background-color: #f5f6fa;
    --text-color: #2c3e50;
    --container-background: #ffffff;
    --task-background: #ffffff;
    --task-hover: #f8f9fa;
    --button-background: #3498db;
    --button-hover: #2980b9;
  }

  :global([data-theme='dark']) {
    --background-color: #121212;
    --text-color: #ecf0f1;
    --container-background: #1d1d1d;
    --task-background: #2d2d2d;
    --task-hover: #383838;
    --button-background: #2980b9;
    --button-hover: #1c6ea4;
  }

  main {
    padding: 20px;
    min-height: 100vh;
    background-color: var(--background-color);
    transition: background-color 0.3s ease;
  }

  @media (max-width: 600px) {
    main {
      padding: 0;
    }

    .theme-toggle {
      top: 15px;
      right: 15px;
      width: 35px;
      height: 35px;
      font-size: 1rem;
    }
  }

  .theme-toggle {
    position: fixed;
    top: 20px;
    right: 20px;
    width: 40px;
    height: 40px;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: var(--task-background);
    color: var(--text-color);
    border: 1px solid var(--text-color);
    border-radius: 50%;
    cursor: pointer;
    font-size: 1.2rem;
    transition: all 0.2s ease;
    padding: 0;
    z-index: 1000;
  }

  .theme-toggle:hover {
    transform: scale(1.1);
    background-color: var(--task-hover);
  }
</style>
