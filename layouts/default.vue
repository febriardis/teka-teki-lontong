<template>
  <div class="l-default">
    <header>
      <router-link to="/">TEKA TEKI SULIT</router-link>
    </header>
    <nuxt />
  </div>
</template>

<script>
export default {}
</script>

<style lang="scss">
.l-default {
  @apply relative w-full md:w-[640px] mx-auto;
}
header {
  @apply bg-[#213760] text-white p-4 w-full text-center font-bold shadow-lg;
}
</style>
