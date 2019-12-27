<template>
  <div class="sku-editer">
    <div class="sku_group mb10" v-for="(spec, index) in specification" :key="spec.id">
      <div class="spec_title">
        <span class="label">规格名：</span>
        <el-input class="input" placeholder="请输入规格名" v-model.trim="spec.value" />
        <el-button
          @click="delSepc(index)"
          type="danger"
          size="mini"
          icon="el-icon-delete"
          plain
          circle
        ></el-button>
      </div>
      <div class="group_container">
        <span class="label">规格值：</span>
        <el-popover
          placement="bottom"
          width="120"
          trigger="click"
          v-for="(option, option_index) in spec.leaf"
          :key="option_index"
        >
          <el-input v-model.trim="option.value" style="width:100%;" />
          <div class="sku_item" slot="reference">
            <span class="remove el-icon-delete" @click.stop="delOption(index, option_index)"></span>
            <div class="text">{{option.value}}</div>
          </div>
        </el-popover>
        <el-input
          class="input"
          suffix-icon="el-icon-plus"
          v-model="addValues[index]"
          placeholder="多个值以空格隔开"
          @keyup.native.enter="addOption(index)"
          @blur="addOption(index)"
        />
      </div>
    </div>
    <div class="spec_title">
      <el-button type="primary" :disabled="disabled" @click="addSpec" plain>添加规格项目</el-button>
    </div>
  </div>
</template>

<script>
import { createUniqueString, uniqueArr } from '../utils';
export default {
  model: {
    prop: 'value',
    event: 'change'
  },
  props: {
    value: {
      type: Array,
      default: () => {
        return [
          {
            id: 1,
            value: '颜色',
            leaf: [
              {
                id: 11,
                value: '白色'
              },
              {
                id: 12,
                value: '黑色'
              },
              {
                id: 13,
                value: '金色'
              }
            ]
          },
          {
            id: 2,
            value: '内存',
            leaf: [
              {
                id: 21,
                value: '128G'
              },
              {
                id: 22,
                value: '256G'
              },
              {
                id: 23,
                value: '512G'
              }
            ]
          }
        ];
      }
    }
  },
  data() {
    return {
      // 用来存储要添加的规格属性
      addValues: [],
      specification: [
        {
          id: 1,
          value: '颜色',
          leaf: [
            {
              id: 11,
              value: '白色'
            },
            {
              id: 12,
              value: '黑色'
            },
            {
              id: 13,
              value: '金色'
            }
          ]
        },
        {
          id: 2,
          value: '内存',
          leaf: [
            {
              id: 21,
              value: '128G'
            },
            {
              id: 22,
              value: '256G'
            },
            {
              id: 23,
              value: '512G'
            }
          ]
        }
      ]
    };
  },
  computed: {
    disabled() {
      return (
        this.specification.length > 3 ||
        this.specification.some(item => !item.value)
      );
    }
  },
  watch: {
    value: {
      handler: function(val) {
        this.specification = val;
      },
      deep: true
    },
    specification: {
      handler: function(newVal) {
        this.$emit('change', newVal);
      },
      deep: true
    }
  },
  mouted() {
    this.specification = this.value;
  },
  methods: {
    addSpec() {
      this.specification.push({
        id: createUniqueString() + '_id',
        value: '',
        leaf: []
      });
    },

    delSepc(index) {
      this.specification.splice(index, 1);
    },

    addOption(index) {
      let str = this.addValues[index] || '';
      str = str.trim();
      if (!str) return;
      const oldArr = this.specification[index].leaf;
      const arr = str
        .split(/\s+/) // 使用空格分割成数组
        .filter(value => !oldArr.some(option => option.value === value)) // 过滤掉 oldArr 已存在的 value
        .map(value => ({ id: createUniqueString() + '_id', value })); // 把 value 转成对象，id 设置为 null
      this.specification[index].leaf = uniqueArr([...oldArr, ...arr]);
      this.$set(this.addValues, index, '');
    },

    delOption(spec_index, option_index) {
      this.specification[spec_index].leaf.splice(option_index, 1);
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

.remove
  display: none
  position: absolute
  z-index: 2
  width: 18px
  height: 18px
  font-size: 12px
  line-height: 18px
  color: #fff
  text-align: center
  cursor: pointer
  background: rgba(0,0,0,.3)
  border-radius: 50%

.sku_group
  margin-bottom: 10px
  &:hover
    .spec_title .remove
      display: block

.spec_title
  position: relative
  padding: 7px 10px
  background-color: #f8f8f8
  line-height: 16px
  font-weight: 400

  .input
    width: 200px

  .el-button
      margin-left: 15px

.group_container
  padding: 10px 10px 0

  .input
    width: 250px

  .sku_item
    background-color: #f8f8f8
    padding: 10px
    display: inline-block
    margin-right: 10px
    vertical-align: middle
    text-align: center
    position: relative
    border-radius: 2px
    cursor: pointer

    &:hover
      .remove
        display: block

    .remove
      top: -8px
      right: -8px

    .text
      display: block
      width: 74px
      margin: 0 auto
      overflow: hidden
      text-overflow: ellipsis
      white-space: nowrap
</style>