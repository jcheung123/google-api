<template>
  <div>
    <h3>Search History</h3>
    <button @click="deleteSelected" :disabled="selectedItems.length === 0">Delete Selected Locations</button>
    <table>
      <thead>
        <tr>
          <th></th>
          <th>Location Coordinates:</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in paginatedItems" :key="item.id">
          <td>
            <input type="checkbox" v-model="selectedItems" :value="item" />
          </td>
          <td>{{ item.location }}</td>
        </tr>
      </tbody>
    </table>
    <div class="pagination">
      <button @click="previousPage" :disabled="pagination.page === 1">&lt; Prev</button>
      <span>{{ pagination.page }}</span>
      <button @click="nextPage" :disabled="pagination.page >= totalPages">Next &gt;</button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'DisplayTable',

  props: {
    items: {
      type: Array,
      required: true,
    },
  },

  data() {
    return {
      selectedItems: [],
      pagination: {
        page: 1,
        perPage: 10,
      },
    }
  },

  computed: {
    paginatedItems() {
      const start = (this.pagination.page - 1) * this.pagination.perPage
      const end = start + this.pagination.perPage
      return this.items.slice(start, end)
    },

    totalPages() {
      return Math.ceil(this.items.length / this.pagination.perPage)
    },
  },

  methods: {
    previousPage() {
      if (this.pagination.page > 1) {
        this.pagination.page--
      }
    },

    nextPage() {
      if (this.pagination.page < this.totalPages) {
        this.pagination.page++
      }
    },

    deleteSelected() {
      this.$emit('delete-selected', this.selectedItems)
      this.selectedItems = []
    },
  },
}
</script>

