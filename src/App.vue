<template>
  <div>
      <MineSweeper/>
  </div>
</template>

<script>

import MineSweeper from "./components/MineSweeper.vue"
export default {
  components:{
    MineSweeper
  },

}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  margin-top: 30px;
 
}
</style>
