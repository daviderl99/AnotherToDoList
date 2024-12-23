<script>
  export let task;
  export let onUpdate;
  export let onDelete;

  let isEditing = false;
  let editedName = task.name;

  function toggleComplete() {
    onUpdate({
      ...task,
      completed: !task.completed
    });
  }

  function startEditing() {
    isEditing = true;
    editedName = task.name;
  }

  function saveEdit() {
    if (editedName.trim() !== '') {
      onUpdate({
        ...task,
        name: editedName
      });
    }
    isEditing = false;
  }

  function handleKeydown(event) {
    if (event.key === 'Enter') {
      saveEdit();
    }
  }

  function formatDate(dateString) {
    const date = new Date(dateString);
    return date.toLocaleString(undefined, {
      hour: '2-digit',
      minute: '2-digit',
      year: 'numeric',
      month: '2-digit',
      day: '2-digit'
    });
  }

  const focusOnMount = (node) => {
    node.focus();
  };
</script>

<div class="task">
  <input 
    type="checkbox" 
    checked={task.completed} 
    on:click={toggleComplete}
  />
  {#if isEditing}
    <input
      type="text"
      bind:value={editedName}
      on:blur={saveEdit}
      on:keydown={handleKeydown}
      class="edit-input"
      use:focusOnMount
    />
  {:else}
    <span class:completed={task.completed}>{task.name}</span>
  {/if}
  <div class="actions">
    <span class="date-text mobile-hide">{formatDate(task.createdAt)}</span>
    <button 
      on:click|preventDefault|stopPropagation={startEditing}
      on:touchend|preventDefault|stopPropagation={startEditing}
      class="edit-btn" 
      aria-label="Edit task">
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="24" height="24" fill="currentColor">
        <path d="M20.71 5.63l-2.34-2.34c-.39-.39-1.02-.39-1.41 0l-1.83 1.83 3.75 3.75 1.83-1.83c.39-.39.39-1.02 0-1.41zM3 17.25V21h3.75L17.81 9.94l-3.75-3.75L3 17.25z"/>
      </svg>
    </button>
    <button on:click={onDelete} class="delete-btn" aria-label="Delete task">
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="24" height="24" fill="currentColor">
        <path d="M3 6l3 16.5c.1.7.7 1.5 1.5 1.5h9c.8 0 1.4-.8 1.5-1.5L21 6H3zm18-3h-5l-1-1h-4l-1 1H3v2h18V3z"/>
      </svg>
    </button>
  </div>
</div>

<style>
  .task {
    display: flex;
    justify-content: space-between;
    background-color: var(--task-background);
    align-items: center;
    padding: 15px;
    margin: 15px 5px;
    background-color: var(--task-background);
    border-radius: 8px;
    box-shadow: 0 3px 8px rgba(0, 0, 0, 0.05);
    transition: transform 0.2s ease, background-color 0.3s ease;
    font-family: 'Helvetica Neue', sans-serif;
    color: var(--text-color);
  }
  .task:hover {
    transform: scale(1.02);
    background-color: var(--task-hover);
  }
  input[type="checkbox"] {
    margin-right: 10px;
    width: 20px;
    height: 20px;
    cursor: pointer;
  }
  .completed {
    text-decoration: line-through;
    color: #95a5a6;
  }
  .actions {
    display: flex;
    gap: 10px;
    align-items: center;
  }
  button {
    background: none;
    border: none;
    cursor: pointer;
    padding: 0;
    transition: color 0.3s ease;
  }
  .edit-btn {
    color: #3498db;
  }
  .edit-btn:hover {
    color: #2980b9;
  }
  .delete-btn {
    color: #e74c3c;
  }
  .delete-btn:hover {
    color: #c0392b;
  }
  svg {
    width: 20px;
    height: 20px;
  }
  .edit-input {
    flex: 1;
    margin: 0 10px;
    padding: 5px;
    border: 2px solid #3498db;
    border-radius: 4px;
    font-size: 1em;
    background-color: var(--task-background);
    color: var(--text-color);
  }
  span {
    flex: 1;
    margin: 0 10px;
  }
  .date-text {
    color: #666;
    font-size: 0.85rem;
    margin: 0 10px;
    white-space: nowrap;
  }

  @media (max-width: 600px) {
    .mobile-hide {
      display: none;
    }

    .task {
      padding: 12px;
      margin: 8px 2px;
    }

    .actions {
      gap: 8px;
    }
  }
</style>
