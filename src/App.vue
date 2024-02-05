<script setup>
import Tumbler from './components/Tumbler.vue'
import Test from './components/Test.vue'
</script>

<template>
  <!-- <header>

  </header> -->
  <Test 
    :key="gameResetKey"
    @resetGame="resetGame()"
  />
</template>

<script>
export default {
  data() {
    return {
      gameResetKey: 0
    };
  },
  methods: {
    resetGame() {
      this.gameResetKey += 1;
    }
  }
}
</script>
<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
