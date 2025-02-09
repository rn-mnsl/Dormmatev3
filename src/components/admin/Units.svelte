<script>
  import { onMount } from 'svelte';
  import { getUnits, createUnit, updateUnit, deleteUnit } from '../../lib/api';

  let units = [];
  let loading = true;
  let error = null;
  let newUnitName = '';
  let newUnitDescription = '';
  let editingUnitId = null;
  let editUnitName = '';
  let editUnitDescription = '';
    let showModal = false;
    let deletingUnitId = null;
     let showDeleteConfirmation = false;
    let searchQuery = ''; // Add searchQuery variable

  // Computed property to filter units based on searchQuery
  $: filteredUnits = units.filter(unit =>
    String(unit.unitId).includes(searchQuery) ||
    unit.unitName.toLowerCase().includes(searchQuery.toLowerCase()) ||
    unit.description.toLowerCase().includes(searchQuery.toLowerCase())
  ); 

  onMount(async () => {
    fetchUnits();
  });

  async function fetchUnits() {
    loading = true;
    try {
      units = await getUnits();
    } catch (err) {
      error = err.message;
    } finally {
      loading = false;
    }
  }

    async function handleSubmit() {
        try {
            const response = await createUnit({
                unitName: newUnitName,
                description: newUnitDescription
            });

            // Update the local list
            await fetchUnits();

            newUnitName = "";
            newUnitDescription = "";
            showModal = false;


            console.log('New unit created:', response);

        } catch (err) {
            error = err.message;
        }
    }
    
  function startEdit(unit) {
    editingUnitId = unit.unitId;
    editUnitName = unit.unitName;
    editUnitDescription = unit.description;
  }

  async function updateUnitData() {
    try {
      const response = await updateUnit(editingUnitId, {
        unitName: editUnitName,
        description: editUnitDescription,
      });

      await fetchUnits();

      editingUnitId = null;

      console.log('Unit updated', response);
    } catch (err) {
      error = err.message;
    }
  }

  function cancelEdit() {
    editingUnitId = null;
  }

  function confirmDelete(unitId) {
    deletingUnitId = unitId;
    showDeleteConfirmation = true;
  }

  function cancelDelete() {
    deletingUnitId = null;
    showDeleteConfirmation = false;
  }

  async function handleDelete() {
    try {
      await deleteUnit(deletingUnitId);
      await fetchUnits(); // Refresh the list after delete

      deletingUnitId = null;
      showDeleteConfirmation = false;

      console.log('Unit deleted');
    } catch (error) {
      error = error.message;
    }
  }
</script>


