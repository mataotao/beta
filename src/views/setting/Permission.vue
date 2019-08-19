<template>
  <a-card :bordered="false">
    <a-row :gutter="8">
      <a-col :offset="4" :span="8">
        <a-tree
          :treeData="treeData"
          defaultExpandAll
          @rightClick="setting"
        >
        </a-tree>
      </a-col>
    </a-row>

  </a-card>
</template>
<script>
import { getOrgTree } from '@/api/manage'

export default {
  data () {
    return {
      treeData: [],
      parentData: {
        id: 0,
        label: '菜单',
        pid: 0,
        level: 0,
        url: '',
        sort: 0,
        component: '',
        icon: 0,
        children: []
      },
      all: true
    }
  },
  created () {
    getOrgTree().then(res => {
      this.parentData.children = res.data
      const menu = [this.parentData]
      this.treeData = menu
      this.conversionTree(menu)
    })
  },
  methods: {
    conversionTree (data) {
      data.forEach((r) => {
        r.key = r.id
        r.title = r.label
        r.icon = null
        if (r.children.length > 0) {
          r.group = true
          this.conversionTree(r.children)
        } else {
          delete r.children
        }
      })
    },
    onSelect (selectedKeys, info) {
      console.log('selected', selectedKeys, info)
    },
    onCheck (checkedKeys, info) {
      console.log('onCheck', checkedKeys, info)
    },
    setting ({ event, node }) {
      console.log(node.$options.propsData.dataRef)
    }
  }
}
</script>
