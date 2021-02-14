<template>
  <div class="login_container">
    <div class="login_box">
      <div class="avatar_box">
        <img src="../assets/logo.png" alt="">
      </div>
      <el-form ref='loginFormRef' class="login_form" :model="loginForm" :rules="loginFormRules">
        <!-- 用户名 -->
        <el-form-item prop="username">
          <el-input prefix-icon="el-icon-user" v-model='loginForm.username'></el-input>
        </el-form-item>
        <!-- 密码 -->
        <el-form-item prop="password">
          <el-input prefix-icon="el-icon-lock" v-model='loginForm.password' show-password></el-input>
        </el-form-item>
        <el-form-item class="btns">
          <el-button type="primary" @click='login'>登录</el-button>
          <el-button type="info" @click='resetLoginForm'>重置</el-button>
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
          username: 'admin',
          password: '123456'
        },
        //   这是表单的验证规则
        loginFormRules: {
          username: [{
              required: true,
              message: '请输入用户名',
              trigger: 'blur'
            },
            {
              min: 3,
              max: 10,
              message: '长度在 3 到 10 个字符',
              trigger: 'blur'
            }
          ],
          password: [{
            required: true,
            message: '请输入密码',
            trigger: 'blur'
          }, {
            min: 6,
            max: 15,
            message: '长度在 6 到 15 个字符',
            trigger: 'blur'
          }]
        }
      }
    },

    methods: {
      resetLoginForm() {
        console.log(this)
        this.$refs.loginFormRef.resetFields()
      },
      login() {
        this.$refs.loginFormRef.validate(async (valid) => {
          //   console.log(valid)
          if (!valid) return
          const {
            data: res
          } = await this.$http.post('login', this.loginForm)
          if (res.meta.status !== 200) return this.$message.error('登录失败，请重新登录')
          this.$message({
            message: '登录成功了！',
            type: 'success'
          })
          // console.log(res.data.token)
          window.sessionStorage.setItem('token', res.data.token)
          this.$router.push('/home')
        })
      }
    }
  }

</script>

<style lang="less" scoped>
  .login_container {
    height: 100%;
    background-color: #2b4b6b;
  }

  .login_box {
    width: 450px;
    height: 300px;
    background-color: #fff;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
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

  .avatar_box {
    position: absolute;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 130px;
    height: 130px;
    border: 1px solid #eee;
    border-radius: 50%;
    padding: 10px;
    overflow: hidden;
    box-shadow: 0 0 10px #ddd;
    background-color: #fff;

    img {
      width: 100%;
      border-radius: 50%;
      height: 100%;
      background-color: #eee;
    }

    ;
  }

</style>
