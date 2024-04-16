<template>
  <div>
      <!-- Button to start the engine -->
      <button class="startEngineButton" @click="startEngine" :disabled="engineStarted">
        <img class="enigneIcon" :src="engineIcon" alt="Engine Icon">
        Start Engine</button>
      <!-- Map container -->
      <div id="map" style="height: 600px; width: 100%;"></div>
      <!-- Waypoint list component -->
      <WaypointList
        :waypoints="waypoints"
        @remove-waypoint="removeWaypoint"
        @rename-waypoint="renameWaypoint"
      />
  </div>
</template>

<script>
import L from 'leaflet'; // Importing Leaflet library
import 'leaflet/dist/leaflet.css'; // Importing Leaflet CSS
import WaypointList from '@/components/WaypointList.vue'; // Importing WaypointList component

export default {
  name: 'LeafletMap',
  components: {
  WaypointList, // Registering WaypointList component
  },
  data() {
    return {
      map: null, // Map object
      tileLayer: null, // Tile layer object
      layers: [], // Layers array
      engineIcon: require('@/assets/engine.jpg'), // Engine icon

      ugvLocation: [58.38538, 26.72539], // Initial UGV location
      engineStarted: false,
      
      ugvMarker: null,
      waypoints: [], // Waypoints array
    };
  },
  mounted() {
    this.initializeMap(); 
    window.addEventListener('keydown', this.handleKeyDown);  // Add keydown event listener
    this.map.on('popupopen', this.handlePopup); // Add popupopen event listener
  },
  beforeDestroy() {
    window.removeEventListener('keydown', this.handleKeyDown); // Remove keydown event listener
    this.map.off('popupopen', this.handlePopup); // Remove popupopen event listener
  },
  methods: {
      initializeMap() {
        // Initialize the map
          this.map = L.map('map', { keyboard: false }).setView(this.ugvLocation, 13);

          // Initialize the tile layer
          this.tileLayer = L.tileLayer(
              'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
              {
               maxZoom: 19,
              attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
              },
          );
          
          // Add the tile layer to the map
          this.tileLayer.addTo(this.map);

          // Initialize the UGV marker and add it to the map
          this.ugvMarker = L.marker(this.ugvLocation).addTo(this.map);
          this.ugvMarker.bindPopup("<b>UGV</b>").openPopup();

          // Add click event listener to the map
          this.map.on('click', this.addWaypoint);
      },
    handleKeyDown(event) {
      // Handle keydown event
      if (!this.engineStarted) {
        // If the engine is not started, show an alert when arrow keys are pressed
        if (['ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight'].includes(event.key)) {
          alert('Please start the engine before moving the UGV.');
        }
        return;
      }

      // Calculate the delta based on the pressed key
      let delta;

      switch (event.key) {
        case 'ArrowUp':
          delta = [0.0001, 0];
          break;
        case 'ArrowDown':
          delta = [-0.0001, 0];
          break;
        case 'ArrowLeft':
          delta = [0, -0.0001];
          break;
        case 'ArrowRight':
          delta = [0, 0.0001];
          break;
      }

      // Update the UGV location and marker position
      if (delta) {
        this.ugvLocation[0] += delta[0];
        this.ugvLocation[1] += delta[1];
        this.ugvMarker.setLatLng(this.ugvLocation);
        this.map.setView(this.ugvLocation);
      }
    },

    startEngine() {
      this.engineStarted = true;
    },

    // Add a waypoint to the map
    addWaypoint(event) {
      const waypoint = L.marker(event.latlng).addTo(this.map);
      waypoint.id = this.waypoints.length + 1;

      waypoint.name = `Waypoint ${this.waypoints.length + 1}`;

      waypoint.bindPopup(`
          <button id="driveTo">Drive</button>
          <button id="save">Save</button>
          <button id="discard">Discard</button>
        `).openPopup();

      // Setting this.currentWaypoint to the newly created waypoint
      this.currentWaypoint = waypoint;

      // Handle waypoint click event
      waypoint.on('click', () => {
        this.currentWaypoint = waypoint;
        waypoint.bindPopup(`
          <button id="driveTo">Drive</button>
          <button id="save">Save</button>
          <button id="discard">Discard</button>
        `).openPopup();

        this.handlePopup();
      });

        // Handle popupclose event
        // If the waypoint is not saved, remove it from the map
      waypoint.on('popupclose', () => {
        if (!this.waypoints.find(w => w.id === waypoint.id)) {
          this.map.removeLayer(waypoint);
        }
      });
    },

    // Handle popupopen event
    handlePopup() {
      let driveToButton = document.getElementById('driveTo');
      let saveButton = document.getElementById('save');
      let discardButton = document.getElementById('discard');

      driveToButton.onclick = () => this.driveToWaypoint(this.currentWaypoint.getLatLng());
      saveButton.onclick = () => this.saveWaypoint();
      discardButton.onclick = () => this.removeWaypoint(this.currentWaypoint.id);
    },

    // Drive to a waypoint
    driveToWaypoint(latlng) {
      this.ugvLocation = [latlng.lat, latlng.lng];
      this.ugvMarker.setLatLng(this.ugvLocation);
      this.map.setView(this.ugvLocation);

      // If the waypoint is not saved, remove it from the map
      if(this.currentWaypoint && this.waypoints.find(w => w.id !== this.currentWaypoint.id)) {
        this.removeWaypoint(this.currentWaypoint, this.currentWaypoint.id);
      }
      if(this.currentWaypoint){
        this.currentWaypoint.closePopup();
      }
    },

    // Save a waypoint
    saveWaypoint() {
      if (!this.waypoints.find(w => w.id === this.currentWaypoint.id)) {
        // Add the current waypoint to the waypoints array
        this.currentWaypoint.addTo(this.map);
        this.waypoints.push({
          id: this.currentWaypoint.id,
          name: this.currentWaypoint.name,
          latlng: this.currentWaypoint.getLatLng(),
        });
      }
      this.currentWaypoint.closePopup();
      this.currentWaypoint = null;
    },

    // Remove a waypoint
    removeWaypoint(id) {
      const waypoint = this.waypoints[id];

      // Remove the waypoint from the map
      if (waypoint && waypoint.ugvMarker) {
        waypoint.marker.remove();
      }


      // Remove the waypoint from the waypoints array
      if (this.waypoints.find(w => w.id === id)) {
        this.waypoints.splice(id-1, 1);
      }

      if (this.currentWaypoint) {
        this.currentWaypoint.closePopup();
        this.currentWaypoint = null;
      }
    },

    renameWaypoint({ index, newName }) {
      this.waypoints[index].name = newName;
    },
  },


};
</script>

<style scoped>
/* Styles for the map container */
#map {
  height: 100%;
  width: 100%;
}

/* Styles for the start engine button */
.startEngineButton {
  position: absolute;
  display: flex;
  align-items: center;
  justify-content: center;
  top: 10px;
  right: 10px;
  background-color: gray;
  opacity: 0.8;
  border: 2px solid black;
  color: white;
}

/* Styles for the engine icon */
.enigneIcon {
  width: 25px;
  height: 25px;
  margin-right: 10px;

}
</style>