<script>
  import { dndzone } from 'svelte-dnd-action';
  import Task from './Task.svelte';
  
  export let tasks;
  export let title;
  
  let newTaskName = '';
  let sortBy = 'custom';
  let sortDirection = 'asc';

  function handleDrop(event) {
    const { items } = event.detail;
    // Update positions when items are reordered
    tasks = items.map((item, index) => ({
      ...item,
      position: index
    }));
  }

  function addTask() {
    if (newTaskName.trim()) {
      tasks = [...tasks, {
        id: Date.now(),
        name: newTaskName.trim(),
        completed: false,
        createdAt: Date.now(),
        position: tasks.length
      }];
      newTaskName = '';
    }
  }

  function handleKeydown(event) {
    if (event.key === 'Enter') {
      addTask();
    }
  }

  function updateTask(updatedTask) {
    tasks = tasks.map(task => 
      task.id === updatedTask.id ? updatedTask : task
    );
  }

  function deleteTask(taskId) {
    tasks = tasks.filter(task => task.id !== taskId);
  }

  function clearCompletedTasks() {
    if (confirm(`Are you sure you want to remove ${tasks.filter(task => task.completed).length} completed ${tasks.filter(task => task.completed).length === 1 ? 'task' : 'tasks'}?`)) {
      tasks = tasks.filter(task => !task.completed);
    }
  }

  $: sortedTasks = [...tasks].sort((a, b) => {
    const direction = sortDirection === 'asc' ? 1 : -1;
    
    switch (sortBy) {
      case 'custom':
        return (a.position - b.position) * direction;
      case 'createdAt':
        return (a.createdAt - b.createdAt) * direction;
      case 'completed':
        return (a.completed === b.completed ? 0 : a.completed ? 1 : -1) * direction;
      case 'name':
        return a.name.localeCompare(b.name) * direction;
      default:
        return 0;
    }
  });

  $: completedTasks = tasks.filter(task => task.completed).length;
  $: totalTasks = tasks.length;
  $: progress = totalTasks ? (completedTasks / totalTasks) * 100 : 0;
</script>

