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
          <el-button type='primary'>添加分类</el-button>
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
        }]
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
        console.log(res)
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
      }
    }
  }

</script>

<style lang="less" scoped>
  .treeTable {
    margin-top: 15px;
  }

</style>
