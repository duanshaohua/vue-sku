<template>
  <div id="app">
    <div class="sku_container">
      <sku-editer v-model="specification" />
    </div>
    <div class="sku_container">
      <sku-select :skusData="specificationFilter" />
    </div>
    <div class="sku_container">
      <sku-table v-model="sku" :skusData="specificationFilter" />
    </div>
    <div class="sku_container">
      <vue-json-pretty :data="specificationFilter" />
    </div>
  </div>
</template>

<script>
import VueJsonPretty from 'vue-json-pretty';
import SkuEditer from './components/sku-editer';
import SkuSelect from './components/sku-select';
import SkuTable from './components/sku-table';

export default {
  components: {
    VueJsonPretty,
    SkuEditer,
    SkuSelect,
    SkuTable
  },

  data: () => ({
    specification: [],
    sku: []
  }),

  computed: {
    // 过滤掉属性名和属性值为空的数据规格项目
    specificationFilter() {
      return this.specification.filter(item => item.value && item.leaf.length);
    }
  }
};
</script>

<style lang="sass" scoped>
.sku_container
  margin-bottom: 30px
  font-size: 12px
  color: #666
  padding: 10px
  border: 1px solid #e5e5e5
</style>