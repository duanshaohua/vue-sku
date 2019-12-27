<template>
  <div id="app">
    <div class="sku_container">
      <sku-editer v-model="specification" />
    </div>
    <div class="sku_container">
      <sku-select :skusData="specificationFilter" />
    </div>
    <div class="sku_container">
      <sku-table
        v-model="sku"
        :skusData="specificationFilter"
        :fields="fields"
        :custom-columns="columns"
      />
    </div>
    <div class="sku_container">
      <vue-json-pretty :data="specificationFilter" />
    </div>
  </div>
</template>

<script>
import Vue from 'vue';
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
    sku: [],
    fields: {
      aa: '',
      format: '',
      guide_price: undefined,
      purchase_price: undefined,
      sell_price: undefined
    },
    columns: [
      {
        label: '测试',
        width: 150,
        component: Vue.extend({
          props: ['row'],
          render() {
            return (
              <ElInput
                placeholder="请输入测试"
                value={this.row.aa}
                // eslint-disable-next-line vue/no-side-effects-in-computed-properties
                oninput={e => (this.row.aa = e.trim())}
              />
            );
          }
        })
      },
      {
        label: '规格',
        width: 150,
        component: Vue.extend({
          props: ['row'],
          render() {
            return (
              <ElInput
                placeholder="请输入规格"
                value={this.row.format}
                // eslint-disable-next-line vue/no-side-effects-in-computed-properties
                oninput={e => (this.row.format = e.trim())}
              />
            );
          }
        })
      },
      {
        label: '厂家指导价',
        width: 150,
        component: Vue.extend({
          props: ['row'],
          render() {
            return (
              <ElInputNumber
                placeholder="请输入厂家指导价"
                value={this.row.guide_price}
                step={1}
                min={0}
                controls={false}
                precision={0}
                // eslint-disable-next-line vue/no-side-effects-in-computed-properties
                oninput={e => (this.row.guide_price = e)}
              />
            );
          }
        })
      },
      {
        label: '进价',
        width: 150,
        component: Vue.extend({
          props: ['row'],
          render() {
            return (
              <ElInputNumber
                placeholder="请输入进价"
                value={this.row.purchase_price}
                step={1}
                min={0}
                controls={false}
                precision={0}
                // eslint-disable-next-line vue/no-side-effects-in-computed-properties
                oninput={e => (this.row.purchase_price = e)}
              />
            );
          }
        })
      },
      {
        label: '标价',
        width: 150,
        component: Vue.extend({
          props: ['row'],
          render() {
            return (
              <ElInputNumber
                placeholder="请输入标价"
                value={this.row.sell_price}
                step={1}
                min={0}
                controls={false}
                precision={0}
                // eslint-disable-next-line vue/no-side-effects-in-computed-properties
                oninput={e => (this.row.sell_price = e)}
              />
            );
          }
        })
      }
    ]
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