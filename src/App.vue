<script setup lang="ts">
import { ref } from 'vue'
import Button from 'primevue/button'
import 'primeicons/primeicons.css'

const status = ref('Online')
</script>

<template>
  <header>
    <div class="wrapper">
      <h1>Minecraft Server Starter</h1>
    </div>
  </header>
  <main>
    <h3
      :class="{
        red: status === 'Offline',
        green: status === 'Online'
      }"
    >
      Server Status: {{ status }}
    </h3>
    <Button
      class="button"
      v-if="status === 'Offline'"
      icon="pi pi-play"
      label="Start Server"
      @click="status = 'Online'"
    />
    <Button
      class="button"
      v-else
      icon="pi pi-times"
      label="Stop Server"
      severity="danger"
      @click="status = 'Offline'"
    />
  </main>
</template>

<style scoped>
.red {
  color: rgb(233, 122, 122);
}

.green {
  color: rgb(122, 233, 146);
}

.button {
  width: 50%;
}

h1 {
  font-weight: 700;
  font-size: 4rem;
}

h3 {
  font-weight: 600;
  font-size: 1.6rem;
}

header {
  line-height: 1.5;
  max-height: 100vh;
  text-align: center;
}

main {
  display: flex;
  text-align: center;
  flex-direction: column;
  align-items: center;
  gap: 2rem;
  justify-content: center;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

nav {
  width: 100%;
  font-size: 12px;
  text-align: center;
  margin-top: 2rem;
}

nav a.router-link-exact-active {
  color: var(--color-text);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid var(--color-border);
}

nav a:first-of-type {
  border: 0;
}

@media (max-width: 1024px) {
  main {
    padding-top: 3rem;
  }

  h1 {
    font-weight: 700;
    font-size: 2rem;
  }

  h3 {
    font-weight: 600;
    font-size: 1rem;
  }
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
    margin-left: 4rem;
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  nav {
    text-align: left;
    margin-left: -1rem;
    font-size: 1rem;

    padding: 1rem 0;
    margin-top: 1rem;
  }
}
</style>
