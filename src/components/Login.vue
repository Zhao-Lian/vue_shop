<template>
  <div class="login_container">
    <div class="login_box">
      <!-- 头像 -->
      <div class="avatar_box">
        <img src="../assets/logo.png" alt />
      </div>
      <!-- 登录区 -->
      <el-form ref="loginFormRef" :model="loginForm" class="login_form" :rules="loginFormRules">
        <!-- 用户名 -->
        <el-form-item prop="username">
          <el-input prefix-icon="el-icon-user" v-model="loginForm.username"></el-input>
        </el-form-item>
        <!-- 密码 -->
        <el-form-item prop="password">
          <el-input prefix-icon="el-icon-lock" v-model="loginForm.password" show-password></el-input>
        </el-form-item>
        <!-- 按钮 -->
        <el-form-item class="btns">
          <el-button type="primary" @click="login">登录</el-button>
          <el-button type="info" @click="resetForm">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  data: function() {
    return {
      // 登录表单的数据绑定对象
      loginForm: {
        username: 'admin',
        password: '123456'
      },
      // 表单验证规则对象
      loginFormRules: {
        username: [
          {
            required: true,
            message: '请输入用户名',
            trigger: 'blur'
          }
        ],
        password: [
          {
            required: true,
            message: '请输入密码',
            trigger: 'blur'
          },
          {
            min: 6,
            max: 12,
            message: '长度在 6 到 12 个字符',
            trigger: 'blur'
          }
        ]
      }
    }
  },
  methods: {
    resetForm: function() {
      // console.log(this)
      this.$refs.loginFormRef.resetFields()
    },
    login: function() {
      this.$refs.loginFormRef.validate(async (valid, obj) => {
        if (valid) {
          const { data: res } = await this.$http.post('login', {
            username: this.loginForm.username,
            password: this.loginForm.password
          })
          // console.log(res)
          if (res.meta.status === 200) {
            this.$message({
              message: '登录成功！',
              type: 'success'
            })
            window.sessionStorage.setItem('token', res.data.token)
            this.$router.push('/home')
          } else {
            this.$message.error('登录失败！')
          }
        } else {
          return false
        }
      })
    }
  }
}
</script>

<style lang="less" scoped>
.login_container {
  width: 100%;
  height: 100%;
  background-color: #2b4b6b;
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
    border-radius: 50%;
    overflow: hidden;
    width: 130px;
    height: 130px;
    position: absolute;
    left: 50%;
    top: 0;
    transform: translate(-50%, -50%);
    background-color: #fff;
    border: 1px solid #eee;
    padding: 10px;
    box-shadow: 0 0 5px #ddd;
    img {
      width: 100%;
      border-radius: 50%;
      background-color: #eee;
    }
  }
  .login_form {
    margin-top: 100px;
    padding: 0 35px;
  }
  .btns {
    text-align: right;
  }
}
</style>
