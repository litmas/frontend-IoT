<template>
  <div id="app">
    <header>
      <h1>IoT Sensor Dashboard</h1>
    </header>
    <main>
      <router-view />
    </main>
    <footer>
      <p>Real-time Temperature & Humidity Monitoring</p>
    </footer>
  </div>
</template>

<script>
export default {
  name: 'App'
};
</script>

<style scoped>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

header {
  background-color: #42b983;
  color: white;
  padding: 1rem;
  text-align: center;
}

main {
  flex: 1;
  padding: 1rem;
}

footer {
  background-color: #f5f5f5;
  padding: 1rem;
  text-align: center;
  font-size: 0.8rem;
}
</style>