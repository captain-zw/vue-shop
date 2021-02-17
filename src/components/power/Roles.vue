<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <!-- 添加角色按钮 -->
      <el-row>
        <el-col>
          <el-button type='primary'>添加角色</el-button>
        </el-col>
      </el-row>
      <el-table :data='roleList' border stripe>
        <!-- 展开列 -->
        <el-table-column type='expand'>
          <template slot-scope='scope'>
            <el-row v-for='(item1,i1) in scope.row.children' :key='item1.id'
              :class="['bdbottom',i1===0?'bdtop':'','vcenter']">
              <!-- 渲染一级权限 -->
              <el-col :span='5'>
                <el-tag closable @close="removeRightById(scope.row,item1.id)">
                  {{item1.authName}}
                </el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级和三级权限 -->
              <el-col :span='19'>
                <!-- 通过 For 循环渲染二级权限 -->
                <el-row :class="[i2===0?'':'bdtop','vcenter']" v-for='(item2,i2) in item1.children' :key='item2.id'>
                  <el-col :span='6'>
                    <el-tag type='success' closable @close="removeRightById(scope.row,item2.id)">
                      {{item2.authName}}
                    </el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span='18'>
                    <!-- 通过 For 循环渲染三级级权限 -->
                    <el-tag type='warning' v-for='(item3) in item2.children' closable :key='item3.id'
                      @close="removeRightById(scope.row,item3.id)">
                      {{item3.authName}}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop='roleName'></el-table-column>
        <el-table-column label="角色描述" prop='roleDesc'></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
            <el-button type="warning" icon="el-icon-setting" size="mini" @click='showSetRightDialog(scope.row)'>分配权限
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 分配权限对话框 -->
    <el-dialog title="分配权限" :visible.sync="setRightDialogVisable" width="50%" @close="setRightDialogClosed">
      <el-tree :data="rightList" :props="treeProps" show-checkbox node-key="id" default-expand-all
        :default-checked-keys='defKeys' ref='treeRef'></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisable = false">取 消</el-button>
        <el-button type="primary" @click="alotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        roleList: [],
        setRightDialogVisable: false,
        // 所有权限的列表
        rightList: [],
        // 树形控件的属性绑定对象
        treeProps: {
          label: 'authName',
          children: 'children'
        },
        // 默认选中的tree节点
        defKeys: [],
        // 当前即将分配角色权限的id
        roleId: ''
      }
    },
    created() {
      this.getRolesList()
    },
    methods: {
      // 获取角色的列表
      async getRolesList() {
        const {
          data: res
        } = await this.$http.get('roles')
        // console.log(res)
        if (res.meta.status !== 200) {
          return this.$message.error('获取角色列表失败')
        }
        this.roleList = res.data
        // console.log(this.roleList)
      },
      // 根据id删除对应的权限
      async removeRightById(role, rightId) {
        const confirm = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
        if (confirm !== 'confirm') return this.$message.info('取消了删除')
        // console.log(id)
        const {
          data: res
        } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
        if (res.meta.status !== 200) return this.$message.error('权限删除失败！')
        this.$message.success('删除成功')
        // this.getRolesList()
        role.children = res.data
      },
      // 展示分配权限的对话框
      async showSetRightDialog(role) {
        const {
          data: res
        } = await this.$http.get('rights/tree')
        // console.log(res)
        if (res.meta.status !== 200) return this.$message.error('权限获取失败')
        this.rightList = res.data
        this.roleId = role.id
        // 递归获取三级节点的信息
        this.getLeafKeys(role, this.defKeys)

        this.setRightDialogVisable = true
      },
      // 通过递归方式获取所有三级权限id并保留到数组中
      getLeafKeys(node, arr) {
        // 如果当前节点不存在children属性，则为三级权限
        if (!node.children) {
          return arr.push(node.id)
        }
        node.children.forEach(element => {
          this.getLeafKeys(element, arr)
        })
      },
      // 今天权限对话框的关闭事件 清空权限id列表
      setRightDialogClosed() {
        this.defKeys = []
      },
      // 点击获取分配的权限数据
      async alotRights() {
        const keys = [...this.$refs.treeRef.getCheckedKeys(), ...this.$refs.treeRef.getHalfCheckedKeys()]
        const idStr = keys.join(',')
        // console.log(keys)
        const {
          data: res
        } = await this.$http.post(`roles/${this.roleId}/rights`, {
          rids: idStr
        })
        // console.log(res)
        if (res.meta.status !== 200) return this.$message.error('权限更新失败！')
        this.setRightDialogVisable = false
        this.$message.success('权限更新成功！')
        this.getRolesList()
      }
    }
  }

</script>

<style lang="less" scoped>
  .el-tag {
    margin: 7px;
  }

  .bdtop {
    border-top: 1px solid #eee;
  }

  .bdbottom {
    border-bottom: 1px solid #eee;
  }

  .vcenter {
    display: flex;
    align-items: center;
  }

</style>
