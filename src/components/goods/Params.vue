<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!--卡片视图区域-->
    <el-card>
      <el-alert show-icon title="注意：只允许为第三级分类设置相关参数！" type="warning" :closable="false">
      </el-alert>
      <el-row :gutter="10" class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <!--       选择商品分类的级连选择框-->
          <el-cascader
            v-model="selectedCateKeys"
            :options="catelist"
            :props="cateProps"
            @change="handleChange"></el-cascader>
        </el-col>
      </el-row>
    </el-card>

  </div>
</template>
<script>
export default {
  data() {
    return {
      // 商品分类列表
      catelist: [],
      // 级联选择器配置对象
      cateProps: {
        // expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 级联选择器选中绑定的数组
      selectedCateKeys: []
    }
  },
  created() {
    this.getcateList()
  },
  methods: {
    // 获取所有的商品分类
    async getcateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败！')
      }
      this.catelist = res.data
      console.log(this.catelist)
    },
    handleChange() {
      // console.log(this.selectedCateKeys)
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
      }
    }
  }
}
</script>
<style lang="less" scoped>
.cat_opt {
  margin: 15px 0;
}
</style>
