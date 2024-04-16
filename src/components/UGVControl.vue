<template>
  <button class="startEngineButton" @click="startEngine" :disabled="engineStarted">Start Engine</button>
</template>

<script>
export default {
  name: 'UGVControl',
  props: ['engineStarted', 'ugvLocation'],
  data() {
    return {
      ugvMarker: null,
    };
  },
  mounted() {
    window.addEventListener('keydown', this.handleKeyDown);
  },
  beforeDestroy() {
    window.removeEventListener('keydown', this.handleKeyDown);
  },
  methods: {
    startEngine() {
      this.$emit('start-engine');
    },
  },
};
</script>