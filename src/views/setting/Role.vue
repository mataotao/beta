<template>
  <div>
    <div class="table-operator">
      <a-button type="primary" icon="plus" @click="handlerForm(1,0)">新建</a-button>
      <a-table
        :columns="columns"
        :dataSource="data"
        @change="handleTableChange"
        :pagination="pagination"
        :rowKey="record => record.id"
      >
        <span slot="action" slot-scope="text, record">
          <a-dropdown>
            <a class="ant-dropdown-link">
              更多 <a-icon type="down" />
            </a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a href="javascript:;" @click="handlerForm(2,record.id)">编辑</a>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定要删除角色吗？一旦删除不可恢复" okText="确定" cancelText="取消" @confirm="delRole(record.id)">
                  <a href="javascript:;">删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>
      </a-table>
    </div>
    <div>
      <a-modal
        title="设置"
        v-model="visible"
        @ok="handleOk"
        @cancel="handleCancel"
      >
        <div>
          <a-form >

            <a-form-item
              label="名称"
            >
              <a-input v-model="form.name" />
            </a-form-item>

            <a-form-item
              label="描述"
            >
              <a-input v-model="form.description" />
            </a-form-item>
          </a-form>
          <a-tree
            v-model="checkedPermission"
            checkable
            checkStrictly
            :treeData="permissions"
            defaultExpandAll
          >
          </a-tree>
        </div>
      </a-modal>
    </div>
  </div>
</template>
<script>
import { getOrgTree } from '@/api/manage'
import notification from 'ant-design-vue/es/notification'
import { axios } from '@/utils/request'

const columns = [{
  dataIndex: 'name',
  key: 'name',
  title: '角色名称'
}, {
  title: '描述',
  dataIndex: 'description',
  key: 'description'
}, {
  title: 'Action',
  key: 'action',
  scopedSlots: { customRender: 'action' }
}]

export default {
  data () {
    return {
      visible: false,
      data: [],
      columns,
      type: 1,
      checkedPermission: {
        checked: [],
        halfChecked: []
      },
      form: {
        id: 0,
        name: '',
        description: '',
        permission: []
      },
      permissions: [],
      pagination: {
        current: 1,
        total: 0
      }
    }
  },
  created () {
    this.initList()
    this.initPermissionList()
  },
  methods: {
    initList () {
      axios({
        url: '/admin/role?page=' + this.pagination.current,
        method: 'get'
      })
        .then((res) => {
          if (res.code === 0) {
            this.pagination.total = res.data.count
            this.data = res.data.list
          }
        })
        .catch((res) => {
          console.log(res)
        })
    },
    initPermissionList () {
      console.log(this.checkedPermission)
      getOrgTree().then(res => {
        this.permissions = res.data
        this.conversionTree(this.permissions)
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
    handlerForm (type, id) {
      this.visible = true
      this.form.id = id
      this.type = type
      switch (type) {
        case 2:
          this.getRole(id)
          break
      }
    },
    getRole (id) {
      axios({
        url: '/admin/role/' + id,
        method: 'get'
      })
        .then((res) => {
          if (res.code === 0) {
            this.form.id = res.data.id
            this.form.name = res.data.name
            this.form.description = res.data.description
            this.form.permission = res.data.permission_id
            this.checkedPermission = {
              checked: this.form.permission,
              halfChecked: []
            }
          }
        })
        .catch((res) => {
          console.log(res)
        })
    },
    delRole (id) {
      axios({
        url: '/admin/role/' + id,
        method: 'delete'
      })
        .then((res) => {
          if (res.code === 0) {
            notification.success({
              message: res.code,
              description: res.message
            })
            this.initList()
          }
        })
        .catch((res) => {
          console.log(res)
        })
    },
    add () {
      axios({
        url: '/admin/role',
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
      axios({
        url: '/admin/role/' + this.form.id,
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
    handleCancel () {
      this.visible = false
      this.initForm()
    },
    handleOk () {
      this.form.permission = this.checkedPermission.checked
      switch (this.type) {
        case 1:
          this.add()
          break
        case 2:
          this.edit()
          break
      }
    },
    initForm () {
      this.form = {
        name: '',
        description: '',
        permission: []
      }
      this.checkedPermission = {
        checked: [],
        halfChecked: []
      }
    },
    handleTableChange (pagination) {
      const pager = { ...this.pagination }
      pager.current = pagination.current
      this.pagination = pager
      this.initList()
    }
  }
}
</script>

<style scoped>
  .table-operator{
    background-color: #fafafa;
  }
</style>
