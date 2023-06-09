<template>
  <div>
    <div class="search-container">
      <h1>Interactive Maps!</h1> <br>
      <input v-model="searchQuery" @keydown.enter="searchLocation" placeholder="Search Location" />
      <button @click="searchLocation">Search</button> 
      <br> OR <br>
      <button @click="getCurrentLocation">Get Current Location</button>
    </div>
    <div id="map" style="height: 400px;"></div>
    <div v-if="timeZone">
      Time Zone: {{ timeZone }}
      <br>
      Local Time: {{ localTime }}
    </div>
    <div v-if="locationNotFound" class="alert">Location not found</div>
  </div>
</template>

<script>
import { onMounted, ref } from 'vue';
import { Loader } from '@googlemaps/js-api-loader';
import axios from 'axios';

export default {
  name: 'MapLocation',

  props: ['apiKey', 'location'],

  setup(props, { emit }) {
    const mapInstance = ref(null)
    const marker = ref(null)
    const loader = ref(null)
    const searchQuery = ref('')
    const timeZone = ref('')
    const localTime = ref('')
    const locationNotFound = ref(false)

    loader.value = new Loader({
      apiKey: props.apiKey,
      version: 'weekly',
    })

    const searchLocation = () => {
      const geocoder = new window.google.maps.Geocoder()

      geocoder.geocode({ address: searchQuery.value }, (results, status) => {
        if (status === window.google.maps.GeocoderStatus.OK) {
          const location = results[0].geometry.location
          mapInstance.value.setCenter(location)
          marker.value.setPosition(location)
          emit('location-updated', { lat: location.lat(), lng: location.lng() })
          getTimeZoneAndLocalTime(location.lat(), location.lng())
          locationNotFound.value = false
        } else {
          locationNotFound.value = true
        }
      })
    }

    const getCurrentLocation = () => {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition((position) => {
          const { latitude, longitude } = position.coords
          const currentLocation = {
            lat: latitude,
            lng: longitude,
          }
          mapInstance.value.setCenter(currentLocation)
          marker.value.setPosition(currentLocation)
          emit('location-updated', currentLocation)
          getTimeZoneAndLocalTime(latitude, longitude)
          locationNotFound.value = false
        })
      } else {
        console.error('Geolocation is not supported by this browser.')
      }
    }

    const getTimeZoneAndLocalTime = (latitude, longitude) => {
      const timestamp = Math.floor(Date.now() / 1000)
      const url = `https://maps.googleapis.com/maps/api/timezone/json?location=${latitude},${longitude}&timestamp=${timestamp}&key=${props.apiKey}`

      axios.get(url)
        .then(response => {
          const { timeZoneId } = response.data
          timeZone.value = timeZoneId
          const localDateTime = new Date().toLocaleString('en-US', { timeZone: timeZoneId })
          localTime.value = localDateTime
        })
        .catch(error => {
          console.error('this response has an error', error)
        })
    }

    onMounted(() => {
      loader.value.load().then(() => {
        mapInstance.value = new window.google.maps.Map(document.getElementById('map'), {
          center: { lat: props.location.lat, lng: props.location.lng },
          zoom: 10,
        })

        marker.value = new window.google.maps.Marker({
          position: props.location,
          map: mapInstance.value,
        })
      })
    })

    return {
      searchQuery,
      mapInstance,
      marker,
      searchLocation,
      getCurrentLocation,
      timeZone,
      localTime,
      locationNotFound,
    }
  },
}
</script>

<style>
#map {
  width: 100%;
  margin: 0 auto;
}

.search-container {
  margin-bottom: 10px;
  text-align: center;
}

button {
  font-family: 'Roboto', sans-serif;
  background-color: rgb(216, 234, 247);;
  color: black;
  padding: 8px;
  border-radius: 4px;
  border: none;
  cursor: pointer;
  margin-right: 10px;
  margin-left: 10px;
}

.alert {
  margin: 10px 0;
  width: 10%;
  padding: 10px;
  border-radius: 4px;
  background-color: #f8d7da;
  color: #721c24;
}
</style>
