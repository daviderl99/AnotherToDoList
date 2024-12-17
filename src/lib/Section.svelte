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
      <button on:click={addTask}>Add Task</button>
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
    <div 
      class="task-list"
      use:dndzone={{items: sortedTasks}}
      on:consider={handleDrop} 
      on:finalize={handleDrop}
    >
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
    <span class="progress-text">{completedTasks} of {totalTasks} tasks completed</span>
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
    background-color: var(--background-color);
    border-radius: 10px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    width: 60%;
    transition: transform 0.2s ease, box-shadow 0.2s ease;
  }

  h2 {
    margin-bottom: 20px;
    font-size: 1.8em;
    color: var(--text-color);
    font-weight: 700;
  }

  .header-controls {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
    gap: 20px;
  }

  .add-task {
    display: flex;
    flex: 1;
    gap: 10px;
  }

  input {
    flex: 1;
    padding: 12px;
    border: 2px solid #ecf0f1;
    border-radius: 5px;
    font-size: 1em;
    background-color: var(--task-background);
    color: var(--text-color);
  }

  button {
    background-color: var(--button-background);
    color: white;
    border: none;
    padding: 12px 20px;
    border-radius: 5px;
    cursor: pointer;
    font-size: 1em;
    transition: background-color 0.3s ease;
  }

  button:hover {
    background-color: var(--button-hover);
  }

  .sort-controls {
    display: flex;
    gap: 10px;
    align-items: center;
  }

  select {
    padding: 12px;
    border: 2px solid #ecf0f1;
    border-radius: 5px;
    background-color: var(--task-background);
    color: var(--text-color);
    font-size: 1em;
    cursor: pointer;
  }

  .sort-direction {
    padding: 12px;
    width: 45px;
  }

  .task-list {
    min-height: 150px;
    border: 2px solid #ecf0f1;
    border-radius: 10px;
    padding: 20px;
    margin: 20px 0;
    background-color: var(--container-background);
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
    height: 30px;
    background-color: #ecf0f1;
    border-radius: 5px;
    margin: 20px 0;
    overflow: hidden;
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
    font-weight: bold;
    white-space: nowrap;
    text-shadow: 0 0 2px rgba(255, 255, 255, 0.8);
  }

  .clear-completed {
    margin-top: 10px;
    background-color: #e74c3c;
    width: 100%;
  }

  .clear-completed:hover {
    background-color: #c0392b;
  }
</style>
