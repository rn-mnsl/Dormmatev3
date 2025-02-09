<script>
  import { onMount } from 'svelte';
  import { getAnnouncements, createAnnouncement, deleteAnnouncement, updateAnnouncement } from '../../lib/api';

  let announcements = [];
  let loading = true;
  let error = null;
  let newAnnouncementTitle = '';
  let newAnnouncementContent = '';
  let showModal = false;
  let deletingAnnouncementId = null;
  let showDeleteConfirmation = false;
  let editingAnnouncementId = null;
  let editAnnouncementTitle = "";
  let editAnnouncementContent = "";
  let userId = null; // Declare userId variable

  onMount(async () => {
    fetchAnnouncements();
    // Get the userId from localStorage
    const storedUserId = localStorage.getItem('userId');
    if (storedUserId) {
      userId = parseInt(storedUserId, 10); // Convert to integer
    } else {
      userId = 1; // Default userId if not found in localStorage
    }
  });

  async function fetchAnnouncements() {
    loading = true;
    try {
      announcements = await getAnnouncements();
      announcements.sort((a, b) => new Date(b.postDate) - new Date(a.postDate)); // Sort by most recent date
    } catch (err) {
      error = err.message;
    } finally {
      loading = false;
    }
  }

  async function handleSubmit() {
    if (!userId) {
      error = "User ID is not available. Please log in again.";
      return;
    }

    try {
      const response = await createAnnouncement({
        title: newAnnouncementTitle,
        content: newAnnouncementContent
      }, userId); // Use the userId from localStorage

      await fetchAnnouncements();

      newAnnouncementTitle = "";
      newAnnouncementContent = "";

      showModal = false;

      console.log('New Announcement Created', response);
    } catch (err) {
      error = err.message;
    }
  }

  function confirmDelete(announcementId) {
    deletingAnnouncementId = announcementId;
    showDeleteConfirmation = true;
  }

  function cancelDelete() {
    deletingAnnouncementId = null;
    showDeleteConfirmation = false;
  }

  async function handleDelete() {
    try {
      await deleteAnnouncement(deletingAnnouncementId);
      await fetchAnnouncements();
      deletingAnnouncementId = null;
      showDeleteConfirmation = false;

      console.log("Announcement Deleted");
    } catch (error) {
      error = error.message;
    }
  }

  function startEdit(announcement) {
    editingAnnouncementId = announcement.announcementId;
    editAnnouncementTitle = announcement.title;
    editAnnouncementContent = announcement.content;
  }

  async function updateAnnouncementData(announcementId) {
    try {
      const response = await updateAnnouncement(announcementId, {
        title: editAnnouncementTitle,
        content: editAnnouncementContent
      });
      await fetchAnnouncements();

      editingAnnouncementId = null;
      console.log("Announcement updated");
    } catch (err) {
      error = err.message;
    }
  }

  function cancelEdit() {
    editingAnnouncementId = null;
  }
</script>

<div class="main-container">
<div class="main-header">Bulletin Board</div>
<p class="subtext"> Post announcements related to dorm updates, power interruptions, or other important information. </p>