<div class="main-container">
  <div class="main-header">Units Management</div>
  <p class="subtext">Allows the admin to add and manage units. They can assign tenants to specific units and store tenant information, ensuring streamlined tracking and updates.</p>

  <!-- Add Search Input Field -->
  <div class="search-container">
    <input
      type="text"
      bind:value={searchQuery}
      placeholder="Search units..."
    />
  </div>

  {#if loading}
    <p>Loading units...</p>
  {:else if error}
    <p>Error: {error}</p>
  {:else}
    <div class="unit-cards">
      {#each filteredUnits as unit}
        <div class="unit-card">
          <div class="card-header">
            <h3>Unit ID No. {unit.unitId}</h3>
          </div>
          {#if editingUnitId === unit.unitId}
            <div class="edit-form">
              <div>
                <label for="editUnitName">Unit Name:</label>
                <input type="text" id="editUnitName" bind:value={editUnitName} required />
              </div>
              <div>
                <label for="editUnitDescription">Description:</label>
                
                <input type="text" id="editUnitDescription" bind:value={editUnitDescription} />
              </div>
              <div class="button-container">
                <button on:click={updateUnitData}>Update</button>
                <button on:click={cancelEdit}>Cancel</button>
              </div>
            </div>
          {:else}
            <p><strong>Unit Name:</strong> {unit.unitName}</p>
            <p><strong>Description:</strong> {unit.description}</p>
            <div class="button-container">
              <button on:click={() => startEdit(unit)} class="editbutton">Edit</button>
              <button on:click={() => confirmDelete(unit.unitId)} class="deletebutton">Delete</button>
            </div>
          {/if}
        </div>
      {/each}
    </div>
  {/if}

  <button on:click={() => (showModal = true)} class="addbutton">Add Unit</button>

  {#if showModal}
    <div class="modal">
      <div class="modal-content">
        <form on:submit|preventDefault={handleSubmit}>
          <h2>Add New Unit</h2>
          <div>
            <label for="unitName">Unit Name:</label>
            <input type="text" id="unitName" bind:value={newUnitName} required />
          </div>
          <div>
            <label for="description">Description:</label>
            <input type="text" id="description" bind:value={newUnitDescription} />
          </div>
          <button type="submit">Add Unit</button>
          <button type="button" on:click={() => (showModal = false)}>Cancel</button>
        </form>
      </div>
    </div>
  {/if}

  {#if showDeleteConfirmation}
    <div class="modal">
      <div class="modal-content">
        <div class="delete-modal">
        <h2>Confirm Deletion</h2>
        <p>Are you sure you want to delete this unit?</p>
        <div>
          <button on:click={handleDelete} class="editbutton">Yes</button>
          <button on:click={cancelDelete} class="deletebutton">Cancel</button>
        </div>
        </div>
      </div>
    </div>
  {/if}
</div>

<style>

  .main-container { 
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
    margin-left: 300px;
  }

  .main-header {
    font-size: 2em; 
    font-weight: bold; 
    margin-top: 5px;
    margin-bottom: 10px; 
  }

  .subtext {
    font-size: 0.9em;
    color: #666;
    margin-top: 1px;
    margin-bottom: 25px;
  }

  .search-container {
    margin-bottom: 20px;
  }

  .search-container input {
    width: 100%;
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-size: 1em;
  }

  .unit-cards {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 20px;
  }

  .unit-card {
    background-color: white;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    padding: 20px;
    display: flex;
    flex-direction: column;
    gap: 5px;
  }

  .unit-card p {
    margin-top: 5px;
    margin-bottom:8px;
  }

  .card-header h3 {
    margin: 0;
    font-size: 1.5em;
  }

  .edit-form {
    display: flex;
    flex-direction: column;
    gap: 10px;
  }

    form {
      display: flex;
      flex-direction: column;
      align-items: center;
      margin-top: 20px;
       width: 50%;
      margin: 20px auto; /* Center the form */
    }

    form div {
      margin-bottom: 10px;
        display: flex;
       flex-direction: column; /* Arrange labels and inputs vertically */
    }

    label {
        text-align: left; /* Align labels to the left */
    }

    input[type="text"]{
       padding: 8px;
         border: 1px solid #ddd;
        border-radius: 4px;
         width: 100%; /* Make input fields fill their container */
    }

  .edit-form input {
    padding: 8px;
    border: 1px solid #ddd;
    border-radius: 4px;
    width: 100%;
  }

  .button-container {
    display: flex;
    justify-content: center;
    gap: 10px;
    margin-top: auto;
    align-items: flex-end;
    flex-shrink: 0;
    width: 100%;
  }

  .addbutton {
    position: fixed;
    bottom: 20px;
    right: 20px;
    padding: 10px 20px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    z-index: 1000 
  }

  button {
    padding: 10px 20px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }

  button:hover {
    background-color: #0056b3;
  }

  .editbutton {
    background-color: green;
  }

  .editbutton:hover {
    background-color: rgb(1, 63, 1);
  }

  .deletebutton {
    background-color: red;
  }

  .delete-modal {
    justify-content: center;
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .deletebutton:hover {
    background-color: rgb(135, 0, 0);
  }

  .modal {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000;
  }

  .modal-content {
    background-color: white;
    padding: 20px;
    border-radius: 8px;
    display: flex;
    flex-direction: column;
    gap: 10px;
    width: 400px;
  }

  form {
    display: flex;
    flex-direction: column;
    gap: 10px;
  }

  form div {
    display: flex;
    flex-direction: column;
    gap: 5px;
  }

  label {
    font-weight: bold;
  }

  input[type="text"] {
    padding: 8px;
    border: 1px solid #ddd;
    border-radius: 4px;
    width: 100%;
  }

</style>
