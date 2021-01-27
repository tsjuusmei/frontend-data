<template>
  <Chartv2 :data="loadData" />
</template>

<script>
import Chartv2 from "./components/Chartv2.vue";
import cleanData from "./assets/index";

export default {
  name: "App",
  components: {
    Chartv2
  },
  data() {
    return {
      loadData: [],
    };
  },
  mounted() {
    console.log("App loaded");
    this.fetchData();
  },
  methods: {
    async fetchData() {
      let data = await cleanData();
      this.loadData = data;
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
