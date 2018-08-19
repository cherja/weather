<template>
  <span>
    <h1>{{ obj.temps | minTemp }}</h1>
    <h1>{{ obj.temps | maxTemp }}</h1>
  </span>
</template>

<script>
export default {
  props: {
    obj: {
      type: Object,
      required: true
    }
  },
  filters: {
    maxTemp (value) {
      return Math.round(Math.max(...value))
    },
    minTemp (value) {
      return Math.round(Math.min(...value))
    }
  }
}
</script>

<style lang="scss">
.icon {
  width: 200px;
  height: 200px;
}

</style>
