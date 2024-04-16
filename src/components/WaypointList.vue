<template>
  <div>
    <h2>Waypoints</h2>
    <div v-for="(waypoint, index) in waypoints" :key="index">
      <span class="waypoint-name">{{ waypoint.id }}</span>
      <span class="waypoint-name">{{ waypoint.name }}</span>
      <button class="waypoint-button" @click="renameWaypoint(index)">Rename</button>
      <button class="waypoint-button" @click="removeWaypoint(index)">Remove</button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'WaypointList',
  props: ['waypoints'],
  methods: {
    removeWaypoint(index) {
      let waypoint = this.waypoints[index];
      this.$emit('remove-waypoint', waypoint.id);
    },
    renameWaypoint(index) {
      const newName = prompt('Enter new name');
      if (newName) {
        this.$emit('rename-waypoint', { index, newName });
      }
    },
  },
};
</script>

<style scoped>
.waypoint-name {
  margin-right: 10px;
}

.waypoint-button {
  margin-left: 5px;
  margin-right: 5px;
}
</style>