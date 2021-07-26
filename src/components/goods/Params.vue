<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 警告区域 -->
      <el-alert show-icon title="注意：只允许为第三级分类设置相关参数！" type="warning" :closable="false"></el-alert>
      <!-- 选择商品分类区域 -->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <!-- 选择商品分类的级联选择框 -->
          <el-cascader v-model="selectedCateKeys" :options="catelist" :props="cateProps" @change="handleChange">
          </el-cascader>
        </el-col>
      </el-row>

      <!-- tab页签区域 -->
      <el-tabs v-model="activeName" @tab-click="handleClick">
        <el-tab-pane label="用户管理" name="first">用户管理</el-tab-pane>
        <el-tab-pane label="配置管理" name="second">配置管理</el-tab-pane>
      </el-tabs>

    </el-card>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        // 商品分类列表
        catelist: [],
        // 级联选择器的配置对象
        cateProps: {
          value: 'cat_id',
          label: 'cat_name',
          children: 'children'
        },
        // 级联选择器双向绑定到的数组
        selectedCateKeys: []
      }
    },
    created() {
      this.getCateList()
    },
    methods: {
      // 获取所有的商品分类列表
      async getCateList() {
        const {
          data: res
        } = await this.$http.get('categories')
        // console.log('111', res)
        if (res.meta.status !== 200) {
          return this.$message.console.error('获取商品分类失败！')
        }
        this.catelist = res.data
      },
      // 级联选择框选中项变化，会触发这个函数
      handleChange() {
        // 证明选中的不是三级分类
        if (this.selectedCateKeys.lenght !== 3) {
          this.selectedCateKeys = []
        }

        // 证明选中的是三级分类
        console.log(this.selectedCateKeys)
      }
    }
  }

</script>

<style lang="less" scoped>
  .cat_opt {
    margin: 15px 0;
  }

</style>
