<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 简单卡片 -->
    <el-card>
      <!-- 搜索添加区域 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容" clearable @clear="getUserList" v-model='queryInfo.query'>
            <el-button slot="append" icon="el-icon-search" @click='getUserList'></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click='addDialogVisible=true'>添加用户</el-button>
        </el-col>
      </el-row>
      <!-- 列表区域 -->
      <el-table :data='userlist' border stripe>
        <el-table-column type='index'></el-table-column>
        <el-table-column label="姓名" prop="username">
        </el-table-column>
        <el-table-column label="邮箱" prop="email">
        </el-table-column>
        <el-table-column label="电话" prop="mobile">
        </el-table-column>
        <el-table-column label="角色" prop="role_name">
        </el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">
            <el-switch v-model="scope.row.mg_state" @change="userStateChange(scope.row)">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180px">
          <template slot-scope="scope">
            <!-- 修改按钮 -->
            <el-button type="primary" icon="el-icon-edit" circle size="mini" @click="showDialogEdit(scope.row.id)">
            </el-button>
            <!-- 删除按钮 -->
            <el-button type="danger" icon="el-icon-delete" @click="deleteUser(scope.row.id)" circle size="mini">
            </el-button>
            <!-- 分配角色按钮 -->
            <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
              <el-button type="info" icon="el-icon-setting" circle size="mini" @click='setRole(scope.row)'></el-button>
            </el-tooltip>

          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区域 -->
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum" :page-sizes="[1, 2, 5, 10]" :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>

      <!-- 用户添加对话框 -->
      <el-dialog title="添加用户" :visible.sync="addDialogVisible" @close="addDialogClosed" width="50%">
        <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
          <el-form-item label="用户名" prop="username">
            <el-input v-model="addForm.username"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input v-model="addForm.password"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="addForm.email"></el-input>
          </el-form-item>
          <el-form-item label="电话" prop="mobile">
            <el-input v-model="addForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addUser">确 定</el-button>
        </span>
      </el-dialog>

      <!-- 修改用户对话框 -->
      <el-dialog title="用户修改" :visible.sync="editDialogVisible" width="50%" @close="edieDialogClosed">
        <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
          <el-form-item label="用户名">
            <el-input v-model="editForm.username" disabled></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop='email'>
            <el-input v-model="editForm.email"></el-input>
          </el-form-item>
          <el-form-item label="电话" prop='mobile'>
            <el-input v-model="editForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editUserInfo">确 定</el-button>
        </span>
      </el-dialog>

      <!-- 分配角色对话框 -->
      <el-dialog title="角色分配" :visible.sync="setRoleDialogVisible" width="50%" @close="removeRoleList">
        <div>
          <p>当前用户：{{userInfo.username}}</p>
          <p>当前角色：{{userInfo.role_name}}</p>
          <p>分配新角色:
            <el-select v-model="selectedRoleId" placeholder="请选择">
              <el-option v-for="item in roleList" :key="item.id" :label="item.roleName" :value="item.id">
              </el-option>
            </el-select>
          </p>
        </div>
        <span slot="footer" class="dialog-footer">
          <el-button @click="setRoleDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
  export default {
    data() {
      var checkEmail = (rule, value, cb) => {
        const regEmail = /^[a-zA-Z0-9_-]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$/
        if (regEmail.test(value)) return cb()
        cb(new Error('请输入正确的邮箱'))
      }
      var checkMobile = (rule, value, cb) => {
        const regMobile =
          /^[1](([3][0-9])|([4][0,1,4-9])|([5][0-3,5-9])|([6][2,5,6,7])|([7][0-8])|([8][0-9])|([9][0-3,5-9]))[0-9]{8}$/
        if (regMobile.test(value)) return cb()
        cb(new Error('请输入正确的手机号'))
      }
      return {
        // 获取用户列表的参数
        queryInfo: {
          query: '',
          pagenum: 1,
          pagesize: 2
        },
        userlist: [],
        total: 0,
        addDialogVisible: false,
        editDialogVisible: false,
        // 添加表单数据
        addForm: {
          username: '',
          password: '',
          email: '',
          mibile: ''
        },
        // 添加表单验证规则对象
        addFormRules: {
          username: [{
              required: true,
              message: '请输入用户名',
              trigger: 'blur'
            },
            {
              min: 3,
              max: 10,
              msssage: '用户名在3~10字符之间',
              trigger: 'blur'
            }
          ],
          password: [{
              required: true,
              message: '请输入密码',
              trigger: 'blur'
            },
            {
              min: 3,
              max: 10,
              msssage: '密码在6~15字符之间',
              trigger: 'blur'
            }
          ],
          email: [{
            required: true,
            message: '请输入邮箱',
            trigger: 'blur'
          }, {
            validator: checkEmail,
            trigger: 'blur'
          }],
          mobile: [{
            required: true,
            message: '请输入手机号',
            trigger: 'blur'
          }, {
            validator: checkMobile,
            trigger: 'blur'
          }]
        },
        editForm: {},
        editFormRules: {
          email: [{
            required: true,
            message: '请输入邮箱',
            trigger: 'blur'
          }, {
            validator: checkEmail,
            trigger: 'blur'
          }],
          mobile: [{
            required: true,
            message: '请输入手机号',
            trigger: 'blur'
          }, {
            validator: checkMobile,
            trigger: 'blur'
          }]
        },
        // 控制分配角色对话框显示
        setRoleDialogVisible: false,
        // 需要被分配角色的用户信息
        userInfo: {},
        // 所有角色列表
        roleList: [],
        // 已选定的角色
        selectedRoleId: ''
      }
    },
    created() {
      this.getUserList()
    },
    methods: {
      async getUserList() {
        const {
          data: res
        } = await this.$http.get('users', {
          params: this.queryInfo
        })
        if (res.meta.status !== 200) return this.$message.error('获取用户列表失败')
        this.userlist = res.data.users
        this.total = res.data.total
      },
      //   获取 pagesize 事件
      handleSizeChange(pagesize) {
        this.queryInfo.pagesize = pagesize
        this.getUserList()
      },
      // 获取 当前页码
      handleCurrentChange(newpage) {
        // console.log(newpage)
        this.queryInfo.pagenum = newpage
        this.getUserList()
      },
      async userStateChange(userInfo) {
        // console.log(userInfo)
        const {
          data: res
        } = await this.$http.put(`users/${userInfo.id}/state/${userInfo.mg_state}`)
        if (res.meta.status !== 201) {
          userInfo.mg_state = !userInfo.mg_state
          return this.$message.error('更新用户数据失败')
        }
        this.$message.success('更新用户状态成功')
      },
      addDialogClosed() {
        this.$refs.addFormRef.resetFields()
      },
      addUser() {
        this.$refs.addFormRef.validate(async valid => {
          if (!valid) return
          const {
            data: res
          } = await this.$http.post('users', this.addForm)
          //   console.log(res)
          if (res.meta.status !== 201) {
            this.addDialogVisible = false
            return this.$message.error('用户创建失败')
          }
          this.addDialogVisible = false
          this.$message.success('用户创建成功')
          this.getUserList()
        })
      },
      async showDialogEdit(id) {
        const {
          data: res
        } = await this.$http.get(`users/${id}`)
        // console.log(res)
        if (res.meta.status !== 200) {
          return this.$message.error('查询用户数据失败')
        }
        this.editForm = res.data
        this.editDialogVisible = true
      },
      edieDialogClosed() {
        this.$refs.editFormRef.resetFields()
      },
      editUserInfo(id) {
        this.$refs.editFormRef.validate(async valid => {
          // console.log(valid)
          if (!valid) return
          // 通过发起请求修改
          const {
            data: res
          } = await this.$http.put(`users/${this.editForm.id}`, {
            email: this.editForm.email,
            mobile: this.editForm.mobile
          })
          if (res.meta.status !== 200) return this.$message.error('用户修改失败！')
          this.$message.success('用户修改成功！')
          this.editDialogVisible = false
          this.getUserList()
          // console.log(res)
        })
      },
      async deleteUser(id) {
        const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
        if (confirmResult !== 'confirm') {
          return this.$message('已经取消删除')
        }
        // console.log(confirmResult)
        const {
          data: res
        } = await this.$http.delete('users/' + id)
        if (res.meta.status !== '200') {
          return this.$message.error('删除用户失败')
        }
        this.$message.success('用户已经删除')
        this.getUserList()
      },
      // 展示分配角色的对话框
      async setRole(userInfo) {
        this.userInfo = userInfo
        const {
          data: res
        } = await this.$http.get('roles')
        if (res.meta.status !== 200) return this.$message.error('角色获取失败')
        this.roleList = res.data
        // console.log(this.roleList)
        this.setRoleDialogVisible = true
      },
      // 点击确定 分配角色
      async saveRoleInfo() {
        if (!this.selectedRoleId) {
          return this.$message.error('请选择分配的角色')
        }
        const {
          data: res
        } = await this.$http.put(`users/${this.userInfo.id}/role`, {
          rid: this.selectedRoleId
        })
        if (res.meta.status !== 200) return this.$message.error('角色分配失败')
        this.setRoleDialogVisible = false
        this.$message.success('角色分配成功！')
        this.getUserList()
      },
      removeRoleList() {
        this.selectedRoleId = ''
        this.userInfo = {}
      }
    }
  }

</script>

<style lang="less" scoped>

</style>
