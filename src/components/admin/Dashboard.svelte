<script>
  import { onMount } from 'svelte';
  import { Link } from 'svelte-navigator';
  import { 
    getUserById,
    getUnits,
    getUsers,
    getPayments,
    getMaintenanceRequests,
    getAnnouncements
  } from '../../lib/api';

  let adminId = null;
  let adminName = null;
  let loading = true;
  let error = null;
  let dashboardStats = {
    totalUnits: 0,
    totalTenants: 0,
    totalPayments: 0,
    totalMaintenanceRequests: 0
  };

  onMount(async () => {
    adminId = localStorage.getItem('userId');
    fetchDashboardData();
  });

  async function fetchDashboardData() {
    loading = true;
    try {
      // Fetch admin info
      const adminData = await getUserById(adminId);
      adminName = adminData.name;

      // Fetch counts for dashboard stats
      const [units, users, payments, maintenance] = await Promise.all([
        getUnits(),
        getUsers(),
        getPayments(),
        getMaintenanceRequests()
      ]);

      dashboardStats = {
        totalUnits: units.length,
        totalTenants: users.length,
        totalPayments: payments.length,
        totalMaintenanceRequests: maintenance.length
      };
    } catch (err) {
      error = err.message;
    } finally {
      loading = false;
    }
  }
</script>

{#if loading}
  <p class="loading-message">Loading dashboard data...</p>
{:else if error}
  <p class="error-message">Error: {error}</p>
{:else}
  <main class="main-content">
    <div class="welcome-container">
      <h2 class="dashboard-title">Welcome, Admin: {adminName}</h2>
      <p class="dashboard-intro">
        This will be your personalized dashboard, providing easy access to essential features as the admin such as rent payments, maintenance requests, announcements management.
      </p>

      <div class="horizontal-divider" />
      <h3 class="dashboard-title-two"> Current Statistics</h3>
      <div class="dashboard-content">
        <div class="stats-grid">
          <div class="stats-card">
            <h3>Total Units</h3>
            <p class="stats-number">{dashboardStats.totalUnits}</p>
          </div>
          <div class="stats-card">
            <h3>Total Tenants</h3>
            <p class="stats-number">{dashboardStats.totalTenants}</p>
          </div>
          <div class="stats-card">
            <h3>Payments</h3>
            <p class="stats-number">{dashboardStats.totalPayments}</p>
          </div>
          <div class="stats-card">
            <h3>Maintenance</h3>
            <p class="stats-number">{dashboardStats.totalMaintenanceRequests}</p>
          </div>
        </div>

        <div class="horizontal-divider" />
        <h3 class="dashboard-title-two"> Features: </h3>
        <div class="features-container">
          <div class="feature-card">
            Unit Management
            <p class="feature-description">
              Add, edit, and manage property units.
              <Link to="/admin/units">
                <button class="dashboard-button">Manage Units</button>
              </Link>
            </p>
          </div>

          <div class="feature-card">
            Tenant Management
            <p class="feature-description">
              Manage tenant information and assignments.
              <Link to="/admin/tenants">
                <button class="dashboard-button">Manage Tenants</button>
              </Link>
            </p>
          </div>

          <div class="feature-card">
            Payment Tracking
            <p class="feature-description">
              Track and verify tenant payments.
              <Link to="/admin/payments">
                <button class="dashboard-button">View Payments</button>
              </Link>
            </p>
          </div>

          <div class="feature-card">
            Bulletin Board
            <p class="feature-description">
              Post and manage announcements.
              <Link to="/admin/announcements">
                <button class="dashboard-button">Manage Announcements</button>
              </Link>
            </p>
          </div>

          <div class="feature-card">
            Maintenance Reports
            <p class="feature-description">
              View and manage maintenance requests.
              <Link to="/admin/maintenance">
                <button class="dashboard-button">View Reports</button>
              </Link>
            </p>
          </div>
        </div>
      </div>
    </div>
  </main>
{/if}

<style>
  .main-content {
  flex-grow: 1;
  padding: 20px;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
  margin-left: 250px;
  }

  .welcome-container {
  width: 885px;
  max-width: 100%;
  margin: 0 auto;
  padding: 20px;
  }

  .dashboard-title {
  color: #343a40;
  font-size: 2.5em;
  font-weight: 700;
  margin-bottom: 10px;
  }

  .dashboard-title-two { 
  color: #343a40;
  font-size: 1.5em; 
  font-weight: 700;
  margin-bottom: 10px; 
  }

  .dashboard-intro {
  color: #000;
  margin-bottom: 30px;
  }

  .stats-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 20px;
  margin-bottom: 30px;
  }

  .stats-card {
  background-color: #f8f9fa;
  border-radius: 10px;
  padding: 20px;
  text-align: center;
  border: 1px solid #dee2e6;
  }

  .stats-number {
  font-size: 2em;
  font-weight: 700;
  color: #007bff;
  margin: 10px 0;
  }

  .horizontal-divider {
  border: 1px solid #dee2e6;
  margin: 20px 0;
  }

  .features-container {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 20px;
  }

  .feature-card {
  border-radius: 10px;
  border: 1px solid #dee2e6;
  padding: 20px;
  font-size: 18px;
  font-weight: 600;
  }

  .feature-description {
  font-size: 14px;
  font-weight: 400;
  margin-top: 10px;
  }

  .dashboard-button {
  display: flex;
  background-color: #007bff;
  color: white;
  padding: 10px 15px;
  border-radius: 5px;
  border: none;
  cursor: pointer;
  margin-top: 10px;
  }

  .dashboard-button:hover {
  background-color: #0056b3;
  }

  @media (max-width: 1024px) {
  .stats-grid, .features-container {
  grid-template-columns: repeat(4, 1fr);
  }
  }

  @media (max-width: 768px) {
  .main-content {
  margin-left: 0;
  }

  .stats-grid, .features-container {
  grid-template-columns: 1fr;
  }
  }
</style>