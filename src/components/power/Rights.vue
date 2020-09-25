<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item><a href="/">权限管理</a></el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
        <!-- 为表格添加边框可以在table中加入border属性 隔行换色可以加入stripe属性-->
      <el-table :data="rightsList" border stripe>
          <el-table-column type="index"></el-table-column>
          <el-table-column label="权限名称" prop="authName"></el-table-column>
          <el-table-column label="路径" prop="path"></el-table-column>
          <!-- 美化权限可以加入el-tag组件 -->
          <el-table-column label="权限等级" prop="level">
              <!-- 通过作用域插槽的形式来自定义输出格式 加slot-scope，用scope来接受所有数据-->
              <template slot-scope="scope">
                  <!-- el-tag需要在element.js中按需导入Tag标签，然后注册为全局可用的组件 -->
                  <!-- 由type属性来选择tag的类型，也可以通过color属性来自定义背景色。 -->
                  <!-- 然后通过v-if判断来进行渲染 -->
                  <el-tag v-if="scope.row.level === '0'">一级</el-tag>
                  <el-tag type="success" v-else-if="scope.row.level === '1'">二级</el-tag>
                  <el-tag type="warning" v-else>三级</el-tag>
              </template>
          </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      //所有的权限列表
      rightsList: []
    }
  },
//   生命周期函数
  created() {
    //获取所有的权限
    this.getRightsList()
  },
  methods: {
    // 获取权限列表
    async getRightsList() {
        // 通过this.$http发起一个get请求，请求地址rights/其中:type直接写死
        //表明我们这次请求的是列表类型的权限数据
        //git函数回返回promise，我们可以加在方法之前使用async进行修饰，
        //之后可以从await返回的数据对象中，通过解构赋值的形式，把data解构出来。
        //然后重命名为res，简化这次promise操作
      const { data: res } = await this.$http.get('rights/list')
    //   再判断这次请求是否成功
    // 如果res.meta.status不等于200的话请求失败，return一个错误消息提示
      if(res.meta.status !==200){
          return this.$add.error('获取权限列表失败！')
      }
    //   如果没有失败的话不会执行这个return
    // 所以紧接着可以把获取到的status挂载到rightsList中，供页面的模板结构来进行使用
      this.rightsList=res.data
    //   console.log(this.rightsList)
    }
  }
}
</script>

<style lang="less" scoped></style>
