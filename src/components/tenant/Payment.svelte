<script>
  import { onMount, createEventDispatcher } from 'svelte';
  import { uploadPaymentProof, getPaymentsByTenantId } from '../../lib/api';
  import { useNavigate } from 'svelte-navigator';

  const navigate = useNavigate();
  const dispatch = createEventDispatcher();

  let proofOfPayment = null;
  let error = null;
  let amount = null;
  let paymentDate = null;
  let dueDate = null;
  let userId = null;
  let payments = [];
  let loading = true;
  let showModal = false; // State to show the form modal

  onMount(async () => {
    userId = localStorage.getItem('userId');
    await fetchPayments();
  });

  async function fetchPayments() {
    loading = true;
    try {
      const tenantId = parseInt(localStorage.getItem('userId'), 10);
      payments = await getPaymentsByTenantId(tenantId);
    } catch (err) {
      error = err.message;
    } finally {
      loading = false;
    }
  }

  async function handleSubmit(event) {
    event.preventDefault();
    error = null;

    if (!proofOfPayment) {
      error = 'Please select a payment proof image';
      return;
    }

    const formData = new FormData();
    formData.append('proofOfPayment', proofOfPayment);

    try {
      const tenantId = parseInt(localStorage.getItem('userId'), 10);
      const response = await uploadPaymentProof(formData, tenantId, {
        amount: amount,
        paymentDate: paymentDate,
        dueDate: dueDate,
      });

      console.log('Payment proof sent successfully');
      await fetchPayments(); // Refresh the payments list
      dispatch('paymentSuccess');
      closeModal();
      // Reset form
      proofOfPayment = null;
      amount = null;
      paymentDate = null;
      dueDate = null;
    } catch (err) {
      error = err.message || 'Failed to submit payment proof';
      console.error('Error submitting payment proof', err);
    }
  }

  function handleFileChange(event) {
    proofOfPayment = event.target.files[0];
  }

  function openModal() {
    showModal = true;
  }

  function closeModal() {
    showModal = false;
    error = null;
    proofOfPayment = null;
    amount = null;
    paymentDate = null;
    dueDate = null;
  }
</script>

<div class="container">
  <h2>Send Payment Proof</h2>

  <div class="payments-history-card">
    <h3>Payment History</h3>

    {#if loading}
      <p class="loading-message">Loading payments...</p>
    {:else if payments.length === 0}
      <p class="no-history">No payment history found.</p>
    {:else}
      <div class="table-responsive">
        <table>
          <thead>
            <tr>
              <th>Amount</th>
              <th>Payment Date</th>
              <th>Due Date</th>
              <th>Status</th>
              <th>Proof</th>
            </tr>
          </thead>
          <tbody>
            {#each payments as payment}
              <tr>
                <td>₱ {payment.amount}</td>
                <td>
                  {payment.paymentDate
                    ? new Date(payment.paymentDate).toLocaleDateString()
                    : 'Pending'}
                </td>
                <td>{new Date(payment.dueDate).toLocaleDateString()}</td>
                <td>
                  <span
                    class="status-badge status-{payment.status
                      .toLowerCase()
                      .replace(' ', '-')}">{payment.status}</span>
                </td>
                <td>
                  {#if payment.proofOfPayment}
                    <img
                      src={payment.proofOfPayment}
                      alt="Proof of Payment"
                      class="payment-proof-thumbnail"
                    />
                  {:else}
                    No proof submitted
                  {/if}
                </td>
              </tr>
            {/each}
          </tbody>
        </table>
      </div>
    {/if}
    <button class="addbutton" on:click={openModal}>Add Payment</button>
  </div>
</div>

{#if showModal}
  <div class="modal-overlay" on:click|self={closeModal}>
    <div class="modal-content">
      <div class="modal-header">
        <h3 class="modal-title">Send Payment Proof</h3>
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
          <label for="proofOfPayment">Proof of Payment:</label>
          <input
            type="file"
            id="proofOfPayment"
            on:change={handleFileChange}
            accept="image/*"
            required
          />
        </div>
        <div class="form-group">
          <label for="amount">Amount Paid:</label>
          <input type="text" id="amount" bind:value={amount} required />
        </div>
        <div class="form-group">
          <label for="paymentDate">Payment Date:</label>
          <input type="date" id="paymentDate" bind:value={paymentDate} required />
        </div>
        <div class="form-group">
          <label for="dueDate">Due Date:</label>
          <input type="date" id="dueDate" bind:value={dueDate} required />
        </div>
        <button type="submit" class="button primary">Send Proof of Payment</button>
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

  .no-history {
    text-align: center;
    color: #555;
  }

  .add-payment-button {
    display: block;
    margin: 10px auto;
  }

  .form-card {
    background-color: #fff;
    border-radius: 12px;
    padding: 25px;
    box-shadow: 0 6px 12px rgba(0, 0, 0, 0.08);
    width: 80%;
    max-width: 600px;
    margin: 20px auto;
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

  input[type='file'],
  input[type='date'],
  input[type='text'] {
    padding: 12px;
    border: 1px solid #ced4da;
    border-radius: 6px;
    width: 100%;
    box-sizing: border-box;
    transition: border-color 0.2s ease;
  }

  input[type='file']:focus,
  input[type='date']:focus,
  input[type='text']:focus {
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
    align: left;
  }

  button.primary {
    background-color: #007bff;
  }

  button.primary:hover {
    background-color: #0056b3;
  }

  .payments-history-card {
    margin-top: 25px;
    padding: 20px;
    border: none;
    border-radius: 12px;
    background-color: #fff;
    text-align: center;
    box-shadow: 0 6px 12px rgba(0, 0, 0, 0.08);
  }

  .payments-history-card h3 {
    margin-bottom: 20px;
    color: #495057;
  }

  .table-responsive {
    overflow-x: auto;
  }

  table {
    width: 100%;
    border-collapse: separate;
    border-spacing: 0;
    margin-top: 20px;
    border-radius: 12px;
    overflow: hidden;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  }

  th,
  td {
    padding: 12px 15px;
    text-align: left;
    border: none;
    border-bottom: 1px solid #f0f0f0;
  }

  thead th {
    background-color: #f8f9fa;
    font-weight: 500;
    color: #555;
  }

  tbody tr {
    background-color: #fff;
    border-bottom: 1px solid #f0f0f0;
  }

  tbody tr:last-child td {
    border-bottom: none;
  }

  .payment-proof-thumbnail {
    max-width: 80px;
    max-height: 80px;
    border-radius: 10px;
    cursor: pointer;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
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

  .modal-content input[type='password'],
  .modal-content input[type='date'],
  .modal-content input[type='text'] {
    padding: 12px;
    border: 1px solid #ced4da;
    border-radius: 6px;
    width: 100%;
    box-sizing: border-box;
    transition: border-color 0.2s ease;
  }

  .modal-content input[type='password']:focus,
  .modal-content input[type='date']:focus,
  .modal-content input[type='text']:focus {
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
</style>