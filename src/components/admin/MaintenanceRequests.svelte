<script>
  import { onMount } from 'svelte';
  import { getMaintenanceRequests, updateMaintenanceRequest, deleteMaintenanceRequest } from '../../lib/api';

  let requests = [];
  let loading = true;
  let error = null;
  let editingRequestId = null;
  let editRequestStatus = '';
  let deletingRequestId = null;
  let showDeleteConfirmation = false;
  let showImageModal = false;
  let selectedImage = null;

  onMount(async () => {
      fetchRequests();
  });

  async function fetchRequests() {
      loading = true;
      try {
          requests = await getMaintenanceRequests();
      } catch (err) {
          error = err.message;
      } finally {
          loading = false;
      }
  }
  
  function confirmDelete(requestId) {
      deletingRequestId = requestId;
      showDeleteConfirmation = true;
  }

  function cancelDelete() {
      deletingRequestId = null;
      showDeleteConfirmation = false;
  }

  async function handleDelete() {
      try {
          await deleteMaintenanceRequest(deletingRequestId);
          await fetchRequests();
          deletingRequestId = null;
          showDeleteConfirmation = false;
      } catch (err) {
          error = err.message;
      }
  }

  function startEdit(request) {
      editingRequestId = request.requestId;
      editRequestStatus = request.status;
  }

  async function updateRequest(requestId) {
      try {
          const response = await updateMaintenanceRequest(requestId, { status: editRequestStatus });
          await fetchRequests();
          editingRequestId = null;
          console.log('Maintenance request updated:', response);
      } catch (err) {
          error = err.message;
      }
  }

  function cancelEdit() {
      editingRequestId = null;
  }

  function openImageModal(imageUrl) {
      selectedImage = imageUrl;
      showImageModal = true;
  }

  function closeImageModal() {
      selectedImage = null;
      showImageModal = false;
  }

  // Handle image loading errors
  function handleImageError(event) {
      event.target.src = '/placeholder-image.png'; // Replace with your placeholder image
      event.target.alt = 'Image not available';
  }
</script>

<div class="main-container">
<div class="main-header">Maintenance Report</div>
<p class="subtext"> View reports submitted by tenants regarding issues or fixtures that need attention. </p>

