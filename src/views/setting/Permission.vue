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
    <div>
      <a-modal
        title="设置"
        v-model="visible"
        @ok="handleOk"
        @cancel="handleCancel"
      >
        <a-radio-group @change="handleSetChange" v-model="setConfig" buttonStyle="solid" style="width: 100%">
          <a-row >
            <a-col :span="8">
              <a-radio-button :value="1" style="width: 100%">新增</a-radio-button>
            </a-col>
            <a-col :span="8">
              <a-radio-button :value="2" style="width: 100%">编辑</a-radio-button>
            </a-col>
            <a-col :span="8">
              <a-radio-button :value="3" style="width: 100%">删除</a-radio-button>
            </a-col>
          </a-row>
        </a-radio-group>
        <div>
          <a-form v-if="setConfig === 1 || setConfig === 2">

            <a-form-item
              label="名称"
            >
              <a-input v-model="form.label" />
            </a-form-item>

            <a-form-item
              label="URL"
            >
              <a-input v-model="form.url" />
            </a-form-item>

            <a-form-item
              label="排序"
            >
              <a-input v-model="form.sort" />
            </a-form-item>

            <a-form-item
              label="组件"
            >
              <a-input v-model="form.component" />
            </a-form-item>
          </a-form>
        </div>
      </a-modal>
    </div>
  </a-card>

</template>
<script>
import { getOrgTree } from '@/api/manage'
import { axios } from '@/utils/request'
import notification from 'ant-design-vue/es/notification'

export default {
  data () {
    return {
      visible: false,
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
      all: true,
      setConfig: 1,
      form: {
        id: 0,
        label: '',
        pid: 0,
        url: '',
        level: 0,
        sort: 500,
        component: '',
        cond: ''
      },
      cForm: {}
    }
  },
  created () {
    this.initList()
  },
  methods: {
    initList () {
      getOrgTree().then(res => {
        this.parentData.children = res.data
        const menu = [this.parentData]
        this.treeData = menu
        this.conversionTree(menu)
      })
    },
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
    setting ({ event, node }) {
      this.visible = true
      this.cForm = node.$options.propsData.dataRef
    },
    handleOk (e) {
      switch (this.setConfig) {
        case 1:
          this.add()
          break
        case 2:
          this.edit()
          break
        case 3:
          this.delete()
          break
      }
    },
    add () {
      this.form.pid = this.cForm.id
      this.form.level = this.cForm.level + 1
      this.form.sort = Number(this.form.sort)
      axios({
        url: '/admin/permission',
        method: 'post',
        data: this.form
      })
        .then((res) => {
          if (res.code === 0) {
            notification.success({
              message: res.code,
              description: res.message
            })
            this.handleCancel()
            this.initList()
          }
        })
        .catch((res) => {
          console.log(res)
        })
    },
    edit () {
      this.form.sort = Number(this.form.sort)
      axios({
        url: '/admin/permission/' + this.cForm.id,
        method: 'put',
        data: this.form
      })
        .then((res) => {
          if (res.code === 0) {
            notification.success({
              message: res.code,
              description: res.message
            })
            this.handleCancel()
            this.initList()
          }
        })
        .catch((res) => {
          console.log(res)
        })
    },
    delete () {
      axios({
        url: '/admin/permission/' + this.cForm.id,
        method: 'delete'
      })
        .then((res) => {
          if (res.code === 0) {
            notification.success({
              message: res.code,
              description: res.message
            })
            this.handleCancel()
            this.initList()
          }
        })
        .catch((res) => {
          console.log(res)
        })
    },
    handleCancel (e) {
      this.visible = false
      this.initForm()
      this.setConfig = 1
    },
    handleSetChange (e) {
      switch (this.setConfig) {
        case 1:
          this.initForm()
          break
        case 2:
          this.initEdit()
          break
      }
    },
    initForm () {
      this.form = {
        id: 0,
        label: '',
        pid: 0,
        url: '',
        level: 0,
        sort: 500,
        component: '',
        cond: ''
      }
    },
    initEdit () {
      this.form = {
        id: this.cForm.id,
        label: this.cForm.label,
        pid: this.cForm.pid,
        url: this.cForm.url,
        level: this.cForm.level,
        sort: 500,
        component: this.cForm.component,
        cond: this.cForm.cond
      }
    }
  }
}
</script>
