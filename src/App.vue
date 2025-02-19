<template>
  <div class="wrapper">
    <OrgChartWrapper :orgData="orgData" />
  </div>
</template>

<script>
import OrgChartWrapper from './components/OrgChartWrapper.vue'
import orgData from './components/orgData'

export default {
  name: 'App',
  components: {
    OrgChartWrapper
  },
  data() {
    return {
      orgData
    }
  }
}
</script>

<style lang="scss" scoped>
.wrapper {
  box-sizing: border-box;
  width: 100%;
  height: 100vh;
}
</style>
