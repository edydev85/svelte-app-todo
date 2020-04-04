<script>
  import { onMount } from "svelte";
  import TaskGridItem from "./TaskGridItem.svelte";
  let tasks = [];
  let newDescription = "";
  let id = null;
  let validDescription = false;
  $: validDescription = newDescription.trim().length >= 5;
  function newTask() {
    id = null;
    newDescription = "";
    document.getElementById("newDescriptionId").focus();
  }
  function submitTask() {
    id ? updateTask() : insertTask();
  }
  function insertTask() {
    const taskData = { description: newDescription };
    fetch("https://svelte-app-db.firebaseio.com/tasks.json", {
      method: "POST",
      body: JSON.stringify(taskData),
      headers: { "Content-Type": "application/json" }
    })
      .then(respose => {
        if (!respose.ok) {
          throw new error("An error occurred, please try again");
        }
        return respose.json;
      })
      .then(data => {
        tasks = [...tasks, { id: data.name, description: newDescription }];
        newTask();
      })
      .catch(error => console.log(error));
  }
  onMount(() => {
    getTasks();
  });
  function getTasks() {
    fetch("https://svelte-app-db.firebaseio.com/tasks.json")
      .then(Response => {
        if (!Response.ok) {
          throw new Error("An error occurred, please try again");
        }
        return Response.json();
      })
      .then(data => {
        for (const key in data) {
          tasks = [...tasks, { id: key, description: data[key].description }];
          console.table(tasks);
        }
      })
      .catch(console.error);
  }
  function updateTask() {
    const taskData = { description: newDescription };
    fetch(`https://svelte-app-db.firebaseio.com/${id}.json`, {
      method: "PATCH",
      body: JSON.stringify(taskData),
      headers: { "Content-Type": "application/json" }
    })
      .then(response => {
        if (!response.ok) {
          throw new error("An Error occurred");
        }
        const index = tasks.findIndex(task => task.id === id);
        tasks[index] = { id: id, description: newDescription };
        newTask();
      })
      .catch(error => console.log(error));
  }
  function editTask(event) {
    id = event.detail.id;
    newDescription = event.detail.description;
  }
  function deleteTask(event) {
    id = event.detail.id;
    fetch(`https://svelte-app-db.firebaseio.com/${id}.json`, {
      method: "DELETE"
    })
      .then(response => {
        if (!response.ok) {
          throw new error("An error occurred");
        }
        tasks = tasks.filter(task => task.id !== event.detail.id);
        newTask();
      })
      .catch(error => console.log(error));
  }
</script>

<style>
  h4 {
    margin-top: 1em;
  }
  .adding-record {
    color: darkblue;
  }
</style>

<div class="container">
  <div class="notification">
    <div class="field is-horizontal">
      <div class="field-label is-normal">
        <label for="" class="label">Task</label>
      </div>
      <div class="field-body">
        <div class="field">
          <p class="control is-expanded has-icons-left">
            <input
              type="text"
              class="input"
              id="newDescriptionId"
              placeholder="Add new task"
              bind:value={newDescription} />
            <span class="icon is-small is-left">
              <i class="fas fa-tasks" />
            </span>
          </p>
        </div>
        <!-- Buttons -->
        <div class="field is-grouped">
          <div class="control">
            <button class="button is-normal" on:click={newTask}>New</button>
          </div>
          <div class="control">
            <button
              class="button is-info"
              on:click={submitTask}
              disabled={!validDescription}>
              <span class="icon is-small">
                <i class="fas fa-check" />
              </span>
              <span>Save</span>
            </button>
          </div>
        </div>
      </div>
    </div>
    <!-- Binding -->
    <div class="field is-horizontal adding-record">
      <div class="field-label">New record:</div>
      <div class="field-body">
        <div class="field">
          <div class="control">{newDescription}</div>
        </div>
      </div>
    </div>
  </div>
</div>
<!-- grid -->
<div class="container">
  <h4 class="title is-4">Tasks</h4>
  <table class="table is-bordered is-striped is-hoverable">
    <thead>
      <tr>
        <th>Task Description</th>
        <th>Edit</th>
        <th>Delete</th>
      </tr>
    </thead>
    <tbody>
      {#each tasks as task}
        <TaskGridItem
          id={task.id}
          description={task.description}
          on:edit={editTask}
          on:delete={deleteTask} />
      {:else}
        <tr>
          <td>
            No tasks
            <i class="fas fa-sad-tear" />
          </td>
        </tr>
      {/each}
    </tbody>
  </table>
</div>
