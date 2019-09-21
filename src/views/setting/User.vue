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
        <span slot="avatar" slot-scope="text, record">
          <img v-if="record.avatar" height="100" width="100" :src="imageHost+record.avatar" alt="avatar" />
        </span>
        <span slot="status" slot-scope="text, record">
          <a-tag color="green" v-if="record.status===1">正常</a-tag>
          <a-tag color="blue" v-if="record.status===2">冻结</a-tag>
        </span>
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
                <a href="javascript:;" @click="changePwd(record.id)">修改密码</a>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定要冻结吗？" okText="确定" cancelText="取消" @confirm="freeze(record.id)" v-if="record.status==1">
                  <a href="javascript:;">冻结</a>
                </a-popconfirm>
                <a-popconfirm title="确定要解冻吗？" okText="确定" cancelText="取消" @confirm="freeze(record.id)" v-if="record.status==2">
                  <a href="javascript:;">解冻</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>
      </a-table>
    </div>
    <div>
      <a-modal
        title="用户"
        v-model="visible"
        @ok="handleOk"
        @cancel="handleCancel"
      >
        <div>
          <a-form >
            <a-form-item
              label="姓名"
            >
              <a-input v-model="form.name" />
            </a-form-item>
            <a-form-item
              label="用户名"
            >
              <a-input v-model="form.username" v-if="type===1" />
              <a-input v-model="form.username" disabled v-else />
            </a-form-item>
            <a-form-item
              label="手机号"
            >
              <a-input v-model="form.mobile" />
            </a-form-item>
            <a-form-item
              label="密码"
            >
              <a-input type="password" v-model="form.password" />
            </a-form-item>
            <a-form-item
              label="头像"
            >
              <a-upload
                name="file"
                listType="picture-card"
                class="avatar-uploader"
                :showUploadList="false"
                :action="imageHost+'upload'"
                @change="handleChange"
              >
                <img v-if="form.avatar" height="100" width="100" :src="imgUrl" alt="avatar" />
                <div v-else>
                  <a-icon :type="loading ? 'loading' : 'plus'" />
                  <div class="ant-upload-text">Upload</div>
                </div>
              </a-upload>
            </a-form-item>
            <a-form-item
              label="角色"
            >
              <a-checkbox-group :options="roleA" v-model="form.role_ids">
                <span style="color: red" slot="label" slot-scope="{ value }">{{ value }}</span>
              </a-checkbox-group>
            </a-form-item>
          </a-form>
        </div>
      </a-modal>
      <a-modal
        title="用户"
        v-model="visiblePwd"
        @ok="handlePwdOk"
        @cancel="handlePwdCancel"
      >
        <div>
          <a-form >
            <a-form-item
              label="密码"
            >
              <a-input v-model="form.password" type="password" />
            </a-form-item>
          </a-form>
        </div>
      </a-modal>
    </div>
  </div>
</template>
<script>
import notification from 'ant-design-vue/es/notification'
import { axios } from '@/utils/request'

const columns = [{
  dataIndex: 'username',
  key: 'username',
  title: '用户名'
}, {
  title: '姓名',
  dataIndex: 'name',
  key: 'name'
}, {
  title: '手机号',
  dataIndex: 'mobile',
  key: 'mobile'
}, {
  title: '头像',
  dataIndex: 'avatar',
  key: 'avatar',
  scopedSlots: { customRender: 'avatar' }
}, {
  title: '上次登录时间',
  dataIndex: 'last_time',
  key: 'last_time'
}, {
  title: '上次登录ip',
  dataIndex: 'last_ip',
  key: 'last_ip'
}, {
  title: '状态',
  dataIndex: 'status',
  key: 'status',
  scopedSlots: { customRender: 'status' }
}, {
  title: 'Action',
  key: 'action',
  scopedSlots: { customRender: 'action' }
}]

export default {
  data () {
    return {
      loading: false,
      visible: false,
      visiblePwd: false,
      data: [],
      columns,
      type: 1,
      form: {
        id: 0,
        name: '',
        username: '',
        mobile: '',
        password: '',
        avatar: '',
        role_ids: []
      },
      imgUrl: '',
      roleA: [],
      pagination: {
        current: 1,
        total: 0
      },
      imageHost: 'http://promotion.new-dhb.com/'
    }
  },
  created () {
    this.initList()
    this.initRoleList()
  },
  methods: {
    handleChange (info) {
      if (info.file.status === 'uploading') {
        this.loading = true
        return
      }
      if (info.file.status === 'done') {
        console.log(info.file.originFileObj)
        const res = info.file.response
        if (res.code === 0) {
          this.form.avatar = res.data[0]
          this.imgUrl = this.imageHost + res.data[0]
        }
        this.loading = false
      }
    },
    img (img) {
      this.imgUrl = this.imageHost + img
    },
    initList () {
      axios({
        url: '/admin/user?page=' + this.pagination.current,
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
    initRoleList () {
      axios({
        url: '/admin/role-a',
        method: 'get'
      })
        .then((res) => {
          if (res.code === 0) {
            res.data.forEach((v) => {
              this.roleA.push({ label: v.name, value: v.id })
            })

            console.log(this.roleA)
          }
        })
        .catch((res) => {
          console.log(res)
        })
    },
    handlerForm (type, id) {
      this.visible = true
      this.form.id = id
      this.type = type
      switch (type) {
        case 2:
          this.getUser(id)
          break
      }
    },
    getUser (id) {
      axios({
        url: '/admin/user/' + id,
        method: 'get'
      })
        .then((res) => {
          if (res.code === 0) {
            this.form.id = res.data.id
            this.form.name = res.data.name
            this.form.username = res.data.username
            this.form.mobile = res.data.mobile
            this.form.password = res.data.password
            this.form.avatar = res.data.avatar
            this.form.role_ids = res.data.role_ids
            this.imgUrl = this.imageHost + res.data.avatar
          }
        })
        .catch((res) => {
          console.log(res)
        })
    },
    freeze (id) {
      console.log(id)
      axios({
        url: '/admin/user-status/' + id,
        method: 'put'
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
    changePwd (id) {
      this.form.id = id
      this.visiblePwd = true
    },
    add () {
      axios({
        url: '/admin/user',
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
        url: '/admin/user/' + this.form.id,
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
      this.form.mobile = Number(this.form.mobile)
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
        id: 0,
        name: '',
        username: '',
        mobile: '',
        password: '',
        avatar: '',
        role_ids: []
      }
    },
    handleTableChange (pagination) {
      const pager = { ...this.pagination }
      pager.current = pagination.current
      this.pagination = pager
      this.initList()
    },
    handlePwdOk () {
      axios({
        url: '/admin/user-pwd/' + this.form.id,
        method: 'put',
        data: this.form
      })
        .then((res) => {
          if (res.code === 0) {
            notification.success({
              message: res.code,
              description: res.message
            })
            this.handlePwdCancel()
            this.initList()
          }
        })
        .catch((res) => {
          console.log(res)
        })
    },
    handlePwdCancel () {
      this.visiblePwd = false
      this.initForm()
    }
  }
}
</script>

<style scoped>
  .table-operator{
    background-color: #fafafa;
  }
</style>
