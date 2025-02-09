<script>
  import { onMount, createEventDispatcher } from 'svelte';
  import { getMaintenanceRequests, uploadMaintenanceProof } from '../../lib/api';

  const dispatch = createEventDispatcher();

  let requests = [];
  let loading = true;
  let error = null;
  let description = '';
  let userId;
  let unitId;
  let proofOfDamage = null;
  let showModal = false;
  let showImageModal = false;
  let selectedImage = null;

  onMount(async () => {
    userId = localStorage.getItem('userId');
    unitId = localStorage.getItem('unitId');

    if (!userId || !unitId) {
      error = 'User ID or Unit ID is missing';
      return;
    }

    await fetchRequests();
  });

  async function fetchRequests() {
    loading = true;
    try {
      requests = await getMaintenanceRequests();
      requests = requests.filter(
        (request) => request.tenant && request.tenant.userId == userId
      );
    } catch (err) {
      error = err.message;
    } finally {
      loading = false;
    }
  }

  async function handleSubmit(event) {
    event.preventDefault();
    error = null;

    if (!description.trim()) {
      error = 'Please provide a description';
      return;
    }

    if (!proofOfDamage) {
      error = 'Please select a proof of damage image';
      return;
    }

    const formData = new FormData();
    formData.append('proofOfDamage', proofOfDamage);

    try {
      const userId = parseInt(localStorage.getItem('userId'), 10);
      const unitId = parseInt(localStorage.getItem('unitId'), 10);

      if (isNaN(userId) || isNaN(unitId)) {
        throw new Error('Invalid user ID or unit ID');
      }

      const now = new Date();
      const formattedDate = now.toISOString().substring(0, 19);
      const response = await uploadMaintenanceProof(formData, userId, {
        description: description,
        unitId: unitId,
        requestDate: formattedDate,
      });

      console.log('Maintenance request submitted:', response);
      await fetchRequests();
      dispatch('requestSuccess');
      closeModal();
      description = '';
      proofOfDamage = null;
    } catch (err) {
      error = err.message || 'Failed to submit maintenance request';
      console.error('Error submitting maintenance request:', err);
    }
  }

  function handleFileChange(event) {
    const file = event.target.files[0];
    proofOfDamage = file ? file : null;
  }

  function openImageModal(imageUrl) {
    selectedImage = imageUrl;
    showImageModal = true;
  }

  function closeImageModal() {
    selectedImage = null;
    showImageModal = false;
  }

  function handleImageError(event) {
    event.target.src = '/placeholder-image.png';
    event.target.alt = 'Image not available';
  }

  function openModal() {
    showModal = true;
    error = null;
  }

  function closeModal() {
    showModal = false;
    error = null;
    description = '';
    proofOfDamage = null;
    if (document.querySelector('#proofOfDamage')) {
      document.querySelector('#proofOfDamage').value = '';
    }
  }
</script>

