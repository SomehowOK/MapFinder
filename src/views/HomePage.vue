<template>
  <div class="location-button">
    <ion-button @click="moveToCurrentLocation">
      <ion-icon :icon="navigateCircle"></ion-icon>
    </ion-button>
  </div>
  <div class="map-container" v-if="renderComponent">
  <ion-searchbar class ="searchBarTop" :search-icon="search" placeholder="Location" v-model="searchInput"
                 @keyup.enter="moveToLocation(searchInput)"></ion-searchbar>
<l-map ref="map" v-model:zoom="zoom" :center="center" :min-zoom="minZoom" :max-bounds="[[90, -180],[-90, 180]]">
      <l-tile-layer
        url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
        layer-type="base"
        name="OpenStreetMap"
      ></l-tile-layer>
    </l-map>
  </div>
</template>

<script lang="ts">
import "leaflet/dist/leaflet.css";
import { IonButton, IonIcon, IonSearchbar  } from "@ionic/vue";
import { LMap, LTileLayer } from "@vue-leaflet/vue-leaflet";
import { Geolocation } from '@capacitor/geolocation';
import {defineComponent, ref, nextTick} from "vue";
import {navigateCircle, search} from "ionicons/icons";
import { NativeGeocoder } from '@capgo/nativegeocoder';

export default defineComponent({
  components: {
    LMap,
    LTileLayer,
    IonButton,
    IonIcon,
    IonSearchbar
  },
  setup() {

    const searchInput = ref('');
    const minZoom = 3;
    const zoom = ref(5);
    const center = ref([47.41322, -1.219482]);
    const map = ref(null);
    const renderComponent = ref(true);


    // Genz Fragen ob es was schöneres als Force rerender gibt
    const forceRender = async () => {
      renderComponent.value = false;
      await nextTick();
      renderComponent.value = true;
};
    const moveToCurrentLocation = async () => {
      const currentLocation = await Geolocation.getCurrentPosition();
      center.value = [currentLocation.coords.latitude, currentLocation.coords.longitude];
      zoom.value = 13; // Adjust zoom level as needed
      await forceRender();
    };

    const moveToLocation = async (adressString: string) => {
      const coordinates = await NativeGeocoder.forwardGeocode({
        addressString: adressString
      });
      center.value = [coordinates.addresses[0].latitude, coordinates.addresses[0].longitude];
      zoom.value = 13; // Adjust zoom level as needed
      await forceRender();
    };


    return {
      minZoom,
      zoom,
      center,
      map,
      moveToCurrentLocation,
      moveToLocation,
      navigateCircle,
      search,
      searchInput,
      renderComponent
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
  background-color: rgba(255, 255, 255, 0.8); /* Semi-transparent background */
}
</style>
