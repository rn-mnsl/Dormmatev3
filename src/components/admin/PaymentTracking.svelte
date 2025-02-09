<script>
  import { onMount } from 'svelte';
  import { getPayments, updatePayment, deletePayment, getPaymentsByTenantId } from '../../lib/api';

  let payments = [];
  let tenants = [];
  let loading = true;
  let error = null;
  let editingPaymentId = null;
  let editPaymentStatus = '';
  let selectedTenantId = null;
  let searchQuery = ''; // Add searchQuery variable
  let showImageModal = false;
  let selectedImage = null;
  

  // Computed property to filter units based on searchQuery
  $: filteredPayments = payments.filter(payment =>
    String(payment.paymentId).includes(searchQuery) ||
    payment.tenant.name.toLowerCase().includes(searchQuery.toLowerCase()) ||
    payment.paymentDate.toLowerCase().includes(searchQuery.toLowerCase()) ||
    payment.status.toLowerCase().includes(searchQuery.toLowerCase())
  )
  .sort((a, b) => {
      // Convert payment dates to timestamps for comparison
      const dateA = a.paymentDate ? new Date(a.paymentDate).getTime() : 0;
      const dateB = b.paymentDate ? new Date(b.paymentDate).getTime() : 0;
      // Sort in descending order (newest first)
      return dateB - dateA;
    });

  onMount(async () => {
    fetchPayments();
  });

  async function fetchPayments() {
    loading = true;
    try {
      payments = selectedTenantId ? await getPaymentsByTenantId(selectedTenantId) : await getPayments();
    } catch (err) {
      error = err.message;
    } finally {
      loading = false;
    }
  }

  async function updateSelectedTenant() {
    await fetchPayments();
  }

  function startEdit(payment) {
    editingPaymentId = payment.paymentId;
    editPaymentStatus = payment.status;
  }

  async function updatePaymentStatus(paymentId) {
    try {
      const response = await updatePayment(paymentId, { status: editPaymentStatus });
      await fetchPayments();
      editingPaymentId = null;
    } catch (err) {
      error = err.message;
    }
  }

  async function removePayment(paymentId) {
    if (confirm("Are you sure you want to delete this payment?")) {
      try {
        await deletePayment(paymentId);
        await fetchPayments();
      } catch (err) {
        error = err.message;
      }
    }
  }

  function cancelEdit() {
    editingPaymentId = null;
  }

  function openImageModal(imageUrl) {
    selectedImage = imageUrl;
    showImageModal = true;
  }

  function closeImageModal() {
    selectedImage = null;
    showImageModal = false;
  }

  // Add error handling for images
  function handleImageError(event) {
    event.target.src = '/placeholder-image.png';
    event.target.alt = 'Image not available';
  }
</script>

<div class="main-content">
  <div class="main-header">Payment Tracking</div>
  <p class="subtext"> Mark rent payments as completed.</p>

  <!-- Search Input Field -->
  <div class="search-add-container">
    <div class="search-container">
      <input
        type="text"
        placeholder="Search payment"
        bind:value={searchQuery}
      />
    </div>  
  </div>  

  {#if loading}
    <p>Loading payments...</p>
  {:else if error}
    <p>Error: {error}</p>
  {:else}
      <table>
        <thead>
          <tr>
            <th>ID</th>
            <th>Tenant</th>
            <th>Amount</th>
            <th>Payment Date</th>
            <th>Due Date</th>
            <th>Status</th>
            <th>Proof of Payment</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          {#each filteredPayments as payment}
            <tr>
              <td>{payment.paymentId}</td>
              <td>{payment.tenant.name}</td>
              <td>₱ {payment.amount}</td>
              <td>{payment.paymentDate ? new Date(payment.paymentDate).toLocaleDateString() : "Pending"}</td>
              <td>{new Date(payment.dueDate).toLocaleDateString()}</td>

              {#if editingPaymentId === payment.paymentId}
                <td>
                  <select bind:value={editPaymentStatus}>
                    <option value="pending">Pending</option>
                    <option value="paid">Paid</option>
                    <option value="overdue">Overdue</option>
                    <option value="partially paid">Partially Paid</option>
                  </select>
                </td>
                <td>
                  <button on:click={() => updatePaymentStatus(payment.paymentId)}>Update</button>
                  <button on:click={cancelEdit}>Cancel</button>
                </td>
              {:else}
                <td>{payment.status}</td>
                <td>
                  {#if payment.proofOfPayment}
                  <!-- svelte-ignore a11y-click-events-have-key-events -->
                  <img 
                    src={payment.proofOfPayment} 
                    alt="Proof of Payment" 
                    style="max-width: 200px; max-height: 200px; cursor: pointer;" 
                    on:error={handleImageError}
                    on:click={() => openImageModal(payment.proofOfPayment)}
                  />
                {:else}
                  No proof submitted
                {/if}
                </td>
                <td>
                  <div class="button-container">
                    <button on:click={() => startEdit(payment)} style="background-color: #007bff;">Edit Status</button>
                    <button on:click={() => removePayment(payment.paymentId)} style="background-color: red;">Delete</button>
                  </div>
                </td>
              {/if}
            </tr>
          {/each}
        </tbody>
      </table>
  {/if}

  {#if showImageModal && selectedImage}
  <!-- svelte-ignore a11y-click-events-have-key-events -->
  <div class="modal" on:click={closeImageModal}>
    <div class="modal-image-content" on:click|stopPropagation>
      <img src={selectedImage} alt="Full size proof of payment" />
      <button class="close-button" on:click={closeImageModal}>Close</button>
    </div>
  </div>
{/if}

</div>

<style>

  .main-content {
  margin-left: 300px; /* Adjust this value based on the width of your sidebar */
  padding: 20px;
  max-width: 1200px;
  padding: 20px;
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

  table {
  width: 100%;
  border-collapse: collapse;
  }

  th, td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: center;
  }

  th {
  background-color: #f0f0f0;
  }

  select {
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  background-color: white;
  }

  button {
  size: 10px;
  padding: 10px 10px;
  color: white;
  background-color: #007bff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin: 0 5px;
  }

  button:hover {
  opacity: 0.8;
  background-color: #0056b3;
  }

  img {
  display: block;
  margin: auto;
  }

  .button-container { 
  display: flex;


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
</style>