<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item><a href="/">权限管理</a></el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮区 -->
      <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表区域 -->
      <el-table :data="rolelist" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']"
              v-for="(item1, i1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag
                  closable
                  @close="removeRightById(scope.row, item1.id)"
                  >{{ item1.authName }}</el-tag
                >
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级和三级权限 -->
              <el-col :span="19">
                <!-- 通过 for 循环 嵌套渲染二级权限 -->
                <el-row
                  :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']"
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row, item2.id)"
                      >{{ item2.authName }}</el-tag
                    >
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      type="warning"
                      v-for="item3 in item2.children"
                      :key="item3.id"
                      closable
                      @close="removeRightById(scope.row, item3.id)"
                      >{{ item3.authName }}</el-tag
                    >
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
            <!-- <pre>
                      {{ scope.row }}
                  </pre
            > -->
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button
              size="mini"
              type="primary"
              icon="el-icon-edit"
              @click="showAddDialog(scope.row.id)"
              >编辑</el-button
            >
            <el-button
              size="mini"
              type="danger"
              icon="el-icon-delete"
              @click="showAddDialog"
              >删除</el-button
            >
            <el-button
              size="mini"
              type="warning"
              icon="el-icon-setting"
              @click="showSetRightDialog(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 分配权限对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%"
      @close="setRightDialogClosed"
    >
      <!-- 树形控件 -->
      <el-tree
        :data="rightslist"
        :props="treeProps"
        show-checkbox
        node-key="id"
        default-expand-all
        :default-checked-keys="defKeys"
        ref="treeRef"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
    <el-dialog title="编辑角色" :visible.sync="editDizlogVisibls" width="50%">
      <el-form
        :model="editForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="70px"
        @close="editAddClosed"
      >
        <el-form-item label="角色名称">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer">
        <el-button @click="editDizlogVisibls = false">取 消</el-button>
        <el-button type="primary" @click="editRoleInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      //所有角色列表数据
      rolelist: [],
      //控制分配权限的显示与隐藏
      setRightDialogVisible: false,
      //   所有权限的数据
      rightslist: [],
      //树形控件的属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      //默认选中的节点Id值数组 defKeys
      // 当defKeys清空后，需要在页面中定义一个递归函数，
      // 然后把角色信息传递到递归函数中，
      // 通过递归的形式把所有三级节点ID都保存到一个数组中
      //这样的话再把这个数组再赋值给defKeys，
      // 就能够实现点击分配权限的时候，把你已有的权限给他加载出来
      // 在methods节点中创建getLeafKeys，通过递归的形式，
      // 获取 角色 下所有三级权限的id，并保存到 defKeys 数组中
      defKeys: [],
      // 当前即将分配权限的角色id
      roleId: '',
      //控制编辑角色对话框得显示于隐藏
      editDizlogVisibls: false,
      //查询到的角色信息对象
      editForm: {},
      //修改角色列表的验证规则对象
      editFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' }
        ]
      }
    }
  },
  //   生命周期函数
  created() {
    //获取所有的权限
    this.getRolesList()
  },
  methods: {
    //获取所有角色的列表
    async getRolesList() {
      // 通过this.$http发起一个get请求，请求地址roles
      //表明我们这次请求的是列表类型的权限数据
      //git函数回返回 promise ，我们可以加在方法之前使用async进行修饰，
      //之后可以从await返回的数据对象中，通过解构赋值的形式，把data解构出来。
      //然后重命名为res，简化这次promise操作
      const { data: res } = await this.$http.get('roles')
      //   再判断这次请求是否成功
      // 如果res.meta.status不等于200的话请求失败，return一个错误消息提示
      if (res.meta.status !== 200) {
        return this.$add.error('获取角色列表失败！')
      }
      //   如果没有失败的话不会执行这个return
      // 所以紧接着可以把获取到的status挂载到 rolelist 中，供页面的模板结构来进行使用
      this.rolelist = res.data
      console.log(this.rolelist)
    },
    // 根据id删除对应的权限
    async removeRightById(role, rightId) {
      // 弹框提示用户是否删除
      const confirmResult = await this.$confirm(
        '此操作将永久删除该权限, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$add.info('取消了删除')
      }
      // console.log('确认了删除')
      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      )
      if (res.meta.status !== 200) {
        return this.$add.error('删除权限失败！')
      }
      this.$add.success('删除权限成功！')
      // this.getRolesList()
      role.children = res.data
    },
    //展示分配权限的对话框
    async showSetRightDialog(role) {
      this.roleId = role.id
      //获取所有权限的数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$add.error('获取权限失败!')
      }
      //获取到的权限数据保存的到data中
      this.rightslist = res.data
      console.log(this.rightslist)
      //递归获取三级节点的id,通过递归的形式，
      // 获取角色下所有三级权限的id，并保存到 defKeys 数组中
      this.getLeafKeys(role, this.defKeys)

      this.setRightDialogVisible = true
    },
    // 通过递归的形式，获取角色下所有三级权限的id，并保存到 defKeys 数组中
    getLeafKeys(node, arr) {
      // 如果当前 node 节点不包含 children 属性，则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    // 解决每次点击按钮时都会把当前角色的已有的三级权限id保存到之后的数组中，
    // 在关闭对话框时，没有清空那个数组，导致后面数组中的id会越来越多，
    // 解决办法
    // 在每次对话框关闭期间都清空一下当前数组里面的元素项
    // 1.绑定 setRightDialogClosed 事件处理函数
    // 2.监听分配权限对话框的关闭事件
    setRightDialogClosed() {
      // 3.在这个事件中为 this.defKeys 重新赋值 （清空里面的id）
      this.defKeys = []
    },
    // 点击为角色分配权限
    // 步骤1.点击分配权限按钮时，立即把当前角色的id roleId 先保存到data中供后续使用
    // 2.然后点击确认按钮时先获取到整个树形结构中半选和全选状态id值，
    async allotRights() {
      // 3. 然后合并为一个完整地数组 keys
      const keys = [
        // ...  展开运算符
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      // console.log(keys)
      // 4.再把得到的数组进行字符串的拼接，用 , 英文的逗号拼接为字符串
      const idStr = keys.join(',')

      // 5.发起一次请求把保存的角色id this.roleId，和新拼接出来的 idStr
      // 发送到服务器端
      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        { rids: idStr }
      )
      //  6.当请求失败之后直接提示用户 分配权限失败！ 如果没有失败，提示 分配权限成功！
      // 并且刷新角色列表，同时关闭分配权限的对话框
      if (res.meta.status !== 200) {
        return this.$add.error('分配权限失败！')
      }
      // 分配成功，就弹框提示用户分配成功
      this.$add.success('分配权限成功！')
      // 重新刷新列表
      this.getRolesList()
      // 之后重新把对话框隐藏关闭起来
      this.setRightDialogVisible = false
    },
    //展示编辑角色用户信息对话框
    async showAddDialog(id) {
      // console.log(id)
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$add.error('查询角色用户失败！')
      }
      this.editForm = res.data
      this.editDizlogVisibls = true
    },
    //监听编辑角色对话框的关闭事件
    editAddClosed() {
      this.$refs.editFormRef.resetFields()
    },
    //修改角色信息并提交
    editRoleInfo() {
      this.$refs.editFormRef.validate(async valid => {
        // console.log(valid)
        if (!valid) return
        //发起编辑修改角色信息得数据请求
       const {data:res} = await this.$http.put('roles/' + this.editForm.id, {
          roleName: this.editForm.roleName,
          roleDesc: this.editForm.roleDesc
        })
        if(res.meta.status !==200){
          return this.$add.error('更新角色信息失败')
        }
       
      })
    }
  }
}
</script>

<style lang="less">
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eeeeee;
}
.bdbottom {
  border-bottom: 1px solid #eeeeee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
