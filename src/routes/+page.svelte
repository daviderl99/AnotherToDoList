<script>
  import Section from '$lib/Section.svelte';
  import { onMount } from 'svelte';

  let tasks = [];
  let darkMode = false;
  let initialized = false;

  onMount(() => {
    // Load tasks from localStorage
    const savedTasks = localStorage.getItem('tasks');
    console.log(savedTasks);
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
  <button on:click={toggleTheme} class="theme-toggle">Toggle Theme</button>
  <div class="sections-container">
    <Section title="Tasks" bind:tasks={tasks} />
  </div>
</main>

<style>
  :global(:root) {
    --background-color: #ffffff;
    --text-color: #2c3e50;
    --container-background: #f5f6fa;
    --task-background: #ffffff;
    --task-hover: #f8f9fa;
    --button-background: #3498db;
    --button-hover: #2980b9;
  }

  :global([data-theme='dark']) {
    --background-color: #121212;
    --text-color: #ecf0f1;
    --container-background: #1e1e1e;
    --task-background: #2d2d2d;
    --task-hover: #383838;
    --button-background: #2980b9;
    --button-hover: #1c6ea4;
  }

  main {
    padding: 20px;
    background-color: var(--background-color);
    color: var(--text-color);
    min-height: 100vh;
  }

  .sections-container {
    display: flex;
    justify-content: space-around;
    gap: 20px;
    margin-top: 60px;
  }

  .theme-toggle {
    position: fixed;
    top: 20px;
    right: 20px;
    background-color: var(--button-background);
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.3s ease;
  }

  .theme-toggle:hover {
    background-color: var(--button-hover);
  }
</style>
