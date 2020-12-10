<template>
  <first-call />

</template>

<script>
import HelloWorld from '@/components/HelloWorld.vue'
// import FirstCall from '@/components/FirstCall.vue'
import FirstCall from '../components/FirstCall.vue'

export default {
  name: 'App',
  components: { HelloWorld, FirstCall }
}
</script>

<style>
html {
  width: 400px;
  height: 400px;
}
</style>