<div class="section">
  <h2>{title}</h2>
  <div class="header-controls">
    <div class="add-task">
      <input 
        type="text" 
        bind:value={newTaskName} 
        placeholder="Add a new task..." 
        on:keydown={handleKeydown}
      />
      <button on:click={addTask} class="square-button" title="Add task" aria-label="Add task"></button>
    </div>
    <div class="sort-controls">
      <select bind:value={sortBy}>
        <option value="custom">Custom Order</option>
        <option value="createdAt">Created Date</option>
        <option value="name">Name</option>
        <option value="completed">Status</option>
      </select>
      <button class="sort-direction" on:click={() => sortDirection = sortDirection === 'asc' ? 'desc' : 'asc'}>
        {sortDirection === 'asc' ? '↑' : '↓'}
      </button>
    </div>
  </div>

  {#if sortedTasks.length === 0}
    <div class="task-list">
      <div class="no-tasks">
        <span>No tasks yet! Add one above.</span>
      </div>
    </div>
  {:else}
    <div class="tasks-container"
      use:dndzone={{items: sortedTasks}}
      on:consider={handleDrop} 
      on:finalize={handleDrop}>
      {#each sortedTasks as task (task.id)}
        <Task 
          {task}
          onUpdate={updateTask}
          onDelete={() => deleteTask(task.id)}
        />
      {/each}
    </div>
  {/if}

  <div class="progress-bar">
    <div class="progress" style="width: {progress}%"></div>
    <div class="progress-text">
      {completedTasks} of {totalTasks} tasks completed ({Math.round(progress)}%)
    </div>
  </div>

  {#if completedTasks > 0}
    <button class="clear-completed" on:click={clearCompletedTasks}>
      Clear {completedTasks} Completed {completedTasks === 1 ? 'Task' : 'Tasks'}
    </button>
  {/if}
</div>

<style>
  .section {
    margin: 20px auto;
    padding: 20px;
    background-color: var(--container-background);
    border-radius: 10px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    max-width: 800px;
  }

  .tasks-container {
    padding: 5px;
    margin: 20px 0;
    max-height: 400px;
    overflow-y: auto;
    padding-right: 8px;
  }

  @media (max-width: 600px) {
    .section {
      margin: 0;
      padding: 15px;
      border-radius: 0;
      min-height: 100vh;
      display: flex;
      flex-direction: column;
    }

    .tasks-container {
      flex: 1;
      margin: 10px 0;
      max-height: none;
      height: calc(100vh - 180px);
    }
  }

  h2 {
    padding: 10px 0;
    font-size: 1.5rem;
    color: var(--text-color);
    margin-bottom: 20px;
  }

  .header-controls {
    display: flex;
    gap: 20px;
    margin-bottom: 20px;
  }

  @media (max-width: 600px) {
    .header-controls {
      flex-direction: column;
      gap: 15px;
    }
  }

  .add-task {
    display: flex;
    gap: 10px;
    flex: 1;
  }

  .add-task input {
    flex: 1;
    height: 40px;
    padding: 8px 12px;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-size: 1rem;
    background-color: var(--task-background);
    color: var(--text-color);
  }

  .sort-controls {
    display: flex;
    gap: 10px;
    align-items: center;
  }

  @media (max-width: 600px) {
    .sort-controls {
      width: 100%;
    }

    .sort-controls select {
      flex: 1;
    }
  }

  select {
    height: 40px;
    padding: 8px 28px 8px 12px;
    border: 1px solid #ddd;
    border-radius: 4px;
    background-color: var(--task-background);
    color: var(--text-color);
    font-size: 1rem;
    appearance: none;
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='8' height='8' viewBox='0 0 8 8'%3E%3Cpath fill='%23666' d='M0 2l4 4 4-4z'/%3E%3C/svg%3E");
    background-repeat: no-repeat;
    background-position: right 12px center;
  }

  button {
    height: 40px;
    padding: 8px 16px;
    background-color: var(--button-background);
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 1rem;
    transition: background-color 0.2s;
  }

  button:hover {
    background-color: var(--button-hover);
  }

  .square-button {
    width: 40px;
    padding: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 1.5rem;
    font-weight: bold;
    line-height: 0;
  }

  .square-button::before {
    content: '+';
    position: relative;
    top: -2px;
  }

  .square-button:hover {
    background-color: var(--button-hover);
  }

  .sort-direction {
    width: 40px;
    padding: 0;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .task-list {
    min-height: 150px;
    border: 2px solid #ecf0f1;
    border-radius: 10px;
    padding: 10px;
    margin: 20px 0;
    background-color: var(--container-background);
  }

  @media (max-width: 600px) {
    .task-list {
      padding: 8px;
      margin: 15px 0;
    }
  }

  .no-tasks {
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 2rem;
    color: var(--text-color);
    font-size: 1.2rem;
    opacity: 0.7;
  }

  .progress-bar {
    width: 100%;
    height: 20px;
    background-color: #ecf0f1;
    border-radius: 5px;
    overflow: hidden;
    margin-top: 20px;
    position: relative;
  }

  .progress {
    height: 100%;
    background-color: #2ecc71;
    transition: width 0.3s ease;
  }

  .progress-text {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    color: #2c3e50;
    font-size: 0.9rem;
    font-weight: 500;
    white-space: nowrap;
    text-shadow: 0 0 2px rgba(255, 255, 255, 0.8);
  }

  .clear-completed {
    margin-top: 20px;
    background-color: #e74c3c;
  }

  .clear-completed:hover {
    background-color: #c0392b;
  }

  .tasks-container::-webkit-scrollbar {
    width: 8px;
  }

  .tasks-container::-webkit-scrollbar-track {
    background: transparent;
  }

  .tasks-container::-webkit-scrollbar-thumb {
    background-color: var(--button-background);
    border-radius: 4px;
  }

  .tasks-container::-webkit-scrollbar-thumb:hover {
    background-color: var(--button-hover);
  }
</style>
