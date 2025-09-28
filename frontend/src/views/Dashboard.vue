<template>
  <div class="dashboard">
    <div class="dashboard-header">
      <h1>DASHBOARD PENGGUNA</h1>
      <button @click="logout" class="btn btn-logout">Logout</button>
    </div>

    <div class="dashboard-content">
      <SearchBar
        v-model="searchQuery"
        @search="handleSearch"
      />

      <UserTable
        :users="filteredUsers"
        @edit="handleEditUser"
        @delete="handleDeleteUser"
      />
    </div>

    <!-- Modal Konfirmasi Delete -->
    <div v-if="showDeleteModal" class="modal-overlay" @click="closeDeleteModal">
      <div class="modal" @click.stop>
        <h3>Konfirmasi Hapus</h3>
        <p>Apakah Anda yakin ingin menghapus user <strong>{{ userToDelete && nama }}</strong>?</p>
        <div class="modal-buttons">
          <button @click="confirmDelete" class="btn btn-danger">Ya, Hapus</button>
          <button @click="closeDeleteModal" class="btn btn-secondary">Batal</button>
        </div>
      </div>
    </div>

    <!-- Loading Overlay -->
    <div v-if="loading" class="loading-overlay">
      <div class="loading-spinner">Loading...</div>
    </div>
  </div>
</template>

<script>
import { mapActions, mapState } from 'vuex'
import SearchBar from '@/components/SearchBar.vue'
import UserTable from '@/components/UserTable.vue'
import api from '@/services/api'

export default {
  name: 'Dashboard',
  components: {
    SearchBar,
    UserTable
  },

  data () {
    return {
      users: [],
      searchQuery: '',
      showDeleteModal: false,
      userToDelete: null,
      loading: false
    }
  },

  computed: {
    ...mapState(['user']),

    filteredUsers () {
      if (!this.searchQuery.trim()) {
        return this.users
      }

      const query = this.searchQuery.toLowerCase()
      return this.users.filter(user =>
        user.nama.toLowerCase().includes(query) ||
        user.username.toLowerCase().includes(query)
      )
    }
  },

  async created () {
    await this.loadUsers()
  },

  methods: {
    ...mapActions(['logout']),

    async loadUsers () {
      this.loading = true
      try {
        const response = await api.get('/users')
        this.users = response.data
      } catch (error) {
        console.error('Error loading users:', error)
        alert('Gagal memuat data pengguna')
      } finally {
        this.loading = false
      }
    },

    handleSearch (query) {
      this.searchQuery = query
    },

    handleEditUser (user) {
      this.$router.push(`/update-user/${user.id}`)
    },

    handleDeleteUser (user) {
      this.userToDelete = user
      this.showDeleteModal = true
    },

    closeDeleteModal () {
      this.showDeleteModal = false
      this.userToDelete = null
    },

    async confirmDelete () {
      if (!this.userToDelete) return

      this.loading = true
      try {
        await api.delete(`/users/${this.userToDelete.id}`)
        await this.loadUsers() // Reload data
        this.closeDeleteModal()
        alert('User berhasil dihapus')
      } catch (error) {
        console.error('Error deleting user:', error)
        alert('Gagal menghapus user')
      } finally {
        this.loading = false
      }
    },

    logout () {
      this.$store.dispatch('logout')
      this.$router.push('/login')
    }
  }
}
</script>

<style scoped>
.dashboard {
  min-height: 100vh;
  background-color: #f8f9fa;
  padding: 2rem;
}

.dashboard-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

.dashboard-header h1 {
  color: #333;
  margin: 0;
}

.btn-logout {
  padding: 0.5rem 1rem;
  background-color: #6c757d;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.btn-logout:hover {
  background-color: #5a6268;
}

.dashboard-content {
  max-width: 1200px;
  margin: 0 auto;
}

/* Modal Styles */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal {
  background: white;
  padding: 2rem;
  border-radius: 8px;
  max-width: 400px;
  width: 90%;
}

.modal h3 {
  margin-top: 0;
  color: #333;
}

.modal p {
  color: #666;
  margin: 1rem 0;
}

.modal-buttons {
  display: flex;
  gap: 1rem;
  justify-content: flex-end;
}

.btn {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.btn-danger {
  background-color: #dc3545;
  color: white;
}

.btn-danger:hover {
  background-color: #c82333;
}

.btn-secondary {
  background-color: #6c757d;
  color: white;
}

.btn-secondary:hover {
  background-color: #5a6268;
}

/* Loading Overlay */
.loading-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.3);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 2000;
}

.loading-spinner {
  background: white;
  padding: 2rem;
  border-radius: 8px;
  font-size: 1.2rem;
  color: #333;
}

@media (max-width: 768px) {
  .dashboard {
    padding: 1rem;
  }

  .dashboard-header {
    flex-direction: column;
    gap: 1rem;
    align-items: flex-start;
  }
}
</style>
