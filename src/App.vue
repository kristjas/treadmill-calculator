<template>
  <nav>
    <router-link to="/">
      <img :src="logo" alt="Logo" class="nav-logo" />
      Home - Calculator
    </router-link>
    |
    <router-link to="/about">About</router-link>
  </nav>
  <router-view/>
</template>

<script setup>
import logo from '@/assets/logo.png';
</script>

<style>
html, body {
  margin: 0;
  padding: 0;
  height: 100%;
  background-color: #ffffff;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #e0e0e0;
  background-color: #ffffff;
  min-height: 100vh;
}


nav {
  background-color: #484848;
  padding: 10px 20px;
  display: flex;
  justify-content: flex-start;
  align-items: center;
  gap: 12px; 
  box-shadow: 0 2px 4px rgba(0,0,0,0.3);
  border-bottom: 1px solid #333;
  font-size: 14px; 
}


nav a {
  font-weight: 600;
  color: #acacac;
  text-decoration: none;
  font-size: 14px; 
  transition: color 0.2s;
}

nav a:hover {
  color: #ffffff;
}

nav a.router-link-exact-active {
  color: #000000;
}


.nav-logo {
  width: 28px;  
  height: 28px;
  margin-right: 6px;
  vertical-align: middle;
  object-fit: contain;
}
</style>

