<!--
 * @Description: Description
 * @version: 1.0.0
 * @Author: FJW
 * @Date: 2024-01-11 13:16:36
 * @LastEditors: FJW
 * @LastEditTime: 2024-01-11 13:23:17
-->
<template>
  <div id="app">
    <table-list></table-list>
  </div>
</template>

<script>
import tableList from './components/tableList.vue'

export default {
  name: 'App',
  components: {
    tableList
  }
}

</script>