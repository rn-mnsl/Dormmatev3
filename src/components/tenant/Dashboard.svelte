<script>
    import { Link } from 'svelte-navigator';
  </script>
  
  <h2>Tenant Dashboard</h2>
  
  <nav>
    <ul>
      <li><Link to="/tenant/information">Unit Management</Link></li>
      <li><Link to="/tenant/payment">Payment Tracking</Link></li>
      <li><Link to="/tenant/announcements">Bulletin Board</Link></li>
      <li><Link to="/tenant/maintenance">Submit Maintenance Request</Link></li>
    </ul>
  </nav>
  
  <p>Welcome to the Tenant Dashboard!</p>