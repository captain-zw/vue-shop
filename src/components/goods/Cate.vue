<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row>
        <el-col>
          <el-button type='primary' @click='showAddCateDialog'>添加分类</el-button>
        </el-col>
      </el-row>
      <!-- 表格区域 -->
      <tree-table class="treeTable" :data='cateList' :columns='columns' :selection-type='false' :expand-type='false'
        show-index index-text='#' border>
        <!-- 是否有效 -->
        <template slot='isok' slot-scope='scope'>
          <i v-if='scope.row.cat_deleted === false' class="el-icon-success" style='color:lightgreen'></i>
          <i class="el-icon-error" v-else></i>
        </template>
        <!-- 排序 -->
        <template slot='order' slot-scope='scope'>
          <el-button size='mini' type='primary' v-if='scope.row.cat_level===0'>一级</el-button>
          <el-button size='mini' type='warning' v-if='scope.row.cat_level===1'>二级</el-button>
          <el-button size='mini' type='danger' v-if='scope.row.cat_level===2'>三级</el-button>
        </template>
        <!-- 操作 -->
        <template slot='opt' slot-scope=''>
          <el-button type="primary" icon="el-icon-edit" size='mini'>编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size='mini'>删除</el-button>
        </template>
      </tree-table>
      <!-- 分页区域 -->
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum" :page-sizes="[2, 5, 10, 15]" :page-size="queryInfo.pagesize"
        layout=" total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </el-card>
    <!-- 添加分类的对话框 -->
    <el-dialog title="添加分类" :visible.sync="addCateDialogVisible" width="50%" @close="clearAddCate">
      <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
        <el-form-item label="分类名称:" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类:">
          <!-- option用来指定数据源 -->
          <!-- props用来指定配置对象 -->
          <el-cascader v-model="selectedKeys" clearable :options="parentCateList" expandTrigger='hover'
            :props="cascaderProps" @change="parentCateChange">
          </el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        // 查询条件
        queryInfo: {
          type: 3,
          pagenum: 1,
          pagesize: 10
        },
        // 商品分类的数据列表
        cateList: [],
        // 总数据条数
        total: 0,
        // 为table指定列
        columns: [{
          label: '分类名称',
          prop: 'cat_name'
        }, {
          label: '是否有效',
          // 表示当前列定义为模板列
          type: 'template',
          // 当前列使用的模板名称
          template: 'isok'
        }, {
          label: '排序',
          // 表示当前列定义为模板列
          type: 'template',
          // 当前列使用的模板名称
          template: 'order'
        }, {
          label: '操作',
          // 表示当前列定义为模板列
          type: 'template',
          // 当前列使用的模板名称
          template: 'opt'
        }],
        // 控制添加分类对话框的显示与隐藏
        addCateDialogVisible: false,
        // 添加分类的表单数据对象
        addCateForm: {
          cat_name: '',
          cat_level: 0,
          // 默认分类等级为一级
          cat_pid: 0
        },
        // 添加分类表单的验证规则
        addCateFormRules: {
          cat_name: [{
            required: true,
            message: '请输入分类的名称',
            trigger: 'blur'
          }]
        },
        parentCateList: [],
        // 指定内联标签的配置属性
        cascaderProps: {
          value: 'cat_id',
          label: 'cat_name',
          children: 'children'
        },
        // 选中的父级分类的id数组
        selectedKeys: []
      }
    },
    created() {
      // 获取商品分类数据
      this.getCateList()
    },
    methods: {
      async getCateList() {
        const {
          data: res
        } = await this.$http.get('categories', {
          params: this.queryInfo
        })
        // console.log(res)
        if (res.meta.status !== 200) return this.$message.error('商品分类获取失败！')
        this.cateList = res.data.result
        // 为总数据条数赋值
        this.total = res.data.total
      },
      // 监听pagesize改变的事件
      handleSizeChange(newSize) {
        this.queryInfo.pagesize = newSize
        this.getCateList()
      },
      handleCurrentChange(newPage) {
        this.queryInfo.pagenum = newPage
        this.getCateList()
      },
      showAddCateDialog() {
        this.getParentCateList()
        this.addCateDialogVisible = true
      },
      // 获取添加分类的分类列表
      async getParentCateList() {
        const {
          data: res
        } = await this.$http.get('categories', {
          params: {
            type: 2
          }
        })
        // console.log(res.data)
        if (res.meta.status !== 200) return this.$message.error('分类列表获取失败')
        this.parentCateList = res.data
      },
      // 选择项发生变化触发
      parentCateChange() {
        // console.log(this.selectedKeys)
        // 如果 selectedKeys的length大于0，表示已经选中
        if (this.selectedKeys.length > 0) {
          // 父级分类的Id
          this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
          this.addCateForm.cat_level = this.selectedKeys.length
        } else {
          this.addCateForm.cat_pid = 0
          this.addCateForm.cat_level = 0
        }
      },
      // 点击添加新的分类
      addCate() {
        // console.log(this.addCateForm)
        this.$refs.addCateFormRef.validate(async (valid) => {
          if (!valid) return
          const {
            data: res
          } = await this.$http.post('categories',
            this.addCateForm
          )
          console.log(res)
          if (res.meta.status !== 201) return this.$message.error('添加商品分类失败')
          this.$message.success('添加分类成功！')
          this.addCateDialogVisible = false
          this.getCateList()
        })
      },
      clearAddCate() {
        this.$refs.addCateFormRef.resetFields()
        this.selectedKeys = []
        this.addCateForm.cat_level = 0
        this.addCateForm.cat_pid = 0
      }
    }
  }

</script>

<style lang="less" scoped>
  .treeTable {
    margin-top: 15px;
  }

  .el-cascader {
    width: 100%;
    height: 50px;
  }

</style>
