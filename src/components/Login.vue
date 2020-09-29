<template>
  <div class="login_container">
    <div class="login_box">
      <!-- 头像区 -->
      <div class="avatar_box">
        <img src="../assets/logo.png" alt="" />
      </div>
      <!-- 登陆表单去区 -->
      <el-form
        label-width="0px"
        ref="loginFormRef"
        :rules="loginFormRules"
        :model="loginForm"
        class="login_form"
      >
        <!-- 用户名 -->
        <el-form-item prop="username">
          <el-input
            v-model="loginForm.username"
            prefix-icon="iconfont icon-user"
          ></el-input>
        </el-form-item>
        <!-- 密码 -->
        <el-form-item prop="password">
          <el-input
            v-model="loginForm.password"
            prefix-icon="iconfont icon-3702mima"
            type="password"
          ></el-input>
        </el-form-item>
        <!-- 按钮区 -->
        <el-form-item class="btns">
          <el-button type="primary" @click="login">登录</el-button>
          <el-button type="info" @click="resetLoginForm">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      loginForm: {
        username: "admin",
        password: "123456"
      },
      loginFormRules: {
        username: [
          { required: true, message: "请输入用户名", trigger: "blur" },
          { min: 3, max: 10, message: "长度在3到10个字符", trigger: "blur" }
        ],
        password: [
          { required: true, message: "请输入密码", trigger: "blur" },
          { min: 6, max: 10, message: "长度在6到10个字符", trigger: "blur" }
        ]
      }
    };
  },
  methods: {
    resetLoginForm() {
      // console.log(this)
      this.$refs.loginFormRef.resetFields();
    },
    // 登录请求
    login() {
      // 判断 valid 是否为 true 如果为 true 就发起请求，不为 true 就阻止这次登录请求
      this.$refs.loginFormRef.validate(async valid => {
        // console.log( valid )
        // 验证通过就配置axios，然后设置请求根路径
        if (!valid) return;
        // 解构出data
        // 再组件中直接用 this 发起登录请求，请求地址可以从api接口文档中查找，请求参数loginForm也要提供好
        // 为了简化promise操作，可以用await 和async来进行优化 await只能用在被async修饰得方法中
        //把外面的箭头函数修饰成异步函数这样的话我们就可以直接拿到服务器返回的数据，其中返回了6个数据
        // data返回的是真实得数据，所以直接解构出data属性，同时重新命名为res对象
        const {data: res} = await this.$http.post('login',this.loginForm);
        // console.log(res);
        // 再根据res.meta属性来判断是否登陆成功，再打印出消息
        if(res.meta.status !==200)return this.$add.error('登录失败');
        this.$add.success('登陆成功');
        // console.log(res);
        window.sessionStorage.setItem('token',res.data.token);
        // 通过编程是导航跳转到后台主页，路由地址是 /home
        this.$router.push('/home');
      });
    }
  }
};
</script>

<style lang="less" scoped>
.login_container {
  background-color: #2b4b6b;
  height: 100%;
}
.login_box {
  width: 450px;
  height: 300px;
  background-color: #fff;
  border-radius: 3px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  .avatar_box {
    height: 130px;
    width: 130px;
    border: 1px solid #eee;
    border-radius: 50%;
    padding: 10px;
    box-shadow: 0 0 10px #ddd;
    position: absolute;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: #fff;
    img {
      width: 100%;
      height: 100%;
      border-radius: 50%;
      background-color: #eee;
    }
  }
}
.login_form {
  position: absolute;
  bottom: 0;
  width: 100%;
  padding: 0 20px;
  box-sizing: border-box;
}
.btns {
  display: flex;
  justify-content: flex-end;
}
</style>
