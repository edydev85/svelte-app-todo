<script>
  import { createEventDispatcher } from "svelte";
  const dispatch = createEventDispatcher();
  export let id;
  export let description;
</script>

<tr>
  <td>{description}</td>
  <td on:click={() => dispatch('edit', { id: id, description: description })}>
    <i class="fas fa-edit" />
  </td>
  <td on:click={() => dispatch('delete', { id: id })}>
    <i class="fas fa-trash" />
  </td>
</tr>
