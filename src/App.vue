<template>
  <div>
      <div class="header-container">

        <input type="text" v-model="searchTerm" placeholder="Search by name or email" class="search-input">

        <select v-model="resultsPerPage" class="page-size-select">
          <option value="10">10</option>
          <option value="15">15</option>
          <option value="20">20</option>
          <option value="all">All</option>
        </select>
        
      </div>

    <div v-if="isLoading">
      Loading...
    </div>

    <div v-else-if="!filteredItems.length">
      No data available.
    </div>

    <table v-else>
      <thead>
        <tr>
          <th>ID</th>
          <th>Name</th>
          <th @click="sortItems('email')">Email {{ sortDirection('email') }}</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in paginatedItems" :key="item.id">
          <td>{{ item.id }}</td>
          <td v-if="!item.isEditing">{{ item.name }}</td>
          <td v-else><input type="text" v-model="item.name" class="edit-input"></td>
          <td v-if="!item.isEditing">{{ item.email }}</td>
          <td v-else><input type="text" v-model="item.email" class="edit-input"></td>
          <td class="center-div">
            <button @click="editItem(item)" class="edit-button">Edit</button>
            <button v-if="item.isEditing" @click="saveItem(item)" class="save-button">Save</button>
            <button @click="deleteItem(item.id)" class="delete-button">Delete</button>
          </td>
        </tr>
      </tbody>
    </table>

    <div>
      <button @click="prevPage" :disabled="currentPage === 1" class="nav-button">Previous</button>
      <span>{{ currentPage }}</span>
      <button @click="nextPage" :disabled="currentPage === totalPages" class="nav-button">Next</button>
    </div>

  </div>
</template>

<script>
export default {
  data() {
    return {
      items: [], 
      currentPage: 1,
      resultsPerPage: 10, 
      searchTerm: '',
      sortBy: null,
      sortDesc: false,
      isLoading: true 
    };
  },
  computed: {
    totalPages() {
      if (this.resultsPerPage === 'all') {
        return 1;
      }
      return Math.ceil(this.filteredItems.length / this.resultsPerPage);
    },

    paginatedItems() {
      if (this.resultsPerPage === 'all') {
        return this.filteredItems;
      }
      const start = (this.currentPage - 1) * this.resultsPerPage;
      const end = start + Number(this.resultsPerPage);
      return this.filteredItems.slice(start, end);
    },

    filteredItems() {
      return this.items.filter(item =>
        item.name.toLowerCase().includes(this.searchTerm.toLowerCase()) ||
        item.email.toLowerCase().includes(this.searchTerm.toLowerCase())
      ).sort((a, b) => {
        if (!this.sortBy) return 0;
        const aValue = a[this.sortBy];
        const bValue = b[this.sortBy];
        return this.sortDesc ? bValue.localeCompare(aValue) : aValue.localeCompare(bValue);
      });
    }

  },

  created() {
    this.fetchData();
  },

  methods: {
    
    async fetchData() {
      this.isLoading = true; 
      try {
        const response = await fetch('https://jsonplaceholder.typicode.com/comments');
        if (!response.ok) {
          throw new Error('Failed to fetch data');
        }
        const data = await response.json();
        this.items = data.map(item => ({ ...item, isEditing: false }));
      } catch (error) {
        console.error(error);
      } finally {
        this.isLoading = false; 
      }
    },

    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++;
      }
    },

    prevPage() {
      if (this.currentPage > 1) {
        this.currentPage--;
      }
    },

    sortItems(column) {
      if (this.sortBy === column) {
        this.sortDesc = !this.sortDesc;
      } else {
        this.sortBy = column;
        this.sortDesc = false;
      }

    },

    sortDirection(column) {
      if (this.sortBy === column) {
        return this.sortDesc ? '▼' : '▲';
      }
      return '';
    },

    editItem(item) {
      item.isEditing = true;
    },

    saveItem(item) {
      item.isEditing = false;
    },

    deleteItem(itemId) {
      this.items = this.items.filter(item => item.id !== itemId);
    }

  }
};
</script>
<style scoped>

.header-container{
  display: flex;
  justify-content: end;
}

.center-div{
  display: flex;
  justify-content: center;
}

table{
  width: 100%;
}
.search-input {
  padding: 8px;
  margin: 10px;
  border-radius: 4px;
  border: 1px solid #ccc;
  font-size: 14px;
}

.page-size-select {
  padding: 8px;
  margin: 10px;
  border-radius: 4px;
  border: 1px solid #ccc;
  font-size: 14px;
}

.edit-input {
  padding: 4px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.edit-button, .save-button, .delete-button, .nav-button {
  padding: 8px 12px;
  margin: 5px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.edit-button {
  background-color: #4CAF50; 
  color: white;
}

.save-button {
  background-color: #2196F3;
  color: white;
}

.delete-button {
  background-color: #f44336; 
  color: white;
}

.nav-button {
  background-color: #555555;
  color: white;
}
</style>
