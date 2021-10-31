<template>
  <Game/>
</template>

<script>
import Game from './components/Game.vue'

export default {
  name: 'App',
  components: {
    Game
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@700&display=swap');
*,*::before,*::after{
  box-sizing: border-box;
  padding: 0;
  margin: 0;

  font-family: 'Montserrat', sans-serif;
}

html, body {
  overflow: hidden;
}
</style>