{#if loading}
  <p>Loading announcements...</p>
{:else if error}
  <p>Error: {error}</p>
{:else}
  <div class="announcement-list">
    {#each announcements as announcement}
      <div class="announcement-card">
        <div class="card-header">
             {#if editingAnnouncementId === announcement.announcementId}
                <input type="text" bind:value={editAnnouncementTitle}/>
              {:else}
              <div class="title-date-container">
                <h3>{announcement.title}</h3>
                <small class="post-date">Posted on: {new Date(announcement.postDate).toLocaleString()}</small>        
              </div>  
                <div class="posted-by"> Posted by Tess Lazaro</div>
              {/if}
        </div>

        <p>
          {#if editingAnnouncementId === announcement.announcementId}
              <input type="text" bind:value={editAnnouncementContent}/>
          {:else}
              {announcement.content}
          {/if}
        </p>

        <!-- Edit Delete Button -->
        <div class="button-container">
          {#if editingAnnouncementId === announcement.announcementId}
              <button on:click={() => updateAnnouncementData(announcement.announcementId)} class = "editbutton"> Update</button>
              <button on:click={cancelEdit} class="deletebutton">Cancel</button>
                {:else}
                  <button on:click={() => confirmDelete(announcement.announcementId)} class="deletebutton"> Delete</button>
                  <button on:click={() => startEdit(announcement)} class = "editbutton">Edit</button>
                {/if}  
              </div>
      </div>
    {/each}
  </div>
{/if}

<!-- Announcement Button -->
<button on:click={() => (showModal = true)} class = "addbutton">Add Announcement</button>

<!-- Announcement Modal -->
{#if showModal}
  <div class="modal">
    <div class="modal-content">
      <form on:submit|preventDefault={handleSubmit}>
        <h2>Add New Announcement</h2>
        <div>
          <label for="title">Announcement Title:</label>
          <input type="text" id="title" bind:value={newAnnouncementTitle} required />
        </div>
        <div>
          <label for="content">Announcement Content:</label>
          <textarea
            id="content" 
            bind:value={newAnnouncementContent} 
            required 
            rows="3"
            style="resize: vertical; width: 100; box-sizing: border-box;"
            ></textarea>
        </div>
        <div class="modal-buttons">
        <button type="submit">Add Announcement</button>
        <button on:click={() => (showModal = false)}>Cancel</button>
        </div>
      </form>
    </div>
  </div>
{/if}

<!-- Delete Confirmation Modal --> 
{#if showDeleteConfirmation}
    <div class="modal">
      <div class="modal-content">
        <h2>Confirm Deletion</h2>
        <p>Are you sure you want to delete this announcement?</p>
        <div>
             <button on:click={handleDelete} class="editbutton">Yes</button>
           <button on:click={cancelDelete} class="deletebutton">Cancel</button>
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

  .announcement-list {
  display: flex;
  flex-direction: column;
  gap: 20px;
  }

  .announcement-card {
  background-color: white;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  position: relative;
  }

  .posted-by {
  font-size: 0.8em;
  color: #666;
  margin-top: 0;
  margin-bottom: 5px;
  } 
  .card-header {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  margin-bottom: 10px; 
  }

  .title-date-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%
  }

  .button-container {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-top: 10px;
  }
  .announcement-card h3 {
  margin-top: 0;
  margin-bottom: 5px;
  }
  .announcement-card p {
  margin-bottom: 10px;
  line-height: 1.5;
  }

  .post-date {
  font-size: 0.8em;
  color: gray;
  margin-left: 10px;
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
  margin-bottom: 20px;
  display: flex;
  flex-direction: column;
  width: 100%;
  }

  label {
  text-align: left; /* Align labels to the left */
  margin-bottom: 5px;
  }
  input[type="text"]{
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  width: 100%; /* Make input fields fill their container */
  box-sizing: border-box; 
  font-size: 1em;    
  }
  button {
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin:  0 10px 10px 0;
  white-space: nowrap;
  }
  button:hover {
  background-color: #0056b3;
  }
  .deletebutton {
  background-color: red;
  }

  .deletebutton:hover {
  background-color: rgb(135, 0, 0);
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
  text-align: left;
  width: 600px;
  }

  .modal-buttons {
  display: flex;
  justify-content: center;
  gap: 10px;
  width: 50%;
  margin-top: 20px;
  }

  textarea {
  padding: 12px; /* Match input field padding */
  border: 1px solid #ddd;
  border-radius: 4px;
  width: 100%; /* Make textarea take full width */
  font-size: 1em; /* Match input field font size */
  resize: vertical; /* Allow vertical resizing */
  box-sizing: border-box; /* Ensure padding and border are included in width */
  min-height: 100px; /* Set a minimum height */
  }
</style>