<div class="container">
  <h2>Maintenance Request</h2>
  <hr class="divider" />

  <div class="request-history">
    <h3>Your Maintenance Requests</h3>
    {#if loading}
      <p class="loading-message">Loading Maintenance Requests...</p>
    {:else}
      <div class="request-cards">
        {#each requests as request}
          <div class="request-card">
            <div class="card-header">
              <h3 class="request-title">Request #{request.requestId}</h3>
            </div>
            <p class="request-description">
              <strong>Description:</strong> {request.description}
            </p>
            {#if request.proofOfDamage}
              <div class="image-preview">
                <p><strong>Proof of Damage:</strong></p>
                <img 
                  src={request.proofOfDamage} 
                  alt="Proof of damage"
                  on:error={handleImageError}
                  on:click={() => openImageModal(request.proofOfDamage)}
                  class="thumbnail"
                />
              </div>
            {/if}
            <p class="request-status">
              <strong>Status:</strong>
              <span class="status-badge status-{request.status.toLowerCase().replace(' ', '-')}">
                {request.status}
              </span>
            </p>
            <small class="post-date">
              Posted on: {new Date(request.requestDate).toLocaleString()}
            </small>
          </div>
        {/each}
      </div>
      <button class="addbutton" on:click={openModal}>Send Maintenance Request</button>
    {/if}
  </div>
</div>

{#if showModal}
  <div class="modal-overlay" on:click|self={closeModal}>
    <div class="modal-content">
      <div class="modal-header">
        <h3 class="modal-title">Send Maintenance Request</h3>
        <button on:click={closeModal} class="modal-close-button">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            class="close-icon"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M6 18L18 6M6 6l12 12"
            />
          </svg>
        </button>
      </div>
      {#if error}
        <p class="error-message">{error}</p>
      {/if}
      <form on:submit|preventDefault={handleSubmit}>
        <div class="form-group">
          <label for="description">Description:</label>
          <textarea id="description" bind:value={description} required />
        </div>
        <div class="form-group">
          <label for="proofOfDamage">Proof of Damage:</label>
          <input
            type="file"
            id="proofOfDamage"
            accept="image/*"
            on:change={handleFileChange}
            required
          />
        </div>
        <button type="submit" class="button primary">Submit Request</button>
      </form>
    </div>
  </div>
{/if}

<style>
  .container {
    max-width: 800px;
    margin-top: 20px;
    margin-left: 435px;
    margin-right: auto;
    padding: 20px;
    background-color: #f8f9fa;
    border-radius: 10px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.05);
  }

  h2 {
    text-align: center;
    margin-bottom: 20px;
    color: #343a40;
  }

  .divider {
    border: 0;
    height: 1px;
    background-color: #e0e0e0; /* Light gray color for the divider */
    margin: 20px 0; /* Spacing above and below the divider */
  }

  .loading-message {
    text-align: center;
    color: #555;
  }

  .error-message {
    color: #dc3545;
    text-align: center;
    margin-bottom: 10px;
  }

  .add-request-button {
    display: block;
    margin: 30px auto; /* Increased margin-top for more spacing */
  }

  .form-section {
    margin-bottom: 30px;
  }

  .form-section h3 {
    text-align: center;
    margin-bottom: 20px;
    color: #495057;
  }

  .form-card {
    background-color: #fff;
    border-radius: 12px;
    padding: 25px;
    box-shadow: 0 6px 12px rgba(0, 0, 0, 0.08);
    width: 80%;
    max-width: 600px;
    margin: 0 auto;
    transition: box-shadow 0.3s ease;
  }

  .form-card:hover {
    box-shadow: 0 8px 16px rgba(0, 0, 0, 0.12);
  }

  form {
    display: flex;
    flex-direction: column;
    align-items: stretch;
    margin: 0 auto;
    width: 100%;
  }

  .form-group {
    margin-bottom: 15px;
    display: flex;
    flex-direction: column;
  }

  .form-group label {
    text-align: left;
    margin-bottom: 5px;
    color: #495057;
  }

  textarea {
    padding: 12px;
    border: 1px solid #ced4da;
    border-radius: 6px;
    width: 100%;
    box-sizing: border-box;
    transition: border-color 0.2s ease;
    height: 200px;
  }

  textarea:focus {
    border-color: #007bff;
    outline: none;
  }

  input[type='file'] {
    padding: 12px;
    border: 1px solid #ced4da;
    border-radius: 6px;
    width: 100%;
    box-sizing: border-box;
    transition: border-color 0.2s ease;
  }

  input[type='file']:focus {
    border-color: #007bff;
    outline: none;
  }

  button {
    padding: 12px 25px;
    color: white;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    font-size: 1rem;
    margin: 10px 0;
    transition: background-color 0.3s ease;
  }

  button.primary {
    background-color: #007bff;
  }

  button.primary:hover {
    background-color: #0056b3;
  }

  .request-history {
    margin-top: 30px;
  }

  .request-history h3 {
    text-align: center;
    margin-bottom: 20px;
    color: #343a40;
  }

  .request-cards {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
    justify-content: center;
  }

  .request-card {
    background-color: #fff;
    border-radius: 12px;
    box-shadow: 0 6px 12px rgba(0, 0, 0, 0.08);
    padding: 20px;
    width: 300px;
    transition: box-shadow 0.3s ease;
  }

  .request-card:hover {
    box-shadow: 0 8px 16px rgba(0, 0, 0, 0.12);
  }

  .card-header {
    margin-bottom: 10px;
    border-bottom: 1px solid #e9ecef;
    padding-bottom: 10px;
  }

  .request-title {
    margin-top: 0;
    color: #343a40;
  }

  .request-description {
    margin-bottom: 5px;
    color: #495057;
  }

  .request-status {
    margin-bottom: 5px;
    display: flex;
    align-items: center;
    gap: 5px;
  }

  .status-badge {
    padding: 5px 10px;
    border-radius: 5px;
    font-size: 0.9em;
    font-weight: 500;
    text-align: center;
  }

  .status-pending {
    background-color: #fff3cd;
    color: #856404;
  }

  .status-paid {
    background-color: #d4edda;
    color: #155724;
  }

  .status-overdue {
    background-color: #f8d7da;
    color: #721c24;
  }

  .status-partially-paid {
    background-color: #cce5ff;
    color: #004085;
  }

  .post-date {
    font-size: 0.8em;
    color: #555;
    display: block;
    text-align: right;
  }

  /* Modal Overlay */
  .modal-overlay {
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

  /* Modal Content */
  .modal-content {
    background-color: #fff;
    border-radius: 12px;
    padding: 25px;
    box-shadow: 0 6px 12px rgba(0, 0, 0, 0.08);
    width: 80%;
    max-width: 500px;
  }

  .modal-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 10px;
  }

  .modal-title {
    margin: 0;
    color: #495057;
  }

  .modal-close-button {
    background: none;
    border: none;
    cursor: pointer;
    padding: 5px;
  }

  .close-icon {
    width: 20px;
    height: 20px;
    color: #495057;
  }

  .modal-content form {
    display: flex;
    flex-direction: column;
    align-items: stretch;
  }

  .modal-content .form-group {
    margin-bottom: 15px;
    display: flex;
    flex-direction: column;
  }

  .modal-content label {
    text-align: left;
    margin-bottom: 5px;
    color: #495057;
  }

  .modal-content textarea {
    padding: 12px;
    border: 1px solid #ced4da;
    border-radius: 6px;
    width: 100%;
    box-sizing: border-box;
    transition: border-color 0.2s ease;
    height: 200px;
  }

  .modal-content textarea:focus {
    border-color: #007bff;
    outline: none;
  }

  .modal-content input[type='file'] {
    padding: 12px;
    border: 1px solid #ced4da;
    border-radius: 6px;
    width: 100%;
    box-sizing: border-box;
    transition: border-color 0.2s ease;
  }

  .modal-content input[type='file']:focus {
    border-color: #007bff;
    outline: none;
  }

  .addbutton{
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

  .image-preview {
    margin: 10px 0;
    text-align: center;
  }

  .thumbnail {
    max-width: 150px;
    max-height: 150px;
    object-fit: cover;
    cursor: pointer;
    transition: transform 0.2s;
  }

  .thumbnail:hover {
    transform: scale(1.05);
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
    cursor: pointer;
  }

  .status-badge {
    padding: 5px 10px;
    border-radius: 5px;
    font-size: 0.9em;
    font-weight: 500;
    text-align: center;
  }

  .status-open {
    background-color: #fff3cd;
    color: #856404;
  }

  .status-in-progress {
    background-color: #cce5ff;
    color: #004085;
  }

  .status-resolved {
    background-color: #d4edda;
    color: #155724;
  }

  .status-closed {
    background-color: #f8d7da;
    color: #721c24;
  }

/* Modal content styling */
.modal-image-content {
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  max-width: 90vw;
  max-height: 90vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  position: relative;
  box-shadow: 0px 5px 15px rgba(0, 0, 0, 0.2);
}


/* Close button positioning */
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
</style>