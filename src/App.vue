<template>
  <div class="app">
    <MapLocation
      :apiKey="apiKey"
      :location="currentLocation"
      @location-updated="handleLocationUpdated"
    />

    <DisplayTable
      :items="searchHistory"
      @delete-selected="deleteSelected"
    />
  </div>
</template>

<script>
import MapLocation from './components/MapLocation.vue';
import DisplayTable from './components/DisplayTable.vue';

export default {
  name: 'App',

  components: {
    MapLocation,
    DisplayTable,
  },

  data() {
    return {
      apiKey: process.env.VUE_APP_API_KEY,
      currentLocation: {
        lat: 0,
        lng: 0,
      },
      searchHistory: [],
    }
  },

  methods: {
    handleLocationUpdated(location) {
      this.searchHistory.unshift({
        id: this.searchHistory.length + 1,
        location: `(${location.lat}, ${location.lng})`,
      })
    },

    deleteSelected(selectedItems) {
      for (const item of selectedItems) {
        const index = this.searchHistory.indexOf(item);
        if (index !== -1) {
          this.searchHistory.splice(index, 1);
        }
      }
    },
  },
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;500;700&display=swap');

.app {
  font-family: 'Roboto', sans-serif;
}
</style>