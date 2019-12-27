<template>
  <div class="container">
    <div class="flex">
      <el-form class="guide_coefficient" :inline="true" :model="coefficient">
        <el-form-item label="指导价倍数" prop="guide_coefficient">
          <el-input-number
            :step="0.1"
            :min="1"
            :precision="2"
            v-model="coefficient.guide_coefficient"
          />
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="setGuideCoefficient">批量设置指导价倍数</el-button>
        </el-form-item>
      </el-form>
      <el-form :inline="true" :model="coefficient">
        <el-form-item label="标价倍数" prop="purchase_coefficient">
          <el-input-number
            :step="0.1"
            :min="1"
            :precision="2"
            v-model="coefficient.purchase_coefficient"
          />
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="setPurchaseCoefficient">批量设置标价倍数</el-button>
        </el-form-item>
      </el-form>
    </div>
    <egrid
      border
      max-height="500"
      v-bind="$attrs"
      v-on="$listeners"
      :data="data"
      :columns="columns"
      :columns-props="columnsProps"
      highlight-current-row
    />
  </div>
</template>

<script>
import Vue from 'vue';
import Egrid from 'egrid';
import { flatten } from '../utils/sku';
import { diffArary } from '../utils';

export default {
  components: {
    Egrid
  },
  model: {
    prop: 'value',
    event: 'change'
  },
  props: {
    value: {
      type: Array,
      default() {
        return [];
      }
    },
    skusData: {
      type: Array,
      default() {
        return [];
      }
    }
  },

  computed: {
    skusList() {
      /**
       * output:
       * [
       *   {
       *     ids: '1-11_2-21',
       *     skus: [{"k_id":1,"k":"颜色","v_id":11,"v":"红色"},{"k_id":2,"k":"尺寸","v_id":21,"v":"大"}],
       *   },
       *   {
       *     ids: '1-11_2-22',
       *     skus: [{"k_id":1,"k":"颜色","v_id":11,"v":"红色"},{"k_id":2,"k":"尺寸","v_id":22,"v":"小"}],
       *   }
       * ]
       */
      return flatten(this.skusData).map(item => ({
        skus: item.skus,
        ids: item.skus.reduce(
          (total, prev, index) =>
            `${total}${prev.k_id}-${prev.v_id}${
              index === item.skus.length - 1 ? '' : '_'
            }`,
          ''
        )
      }));
    },

    columns() {
      const specList = this.skusData.map(item => ({
        label: item.value,
        fixed: true,
        width: 100,
        formater(row) {
          const sku = row.skus.find(sku => sku.k === item.value);
          return sku.v;
        }
      }));
      return [...specList, ...this.customColumns];
    }
  },

  watch: {
    skusList: {
      deep: true,
      immediate: true,
      handler(newSkus, oldSkus) {
        if (!newSkus.length) return (this.data = []);
        if (!oldSkus || !oldSkus.length) return this.initData(newSkus);
        if (newSkus.length === oldSkus.length) {
          // 当规格名和规格值数量未发生变化，更新 skus 即可
          return (this.data = newSkus.map((item, index) => ({
            ...this.data[index],
            ...item
          })));
        }

        // 当规格名的数量发生了变化
        if (newSkus[0].skus.length !== oldSkus[0].skus.length)
          return this.initData(newSkus);

        const diffIds = this.diffIds(newSkus, oldSkus);
        if (newSkus.length > oldSkus.length) {
          // 当添加了规格值
          let data = [];
          newSkus.forEach(item => {
            const sku = this.data.find(_item => _item.ids === item.ids);
            if (sku) {
              data.push(sku);
            } else {
              data.push({
                ...item,
                ...this.fields
              });
            }
          });
          this.data = data;
        } else {
          // 当删除了规格值
          this.data = this.data.filter(_item => !diffIds.includes(_item.ids));
        }
      }
    },
    data: {
      deep: true,
      immediate: true,
      handler(newVal) {
        this.$emit('change', newVal);
      }
    }
  },

  data: () => ({
    data: [],
    // [
    //   {
    //     ids: '1-11_2-21',
    //     skus: [{"k_id":1,"k":"颜色","v_id":11,"v":"红色"},{"k_id":2,"k":"尺寸","v_id":21,"v":"大"}],
    //     price: '',
    //     guide: '',
    //   },
    //   {
    //     ids: '1-11_2-22',
    //     skus: [{"k_id":1,"k":"颜色","v_id":11,"v":"红色"},{"k_id":2,"k":"尺寸","v_id":22,"v":"小"}],
    //     price: '',
    //     guide: '',
    //   }
    // ]

    columnsProps: {
      align: 'center',
      minWidth: 100
    },
    coefficient: {
      purchase_coefficient: 0,
      guide_coefficient: 0
    },
    fields: {
      name: '',
      code: '',
      barcode: '',
      purchasePrice: 0,
      guidePrice: 0,
      sellingPrice: 0,
      deliveryPrice: 0
    },
    customColumns: [
      {
        label: '规格名称',
        width: 230,
        component: Vue.extend({
          props: ['row'],
          render(h) {
            var self = this;
            return h('el-input', {
              attrs: {
                placeholder: '规格名称',
                value: self.row.name
              },
              on: {
                input: e => (self.row.name = e.trim())
              }
            });
          }
        })
      },
      {
        label: '规格编码',
        width: 230,
        component: Vue.extend({
          props: ['row'],
          render(h) {
            var self = this;
            return h('el-input', {
              attrs: {
                placeholder: '规格编码',
                value: self.row.code
              },
              on: {
                input: e => (self.row.code = e.trim())
              }
            });
          }
        })
      },
      {
        label: '建议进价',
        width: 230,
        component: Vue.extend({
          props: ['row'],
          render(h) {
            var self = this;
            return h('el-input-number', {
              props: {
                value: self.row.purchasePrice,
                step: 0.01,
                min: 0,
                controls: false,
                precision: 2
              },
              on: {
                input: e => (self.row.purchasePrice = e)
              }
            });
          }
        })
      },
      {
        label: '厂家指导价',
        width: 230,
        component: Vue.extend({
          props: ['row'],
          render(h) {
            var self = this;
            return h('el-input-number', {
              props: {
                value: self.row.guidePrice,
                step: 0.01,
                min: 0,
                controls: false,
                precision: 2
              },
              on: {
                input: e => (self.row.guidePrice = e)
              }
            });
          }
        })
      },
      {
        label: '建议售价',
        width: 230,
        component: Vue.extend({
          props: ['row'],
          render(h) {
            var self = this;
            return h('el-input-number', {
              props: {
                value: self.row.sellingPrice,
                step: 0.01,
                min: 0,
                controls: false,
                precision: 2
              },
              on: {
                input: e => (self.row.sellingPrice = e)
              }
            });
          }
        })
      },
      {
        label: '建议配送价',
        width: 230,
        component: Vue.extend({
          props: ['row'],
          render(h) {
            var self = this;
            return h('el-input-number', {
              props: {
                value: self.row.deliveryPrice,
                step: 0.01,
                min: 0,
                controls: false,
                precision: 2
              },
              on: {
                input: e => (self.row.deliveryPrice = e)
              }
            });
          }
        })
      }
    ]
  }),

  methods: {
    diffIds(skusList1, skusList2) {
      // 两个数据的 ids 进行相差
      skusList1 = skusList1.map(item => item.ids);
      skusList2 = skusList2.map(item => item.ids);
      return diffArary(skusList1, skusList2);
    },

    initData(skusList) {
      this.data = skusList.map(item => {
        this.fields.name = this.skuName(item.skus);
        return {
          ...item,
          // 初始化属性
          ...this.fields
        };
      });
    },

    skuName(skus) {
      return skus
        .map(item => {
          return item.v;
        })
        .join('');
    },

    setGuideCoefficient() {
      const guide_coefficient = this.coefficient.guide_coefficient;
      this.data = this.data.map(item => ({
        ...item,
        purchasePrice: (item.guidePrice || 0) * guide_coefficient
      }));
    },

    setPurchaseCoefficient() {
      const purchase_coefficient = this.coefficient.purchase_coefficient;
      this.data = this.data.map(item => ({
        ...item,
        sellingPrice: (item.purchasePrice || 0) * purchase_coefficient
      }));
    }
  }
};
</script>

<style lang="sass" scoped>
.container
  padding: 20px 0px;
.flex
  display: flex

  .guide_coefficient
    margin-right: 20px
</style>