{#if loading}
  <p>Loading requests...</p>
{:else if error}
  <p>Error: {error}</p>
{:else}
  <div class="request-cards">
      {#each requests as request}
          <div class="request-card">
              <div class="card-header">
                  <h3>Maintenance Request #{request.requestId}</h3>
                  <small class="request-date">
                      Posted on: {new Date(request.requestDate).toLocaleString()}
                  </small>
              </div>
              <p>
                  <strong>Description:</strong>
                  {request.description}
              </p>
              <p>
                  <strong>Tenant:</strong>
                  {request.tenant.name}
              </p>
              <p>
                  <strong>Unit:</strong>
                  {request.unit.unitName}
              </p>
              
              {#if request.proofOfDamage}
                  <div class="image-preview">
                      <p><strong>Proof of Damage:</strong></p>
                      <!-- svelte-ignore a11y-click-events-have-key-events -->
                      <img 
                          src={request.proofOfDamage} 
                          alt="Proof of damage"
                          on:error={handleImageError}
                          on:click={() => openImageModal(request.proofOfDamage)}
                          class="thumbnail"
                      />
                  </div>
              {/if}

              {#if editingRequestId === request.requestId}
                  <div class="status-update">
                    <div class ="status-row">
                      <label for="status">Status</label>
                      <select bind:value={editRequestStatus}>
                          <option value="OPEN">OPEN</option>
                          <option value="IN PROGRESS">IN PROGRESS</option>
                          <option value="RESOLVED">RESOLVED</option>
                          <option value="CLOSED">CLOSED</option>      
                      </select>
                    </div> 
                    <div class="button-container"> 
                      <div class="update-buttons-container">
                      <button on:click={() => updateRequest(request.requestId)} class="editbutton">Update</button>
                      <button on:click={cancelEdit} class="deletebutton">Cancel</button>
                      </div>
                    </div>
                  </div>
              {:else}
                  <p class="status">
                      <strong>Status:</strong> {request.status}
                  </p>
                  <div class="button-container">
                      <button on:click={() => startEdit(request)} class="editbutton">Edit</button>
                      <button on:click={() => confirmDelete(request.requestId)} class="deletebutton">Delete</button>
                    </div>  
              {/if}
          </div>
      {/each}
  </div>
{/if}

{#if showDeleteConfirmation}
  <div class="modal">
      <div class="modal-content">
          <h2>Confirm Deletion</h2>
          <p>Are you sure you want to delete this maintenance request?</p>
          <div>
              <button on:click={handleDelete} class="editbutton">Yes</button>
              <button on:click={cancelDelete} class="deletebutton">Cancel</button>
          </div>
      </div>
  </div>
{/if}

{#if showImageModal && selectedImage}
  <!-- svelte-ignore a11y-click-events-have-key-events -->
  <div class="modal" on:click={closeImageModal}>
      <div class="modal-image-content" on:click|stopPropagation>
          <img src={selectedImage} alt="Full size proof of damage" />
          <button class="close-button" on:click={closeImageModal}>Close</button>
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

    .request-cards {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 20px;
    justify-content: center;
    }

    .request-card {
    background-color: white;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    padding: 20px;
    width: 300px;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    }

    .image-preview {
    margin: 10px 0;
    text-align: center;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-height: 150px;
    }

    .thumbnail {
    max-width: 150px;
    max-height: 150px;
    object-fit: cover;
    border-radius: 4px;
    cursor: pointer;
    transition: transform 0.2s;
    }

    .thumbnail:hover {
    transform: scale(1.05);
    }

    .modal-image-content {
    background-color: white;
    padding: 20px;
    border-radius: 5px;
    max-width: 90vw;
    max-height: 90vh;
    position: relative;
    }

    .modal-image-content img {
    max-width: 100%;
    max-height: 80vh;
    object-fit: contain;
    }

    .close-button {
    position: absolute;
    top: 10px;
    right: 10px;
    background-color: #333;
    color: white;
    border: none;
    border-radius: 50%;
    width: 30px;
    height: 30px;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    }

    .close-button:hover {
    background-color: #555;
    }
    .card-header {
    display: flex;
    flex-direction: column;
    align-items: flex-start;;
    margin-bottom: 10px;
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

    .status-update {
    display: flex;
    flex-direction: column;
    gap: 10px;
    margin-top: 10px;
    }

    .status-update label {
    margin-right: 10px;
    }

    .status-update select {
    padding: 8px;
    border: 1px solid #ddd;
    border-radius: 4px;
    background-color: white;
    width: 100%;
    }

    .status-row {
    display: flex;
    align-items: center;
    gap: 10px;
    }

    .status-update-container {
    display: flex;
    justify-content: center;    
    gap: 10px;
    margin-top: 10px;
    align-items: center;
    width: 100%;
    }

    .status-update button {
    padding: 10px 20px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    }

    .status-update button:hover {
    background-color: #0056b3;
    }

    .request-card h3 {
    margin-top: 0;
    margin-bottom: 10px;
    }

    .request-card p {
    margin-bottom: 5px;
    position: relative;
    }

    .request-date {
    font-size: 0.8em;
    color: gray;
    display: block;
    }

    select {
    padding: 8px;
    border: 1px solid #ddd;
    border-radius: 4px;
    background-color: white;
    }

    button {
    padding: 10px 20px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    margin: 0 10px 10px 0;
    }

    button:hover {
    background-color: #0056b3;
    }

    .button-container {
    display: flex;
    justify-content: center;
    gap: 10px;
    margin-top: 10px;
    align-items: flex-end;
    flex-shrink: 0;
    width: 100%;
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

    .deletebutton:hover {
    background-color: rgb(135, 0, 0);
    }

    .modal {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.7);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000;
    }

    .modal-content {
    background-color: white;
    padding: 20px;
    border-radius: 5px;
    display: flex;
    flex-direction: column;
    text-align: center;
    width: 400px;
    }
</style>