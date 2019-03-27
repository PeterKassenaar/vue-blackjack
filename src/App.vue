<template>
  <div id="app">
    <div class="header">
      <h1>Welcome to Vue.js Blackjack!</h1>
    </div>
    <router-view></router-view>
  </div>
</template>

<style>
.header {
  background-color: #FF0000;
  padding: 2px;
  color: white;
}

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;

}
</style>
