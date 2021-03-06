<template>
<table class="veui-table" :ui="ui">
  <colgroup>
    <col v-if="selectable" width="60">
    <col v-for="col in displayedColumns" :width="col.width">
  </colgroup>
  <table-head :columns="displayedColumns" :selectable="selectable" :select-status="selectStatus" @select="select" @sort="sort"></table-head>
  <slot name="foot"><table-foot v-if="hasFoot" :data="data" :columns="displayedColumns"></table-foot></slot>
  <tbody v-if="!data.length">
    <tr><td class="veui-table-no-data" :colspan="displayedColumns.length"><slot name="no-data">没有数据</slot></td></tr>
  </tbody>
  <template v-else>
    <table-body :data="data" :columns="displayedColumns" :selectable="selectable"
      :selected-items="selectedItems" :keys="realKeys" @select="select"></table-body>
  </template>
  <slot></slot>
</table>
</template>

<script>
import { map, zipObject, intersection, isString, isArray, includes } from 'lodash'
import Body from './Body'
import Head from './Head'
import Foot from './Foot'

export default {
  name: 'veui-table',
  uiTypes: ['table'],
  components: {
    'table-body': Body,
    'table-head': Head,
    'table-foot': Foot
  },
  props: {
    ui: String,
    data: {
      type: Array,
      default () {
        return []
      }
    },
    keys: {
      validator (value) {
        if (!value) {
          return true
        }
        return isString(value) || isArray(value) && value.length === this.data.length
      }
    },
    selectable: Boolean,
    order: [String, Boolean],
    orderBy: String,
    columnFilter: Array
  },
  data () {
    return {
      columns: [],
      selectedItems: {}
    }
  },
  computed: {
    displayedColumns () {
      if (!this.columnFilter) {
        return this.columns
      }
      return this.columns.filter(col => includes(this.columnFilter, col.field))
    },
    realKeys () {
      let keys = this.keys
      if (!keys) {
        keys = Object.keys(this.data)
      }
      if (typeof keys === 'string') {
        keys = map(this.data, keys)
      }
      return keys.map(String)
    },
    selectStatus () {
      let keys = this.realKeys
      let selectedKeys = Object.keys(this.selectedItems)
      let inter = intersection(keys, selectedKeys)
      if (!inter.length) {
        return 'none'
      }
      if (inter.length === keys.length) {
        return 'all'
      }
      return 'partial'
    },
    hasFoot () {
      return this.$slots.foot || this.columns.some(col => col.hasFoot)
    }
  },
  methods: {
    select (selected, index) {
      if (index !== undefined) {
        let item = this.data[index]
        index = this.realKeys ? this.realKeys[index] : index
        if (selected) {
          this.$set(this.selectedItems, index, item)
        } else {
          this.$delete(this.selectedItems, index)
        }
        this.$emit('select', selected, item, this.selectedItems)
      } else {
        if (selected) {
          let items = zipObject(this.realKeys, this.data)
          this.selectedItems = {
            ...this.selectedItems,
            ...items
          }
        } else {
          this.selectedItems = {}
        }
        this.$emit('select', selected, this.selectedItems)
      }
    },
    sort (field, order) {
      this.$emit('sort', field, order)
    }
  }
}
</script>

<style lang="less">
@import "../../styles/theme-default/lib.less";

.veui-table {
  table-layout: fixed;
  width: 100%;
  border-collapse: collapse;

  th,
  td {
    height: 54px;
    padding: 20px;
    color: @veui-gray-color-normal;
    text-align: left;
    white-space: nowrap;
  }

  th {
    font-weight: @veui-font-weight-bold;
  }

  td {
    border: 1px solid @veui-gray-color-sup-2;
    border-style: solid none;
  }

  tfoot th {
    border-top: 1px solid @veui-gray-color-sup-2;
  }

  &[ui~="slim"] {
    th,
    td {
      height: 48px;
      .padding(17px, _);
    }
  }

  &[ui~="alt"] {
    thead th {
      color: @veui-text-color-weak;
    }

    thead,
    tfoot {
      background-color: @veui-gray-color-sup-4;
    }
  }

  .veui-table-no-data {
    text-align: center;
  }

  .veui-button + .veui-button {
    margin-left: 30px;
  }

  .veui-table-header {
    display: inline-block;
    vertical-align: middle;
  }

  .veui-table-sorter {
    margin-left: 5px;
    vertical-align: middle;
  }
}
</style>
