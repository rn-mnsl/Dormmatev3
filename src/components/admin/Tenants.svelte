<script>
  import { onMount } from 'svelte';
  import { getUsers, createUser, getUnits, updateUser } from '../../lib/api';

  let tenants = [];
  let units = [];
  let loading = true;
  let error = null;
  let newTenantName = '';
  let newTenantEmail = '';
  let newTenantPassword = '';
  let newTenantContactNumber = "";
  let newTenantMoveInDate = "";
  let selectedUnitId = null;
  let showModal = false;
  let showEditModal = false;
  let searchQuery = "";
  let editingTenant = null;

  $: filteredTenants = tenants.filter(tenant =>
    String(tenant.userId).includes(searchQuery) ||
    tenant.name.toLowerCase().includes(searchQuery.toLowerCase()) ||
    tenant.username.toLowerCase().includes(searchQuery.toLowerCase()) ||
    tenant.email.toLowerCase().includes(searchQuery.toLowerCase()) ||
    (tenant.unit ? tenant.unit.unitName.toLowerCase().includes(searchQuery.toLowerCase()) : false)
  );

  onMount(async () => {
    fetchTenants();
    fetchUnits();
  });

  async function fetchTenants() {
    loading = true;
    try {
      tenants = await getUsers();
    } catch (err) {
      error = err.message;
    } finally {
      loading = false;
    }
  }

  async function fetchUnits(){
    try {
      units = await getUnits();
    } catch (err) {
      error = err.message;
    }
  }

  async function handleAddTenant(){
    try {
      const response = await createUser({
        username: newTenantName,
        email: newTenantEmail,
        password: newTenantPassword,
        role: "tenant",
        name: newTenantName,
        contactNumber: newTenantContactNumber,
        moveInDate: newTenantMoveInDate
      }, selectedUnitId);

      await fetchTenants();
      newTenantName = "";
      newTenantEmail = "";
      newTenantPassword = "";
      newTenantContactNumber = "";
      newTenantMoveInDate = "";
      selectedUnitId = null;
      showModal = false;
    } catch (err) {
      error = err.message;
    }
  }

  function handleEditClick(tenant) {
    editingTenant = { ...tenant };
    selectedUnitId = tenant.unit ? tenant.unit.unitId : null;
    showEditModal = true;
  }

  async function handleEditTenant() {
    try {
      await updateUser({
        userId: editingTenant.userId,
        username: editingTenant.username,
        email: editingTenant.email,
        name: editingTenant.name,
        contactNumber: editingTenant.contactNumber,
        moveInDate: editingTenant.moveInDate
      }, selectedUnitId);

      await fetchTenants();
      showEditModal = false;
      editingTenant = null;
      selectedUnitId = null;
    } catch (err) {
      error = err.message;
    }
  }
</script>
  
  <div class="main-container">
  <div class="main-header">Tenant Management</div>
  <p class="subtext"> View a list of tenants and manage their information, including contact details, assigned units, and move-in dates. </p>

  <!-- Search Input Field -->
  <div class="search-add-container">
    <div class="search-container">
      <input
        type="text"
        placeholder="Search tenants"
        bind:value={searchQuery}
      />
    </div>  
    <button on:click={() => showModal = true}> Add Tenant </button>
  </div>  
  
  {#if loading}
    <p>Loading tenants...</p>
  {:else if error}
    <p>Error: {error}</p>
  {:else}
    <table>
      <thead>
        <tr>
          <th>ID</th>
          <th>Name</th>
          <th>Username</th>
          <th>Email</th>
            <th>Unit</th>
          <th>Contact #</th>
          <th>Move-In Date</th>
        </tr>
      </thead>
      <tbody>
        {#each filteredTenants as tenant}
          <tr>
            <td>{tenant.userId}</td>
            <td>{tenant.name}</td>
            <td>{tenant.username}</td>
              <td>{tenant.email}</td>
              <td>{tenant.unit ? tenant.unit.unitName : 'Not Assigned'}</td>
              <td>{tenant.contactNumber}</td>
            <td>
               {#if tenant.moveInDate}
                 {new Date(tenant.moveInDate).toLocaleDateString()}
                {:else}
                 Not Set
               {/if}
             </td>
          </tr>
        {/each}
      </tbody>
    </table>

  <!-- handle the Add Tenant Button -->
      {#if showModal}
         <div class="modal">
            <div class="modal-content">
               <form on:submit|preventDefault={handleAddTenant}>
                   <h2>Add New Tenant</h2>
                   <div>
                     <label for="tenantName">Tenant Name:</label>
                    <input type="text" id="tenantName" bind:value={newTenantName} required />
                 </div>
                <div>
                     <label for="tenantEmail">Tenant Email:</label>
                    <input type="email" id="tenantEmail" bind:value={newTenantEmail} required />
                 </div>
                  <div>
                    <label for="tenantPassword">Tenant Password:</label>
                       <input type="password" id="tenantPassword" bind:value={newTenantPassword} required />
                  </div>
                  <div>
                     <label for="tenantContactNumber">Contact Number: </label>
                     <input type="text" id="tenantContactNumber" bind:value={newTenantContactNumber} required />
                   </div>
                   <div>
                      <label for="tenantMoveInDate">Move In Date: </label>
                      <input type="date" id="tenantMoveInDate" bind:value={newTenantMoveInDate} required />
                  </div>
                  <div>
                       <label for="units"> Select Unit </label>
                       <select id="units" bind:value={selectedUnitId}>
                             <option value={null}>Select a Unit</option>
                            {#each units as unit}
                                <option value={unit.unitId}> {unit.unitName}</option>
                          {/each}
                      </select>
                     </div>
                   <button type="submit">Add Tenant</button>
                  <button on:click={() => showModal = false}>Cancel</button>
               </form>
              </div>
          </div>
      {/if}
  {/if}
</div>
  
  <style>

  .main-container { 
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  margin-left: 300px;
  }

  .search-add-container {
  display: flex;
  justify-content: space-between; 
  align-items: center; 
  margin-bottom: 20px;
  }

  .search-container {
  flex: 1;
  margin-right: 20px;
  }

  .search-container input {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1em;
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

  table {
  width: 100%;
  border-collapse: collapse;
  }
  th,
  td {
  border: 1px solid #ddd;
  padding: 8px;
  }
  th {
  background-color: #f0f0f0;
  }
  form {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 20px;
  width: 100%;
  margin: 20px auto; /* Center the form */
  }

  form div {
  margin-bottom: 10px;
  display: flex;
  flex-direction: column; /* Arrange labels and inputs vertically */
  width: 100%;
  }
  label {
  text-align: left; /* Align labels to the left */
  margin-bottom: 5px;
  }
  input[type="text"],
  input[type="email"],
  input[type="password"],
  input[type="date"],
  select {
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  width: 100%; /* Make input fields fill their container */
  box-sizing: border-box;
  }
  button {
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin: 0 10px 10px 0;
  white-space: nowrap;
  }
  button:hover {
  background-color: #0056b3;
  }
  .modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5); /* semi-transparent black */
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  }
  .modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 20px;
  display: flex;
  flex-direction: column;
  text-align: center;
  width: 400px;
  }
  </style>