<template>
  <div class="map-wrapper">
    <div ref="mapContainer" class="map-container"></div>
    <div class="directions">
      <span class="north">N 360&deg;</span>
      <span class="east">E 90&deg;</span>
      <span class="south">S 180&deg;</span>
      <span class="west">W 270&deg;</span>
    </div>
    <button v-if="!isFlying" class="start-button" @click="startFlight">Почати</button>
    <button v-if="isFlying" class="stop-button" @click="stopFlight">Стоп</button>
  </div>
</template>

<script setup>
import { ref, onMounted, shallowRef } from "vue";
import flightData from "@/data/flight.json";

let L = null;
const mapContainer = ref(null);
const map = shallowRef(null);
const marker = shallowRef(null);
let flightInterval = null;
const isFlying = ref(false);

const startLat = 50.4501;
const startLng = 30.5234;
let currentLat = startLat;
let currentLng = startLng;
let currentDirection = 0;

onMounted(async () => {
  if (typeof window !== "undefined") {
    L = await import("leaflet");
    await import("leaflet/dist/leaflet.css");

    map.value = L.map(mapContainer.value).setView([startLat, startLng], 10);
    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution: "&copy; OpenStreetMap contributors"
    }).addTo(map.value);

    marker.value = L.marker([startLat, startLng], {
      icon: createPlaneIcon(90)
    }).addTo(map.value);
  }
});

const createPlaneIcon = (rotation) => {
  return L.divIcon({
    html: `<img src="plane-icon.png" style="width: 20px; transform: rotate(${rotation - 90}deg); transition: transform 0.5s linear;">`,
    className: "custom-icon",
    iconSize: [40, 40],
    iconAnchor: [20, 20]
  });
};

const calculateNewPosition = (speed, direction) => {
  const distance = (speed / 3600) * 1;
  const deltaLat = (distance * Math.sin(direction * (Math.PI / 180))) / 110.574;
  const deltaLng =
      (distance * Math.cos(direction * (Math.PI / 180))) /
      (111.320 * Math.cos((currentLat * Math.PI) / 180));

  currentLat += deltaLat;
  currentLng += deltaLng;
  currentDirection = direction;
};

const startFlight = () => {
  let index = 0;
  stopFlight();
  isFlying.value = true;

  flightInterval = setInterval(() => {
    if (index >= flightData.length) {
      stopFlight();
      return;
    }

    const { speed, direction } = flightData[index];
    calculateNewPosition(parseFloat(speed), parseFloat(direction));

    if (marker.value) {
      marker.value.setLatLng([currentLat, currentLng]);
      marker.value.setIcon(createPlaneIcon(currentDirection));
      map.value?.panTo([currentLat, currentLng], { animate: true });
    }

    index++;
  }, 500);
};

const stopFlight = () => {
  if (flightInterval) {
    clearInterval(flightInterval);
    flightInterval = null;
    isFlying.value = false;
    currentLat = startLat;
    currentLng = startLng;
    currentDirection = 0;

    if (marker.value) {
      marker.value.setLatLng([startLat, startLng]);
      marker.value.setIcon(createPlaneIcon(90));
      map.value?.panTo([startLat, startLng]);
    }
  }
};
</script>

<style scoped>
.map-wrapper {
  position: absolute;
  width: 100vw;
  height: 100vh;
  top: 0;
  left: 0;
  overflow: hidden;
}

.map-container {
  width: 100%;
  height: 100%;
}

.directions {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  pointer-events: none;
}

.north {
  position: absolute;
  top: 10px;
  left: 50%;
  transform: translateX(-50%);
  color: black;
  font-weight: bold;
  z-index: 1000;
}

.east {
  position: absolute;
  right: 10px;
  top: 50%;
  transform: translateY(-50%);
  color: black;
  font-weight: bold;
  z-index: 1000;
}

.south {
  position: absolute;
  bottom: 40px;
  left: 50%;
  transform: translateX(-50%);
  color: black;
  font-weight: bold;
  z-index: 1000;
}

.west {
  position: absolute;
  left: 10px;
  top: 50%;
  transform: translateY(-50%);
  color: black;
  font-weight: bold;
  z-index: 1000;
}

.start-button {
  position: absolute;
  bottom: 70px;
  left: 50%;
  transform: translateX(-50%);
  background-color: yellow;
  border: none;
  padding: 10px 20px;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  border-radius: 5px;
  z-index: 1000;
}

.stop-button {
  position: absolute;
  bottom: 70px;
  left: 50%;
  transform: translateX(-50%);
  background-color: black;
  color: white;
  border: none;
  padding: 10px 20px;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  border-radius: 5px;
  z-index: 1000;
}
</style>
