<template>
  <nav>
    <router-link to="/">Home</router-link> |
    <router-link to="/about">About</router-link>
  </nav>
  <router-view />
</template>

<style lang="scss">
nav {
  padding: 10px;
  background: #f0f0f0;
  a {
    color: #42b983;
    margin: 0 10px;
  }
}
</style>
