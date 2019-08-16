<template>
  <a-card :bordered="false">
    <a-row :gutter="8">
      <a-col :offset="4" :span="8">
        <s-tree
          :dataSource="orgTree"
          @click="handleClick"
          @add="handleAdd"
          @titleClick="handleTitleClick">

        </s-tree>
      </a-col>
    </a-row>

    <org-modal ref="modal" @ok="handleSaveOk" @close="handleSaveClose" />
  </a-card>
</template>

<script>
import STree from '@/components/Tree/Tree'
import { STable } from '@/components'
import OrgModal from './modules/OrgModal'
import { getOrgTree } from '@/api/manage'

export default {
  components: {
    STable,
    STree,
    OrgModal
  },
  data () {
    return {
      orgTree: []
    }
  },
  created () {
    getOrgTree().then(res => {
      this.orgTree = res.data
      this.conversionTree(res.data)
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
    handleClick (e) {
      console.log('handleClick', e)
      this.queryParam = {
        key: e.key
      }
    },
    handleAdd (item) {
      console.log('add button, item', item)
      this.$refs.modal.add(item.title)
    },
    handleTitleClick (item) {
      console.log('handleTitleClick', item)
    },
    titleClick (e) {
      console.log('titleClick', e)
    },
    handleSaveOk () {

    },
    handleSaveClose () {

    },

    onSelectChange (selectedRowKeys, selectedRows) {
      this.selectedRowKeys = selectedRowKeys
      this.selectedRows = selectedRows
    }
  }
}
</script>

<style lang="less">
  .custom-tree {

    /deep/ .ant-menu-item-group-title {
      position: relative;
      &:hover {
        .btn {
          display: block;
        }
      }
    }

    /deep/ .ant-menu-item {
      &:hover {
        .btn {
          display: block;
        }
      }
    }

    /deep/ .btn {
      display: none;
      position: absolute;
      top: 0;
      right: 10px;
      width: 20px;
      height: 40px;
      line-height: 40px;
      z-index: 1050;

      &:hover {
        transform: scale(1.2);
        transition: 0.5s all;
      }
    }
  }
</style>
