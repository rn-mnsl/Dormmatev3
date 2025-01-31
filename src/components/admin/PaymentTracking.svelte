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
</script>

<div class="main-container">
<h2>Payment Tracking</h2>

{#if loading}
  <p>Loading payments...</p>
{:else if error}
  <p>Error: {error}</p>
{:else}
  <div>
    <label for="tenant">Select Tenant:</label>
    <select bind:value={selectedTenantId} on:change={updateSelectedTenant}>
      <option value={null}>All</option>
      {#each tenants as tenant}
        <option value={tenant.tenantId}>{tenant.name}</option>
      {/each}
    </select>

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
        {#each payments as payment}
          <tr>
            <td>{payment.paymentId}</td>
            <td>{payment.tenant.name}</td>
            <td> ₱ {payment.amount}</td>
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
                <button on:click={() => updatePaymentStatus(payment.paymentId) } class="editbutton">Update</button>
                <button on:click={cancelEdit} class="deletebutton">Cancel</button>
              </td>
            {:else}
              <td>{payment.status}</td>
              <td>
                {#if payment.proofOfPayment}
                  <img src={payment.proofOfPayment} alt="Proof of Payment" style="max-width: 200px; max-height: 200px;" />
                {:else}
                  No proof submitted
                {/if}
              </td>
              <td>
                <button on:click={() => startEdit(payment)} class="editbutton">Edit Status</button>
                <button on:click={() => removePayment(payment.paymentId)} class="deletebutton">Delete</button>
              </td>
            {/if}
          </tr>
        {/each}
      </tbody>
    </table>
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
    padding: 10px 20px;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    margin: 0 5px;
  }

  button:hover {
    opacity: 0.8;
  }

  img {
    display: block;
    margin: auto;
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
</style>