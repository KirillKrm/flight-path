<template>
  <FlightPath />
</template>

<script>
import FlightPath from './components/FlightPath.vue';

export default {
  name: 'App',
  components: {
    FlightPath
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  @apply w-full h-full;
}
</style>
