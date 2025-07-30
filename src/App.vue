<template>
 <!-- <TaktTime/> -->
  <div class="min-h-screen flex items-center justify-center bg-gradient-to-br from-blue-500 to-purple-600">
    <div class="p-8 rounded-xl shadow-xl text-center">
      <h1 class="text-2xl font-bold text-gray-800">Tailwind funcionando 🎉</h1>
      <p class="mt-4 text-green-600">Se você está vendo este layout bonito, é porque o Tailwind foi instalado com sucesso.</p>
    </div>
  </div>
</template>


<script>
import TaktTime from "./components/TaktTime.vue"

export default {
  name: 'App',
  components: {
    TaktTime
  }
}
</script>

<style>

</style>
