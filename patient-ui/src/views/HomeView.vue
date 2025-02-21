<!-- favicon from https://www.flaticon.com/free-icon/tooth-whitening_1060876 -->

<template>
  <div>
    <div class="banner">
      <h1> Welcome to Dentago! </h1>
      <p>
        Booking dentist appointments in Sweden is difficult: free times seem to be few and far between, and people who don't speak Swedish are at a disadvantage.
        We at Dentago understand these struggles, so we created this centralized platform to help ease the process.
        We collaborate with hundreds of dental clinics to ensure there are always places nearby to choose from.

        Create an account, find clinics near you using the Google Map below, check available booking times, and pick the best one for you! You'll find all your upcoming appointments under your profile.
      </p>
    </div>
    <div class="p-4">
      <!-- Google Map
        - shows all clinics at their location using markers.
        - centers on the user's location if they enable that permission, and gives a different zoom level
        - centers on a "middle point" between Gothenburg and Stockholm with the appropriate zoom level
      -->
      <div class="map-container mt-2">
        <GoogleMap :api-key=API_KEY style="width: 100%; height: 500px" :center="center" :zoom="zoom">
          <Marker v-for="clinic in clinics" :key="clinic.id" :options="{ position: clinic.location }">
            <InfoWindow>
              Clinic name: {{ clinic.name }} <br>
              Opening hours: {{ clinic.hours[0] + '-' + clinic.hours[1] }} <br>
              Clinic booking page
              <BookingButton :clinicId="clinic._id"></BookingButton>
            </InfoWindow>
          </Marker>
      </GoogleMap>
      </div>
    </div>
  </div>
</template>

<style scoped>
h1 {
  font-size: 3em;
  color: var(--primary-color);
  /*filter: drop-shadow(1px 3px #32292F);*/
  font-weight: 600;
}
.banner{
  background-color: var(--secondary-color);
  padding: 2%;
}
</style>

<script>
import { defineComponent } from 'vue';
import { GoogleMap, Marker, InfoWindow } from "vue3-google-map";
import { Api } from '@/Api.js';
import BookingButton from '@/components/BookingButton.vue';

export default defineComponent ({
  // When the page is created, it calls to get all clinics and tries to access the user's location
  created() {
    this.getClinics(),
    this.findUserLocation()
  },
  components: {
    GoogleMap,
    // eslint-disable-next-line vue/no-reserved-component-names
    Marker,
    InfoWindow,
    BookingButton,
  },
  data() {
    return {
      API_KEY: import.meta.env.VITE_API_KEY,
      center: {lat: 58.572053, lng: 14.702880},
      zoom: 7,
      clinics: [],
    }
  },
  methods: {
    async getClinics() {
            Api.get('/clinics', {headers: {Authorization: `Bearer ${localStorage.getItem("access-token")}`}})
        .then(response => {
          for(let i=0; i<response.data.length; i++){
            this.clinics.push(response.data[i]);
          }
        }).catch(error => {
          console.log(error.message);
        })
    },
    /*
    This is taken and adapted from https://developers.google.com/maps/documentation/javascript/geolocation#maps_map_geolocation-javascript
    It uses the HTML5 Geolocation to find the user's position (if they allow it and the browser supports it), so that the map centers at their location.
    Otherwise, it centers at roughly the middle point between Gothenburg and Stockholm.
    */ 
    findUserLocation() {
      // if browser supports HTML5 Geolocation
      if (navigator.geolocation){
        navigator.geolocation.getCurrentPosition(
          // Success case
          (position) => {
            this.center = {
              lat: position.coords.latitude,
              lng: position.coords.longitude
            };
            this.zoom = 13;
          },
          // If user does not approve location services
          () => {
            alert("Permission to access location not approved.")
          }
        );
      } else {
        // if browser does not support HTML5 Geolocation
        alert("Browser does not support Geolocation tools used.")
      }
    }
  }
});
</script>

