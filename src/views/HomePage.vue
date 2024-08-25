<template>
  <div class="spinner-container" v-if="loading">
    <ion-spinner name="crescent"></ion-spinner>
  </div>
  <ion-alert :is-open="showAlert" onDidDismiss="showAlert = false" header="Error" @ionAlertDidDismiss="showAlert = false"
             message="Could not find the location. Please try again."></ion-alert>
  <div class="location-button">
    <ion-button @click="moveToCurrentLocation">
      <ion-icon :icon="navigateCircle"></ion-icon>
    </ion-button>
  </div>
  <div class="map-container" v-if="renderComponent">
    <ion-searchbar class ="searchBarTop"
                   :search-icon="search"
                   placeholder="Location"
                   v-model="searchInput"
                   @keyup.enter="moveToLocation(searchInput)"
    ></ion-searchbar>
    <l-map ref="map" v-model:zoom="zoom" :center="center" :min-zoom="minZoom" :max-bounds="[[90, -180],[-90, 180]]">
      <l-tile-layer
          url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
          layer-type="base"
          name="OpenStreetMap"
      ></l-tile-layer>

      <l-marker v-if="currentMarker" :lat-lng="currentMarkerCoordinates">
        <l-icon :icon-url="manIconUrl" :icon-size="[50, 50]" ></l-icon>
      </l-marker>
      <l-marker v-if="searchMarker" :lat-lng="searchMarkerCoordinates">
      </l-marker>
    </l-map>
  </div>
</template>

<script lang="ts">
import "leaflet/dist/leaflet.css";
import { IonButton, IonIcon, IonSearchbar, IonSpinner, IonAlert  } from "@ionic/vue";
import {LMap, LTileLayer, LMarker, LIcon} from "@vue-leaflet/vue-leaflet";
import { Geolocation } from '@capacitor/geolocation';
import { Preferences } from '@capacitor/preferences';
import {defineComponent, ref, nextTick} from "vue";
import {navigateCircle, search, man} from "ionicons/icons";
import { NativeGeocoder } from '@capgo/nativegeocoder';
import manIconUrl from '@/../public/man.png';


export default defineComponent({
  components: {
    LIcon,
    LMap,
    LTileLayer,
    LMarker,
    IonButton,
    IonIcon,
    IonSearchbar,
    IonSpinner,
    IonAlert
  },
  setup() {

    const searchMarker = ref(false)
    const searchMarkerCoordinates = ref([0, 0])
    const currentMarker = ref(false)
    const currentMarkerCoordinates = ref([0, 0])

    const searchInput = ref('');
    const minZoom = 3;
    const zoom = ref(5);
    const center = ref([47.41322, -1.219482]);
    const map = ref(null);
    const renderComponent = ref(true);
    const loading = ref(false);
    const showAlert = ref(false);


    // Marker in anderer Farbe und Größe !!!
    // Theoretisch noch Logo und Icon verändern
    // Genz Fragen ob es was schöneres als Force rerender gibt

    const forceRender = async () => {
      renderComponent.value = false;
      await nextTick();
      renderComponent.value = true;
    };

    const moveToCurrentLocation = async () => {
      // Move the map to the current location
      loading.value = true;
      const currentLocation = await Geolocation.getCurrentPosition();
      center.value = [currentLocation.coords.latitude, currentLocation.coords.longitude];
      zoom.value = 13;
      currentMarker.value = true;
      currentMarkerCoordinates.value = [currentLocation.coords.latitude, currentLocation.coords.longitude];

      await Preferences.set({
        key: 'center',
        value: JSON.stringify(center.value),
      });
      await Preferences.set({
        key: 'zoom',
        value: JSON.stringify(zoom.value),
      });

      await forceRender();
      loading.value = false;
    };

    const moveToLocation = async (adressString: string) => {
      // Move the map to the location specified by the address string
      loading.value = true;
      try {
        const coordinates = await NativeGeocoder.forwardGeocode({
          addressString: adressString
        });
        center.value = [coordinates.addresses[0].latitude, coordinates.addresses[0].longitude];
        zoom.value = 13; // Adjust zoom level as needed
        searchMarker.value = true;
        searchMarkerCoordinates.value = [coordinates.addresses[0].latitude,
          coordinates.addresses[0].longitude];

        await Preferences.set({
          key: 'center',
          value: JSON.stringify(center.value),
        });
        await Preferences.set({
          key: 'zoom',
          value: JSON.stringify(zoom.value),
        });

        await forceRender();
      } catch (error) {
        console.error("Error occurred while moving to location: ", error);
        // You can handle the error here, for example, show a notification to the user
        showAlert.value = true;
      } finally {
        loading.value = false;
      }
    };

    const setupView = async () => {
      // Retrieve the last center and zoom level from the Preferences storage
      const savedCenter = await Preferences.get({ key: 'center' });
      const savedZoom = await Preferences.get({ key: 'zoom' });
      console.log(Preferences.get({ key: 'center' }))
      console.log(Preferences.get({ key: 'zoom' }))

      // If the saved values exist, use them to initialize the center and zoom refs
      if (savedCenter.value) {
        center.value = JSON.parse(savedCenter.value);
        console.log("if savedCenter.value")
        console.log(JSON.parse(savedCenter.value))
      }
      if (savedZoom.value) {
        zoom.value = JSON.parse(savedZoom.value);
        console.log("if savedZoom.value")
        console.log(JSON.parse(savedZoom.value))
      }

      await forceRender();
    };

    // Call the setupView function when the component is created

    console.log("setupView")
    setupView();

    return {
      minZoom,
      zoom,
      center,
      map,
      loading,
      moveToCurrentLocation,
      moveToLocation,
      navigateCircle,
      search,
      man,
      searchInput,
      renderComponent,
      searchMarker,
      searchMarkerCoordinates,
      currentMarker,
      currentMarkerCoordinates,
      showAlert,
      manIconUrl
    };
  },
});
</script>

<style scoped>

.location-button {
  position: absolute;
  right: 20px;
  bottom: 20px;
  z-index: 1000;
}

.map-container {
  width: 100vw;
  height: 100vh;
  position: relative;
}

.searchBarTop {
  width: 80%;
  max-width: 600px;
  position: absolute;
  top: 10px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 1000;
  background-color: rgba(255, 255, 255, 0);
}

.spinner-container {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 1001;
  font-size: 10em;
}

</style>
