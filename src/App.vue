
<template>
  <div id="app">
    <map-with-form></map-with-form>
  </div>
  <!-- <div><teste></teste></div> -->
</template>

<script>
import MapWithForm from './components/Mapwithform.vue';
import teste from './components/teste.vue';

export default {
  name: 'App',
  components: {
    MapWithForm,
    teste,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>