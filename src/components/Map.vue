<template>
  <div class="map">{{ mapString }}</div>
</template>

<script>
export default {
  name: 'Map',
  props: {
    mapString: String
  },
  computed: {
  }
}
</script>

<style scoped>
.map {
  padding-left: 10em;
  padding-top: 3em;
  width: 30em;
  height: 24em;
  font-family: 'Menlo', monospace;
  white-space: pre;
  color: white;
  background-color: black;
}
</style>
