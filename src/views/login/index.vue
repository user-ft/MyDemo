<template>
  <div class="login-wrap">
    <div class="login-body">
      <div class="login-logo">
        <img src="./logo_index.png">
      </div>
      <el-form ref="form" :rules="rules" :model="form">
        <el-form-item prop="mobile">
          <el-input v-model="form.mobile" placeholder="请输入手机号"></el-input>
        </el-form-item>
        <el-form-item prop="code">
          <el-col :span="16">
            <el-input v-model="form.code" placeholder="请输入验证码"></el-input>
          </el-col>
          <el-col :span="6" :offset="2">
            <el-button
            @click.prevent="handleSendCode"
            >获取验证码</el-button>
          </el-col>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" class="btn-login" @click.prevent="handleLogin"  :loading="loginLoading">登陆</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script >
import axios from 'axios'
import '@/vendor/gt'
export default {
  name: 'AppLogin',
  data () {
    return {
      form: {
        mobile: '',
        code: ''
      },
      rules: {
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { len: 11, message: '长度必须11个字符', trigger: 'blur' }
        ],
        code: [
          { required: true, message: '请输入验证码', trigger: 'blur' },
          { len: 6, message: '长度必须6个字符', trigger: 'blur' }
        ]
      },
      captchaObj: null,
      loginLoading: false
    }
  },
  components: {},
  methods: {
    handleLogin () {
      this.$refs['form'].validate((valid) => {
        if (!valid) {
          return
        }
        this.btnLogin()
      })
    },
    btnLogin () {
      this.loginLoading = true
      axios({
        method: 'POST',
        url: 'http://ttapi.research.itcast.cn/mp/v1_0/authorizations',
        data: this.form
      }).then(res => {
        this.loginLoading = false
        this.$message({
          message: '登陆成功',
          type: 'success'
        })
        this.$router.push({ name: 'home' })
      }).catch(err => {
        if (err.response.status === 400) {
          this.$message.error('登陆失败，手机号或验证码错误')
        }
        this.loginLoading = false
      })
    },
    handleSendCode () {
      const { mobile } = this.form
      if (this.captchaObj) {
        return this.captchaObj.verify()
      }
      axios({
        method: 'GET',
        url: `http://ttapi.research.itcast.cn/mp/v1_0/captchas/${mobile}`
      }).then(res => {
        const { data } = res.data
        window.initGeetest({
          gt: data.gt,
          challenge: data.challenge,
          offline: !data.success,
          new_captcha: data.new_captcha,
          product: 'bind'
        },
        captchaObj => {
          this.captchaObj = captchaObj
          captchaObj.onReady(function () {
            captchaObj.verify()
          })
          captchaObj.onSuccess(function () {
            const { geetest_challenge: challenge, geetest_seccode: seccode, geetest_validate: validate } = captchaObj.getValidate()
            axios({
              method: 'GET',
              url: `http://ttapi.research.itcast.cn/mp/v1_0/sms/codes/${mobile}`,
              params: {
                challenge, seccode, validate
              }
            })
          })
        })
      })
    }
  }
}
</script>

<style lang='less' scoped >
.login-wrap {
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  background-image: url(./login_bg.jpg);
  .login-logo {
    text-align: center;
    padding: 20px 0 20px 0;
  }
  .login-body {
    width: 450px;
    height: 350px;
    background-color: #fff;
    padding: 10px 40px 0;
    .btn-login {
      width: 100%;
    }
  }
}
</style>
