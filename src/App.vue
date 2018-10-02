<template>
  <div id="app">
    <b-navbar id="nav" toggleable="md" type="dark" variant="info">
      <b-navbar-toggle target="nav_collapse"></b-navbar-toggle>
      <b-navbar-brand><router-link to="/" class="brand">Epic Game Nights</router-link></b-navbar-brand>
      <b-collapse is-nav id="nav_collapse">
        <b-navbar-nav>
          <b-nav-item><router-link to="/">Accueil</router-link></b-nav-item>
          <b-nav-item><router-link to="/add">Ajouter un jeu</router-link></b-nav-item>
          <b-nav-item><router-link to="/create-night">Prévoir une Game Night</router-link></b-nav-item>
        </b-navbar-nav>
      </b-collapse>
    </b-navbar>
    <router-view/>
  </div>
</template>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
#nav {
  a {
    color: #ffffff;
    &.router-link-exact-active:not(.brand) {
      color: #b5dac9;
    }
  }
}
</style>
