<!--suppress ALL -->
<template>
  <div class="login-container columnCC">
    <el-form ref="refloginForm" size="medium" class="login-form" :model="formInline" :rules="formRulesMixin">
      <div class="title-container">
        <h3 class="title text-center">{{ settings.title }}</h3>
      </div>
      <el-form-item prop="username" :rules="formRulesMixin.isNotNull">
        <div class="rowSC">
          <span class="svg-container">
            <svg-icon icon-class="user" />
          </span>
          <el-input v-model="formInline.username" placeholder="用户名(admin)" />
          <!--占位-->
          <div class="show-pwd" />
        </div>
      </el-form-item>
      <!--<el-form-item prop="password" :rules="formRulesMixin.passwordValid">-->
      <el-form-item prop="password" :rules="formRulesMixin.isNotNull">
        <div class="rowSC">
          <span class="svg-container">
            <svg-icon icon-class="password" />
          </span>
          <el-input
            :key="passwordType"
            ref="refPassword"
            v-model="formInline.password"
            :type="passwordType"
            name="password"
            placeholder="password(123456)"
            @keyup.enter="handleLogin"
          />
          <span class="show-pwd" @click="showPwd">
            <svg-icon :icon-class="passwordType === 'password' ? 'eye' : 'eye-open'" />
          </span>
        </div>
      </el-form-item>
      <div class="tip-message">{{ tipMessage }}</div>
      <el-button :loading="loading" type="primary" class="login-btn" size="medium" @click.prevent="handleLogin">
        Login
      </el-button>
    </el-form>
  </div>
</template>

<script>
/*可以设置默认的名字*/
export default {
  name: 'Login'
}
</script>

<script setup>
import { reactive, getCurrentInstance, watch, ref } from 'vue'
import settings from '@/settings'
import { useRoute } from 'vue-router'
import { useStore } from 'vuex'
import { ElMessage } from 'element-plus'
let { proxy } = getCurrentInstance()
//form
let formInline = reactive({
  username: 'admin',
  password: '123456'
})
let state = reactive({
  otherQuery: {},
  redirect: undefined
})

/* listen router change  */
const route = useRoute()
let getOtherQuery = (query) => {
  return Object.keys(query).reduce((acc, cur) => {
    if (cur !== 'redirect') {
      acc[cur] = query[cur]
    }
    return acc
  }, {})
}

watch(
  route,
  (route) => {
    const query = route.query
    if (query) {
      state.redirect = query.redirect
      state.otherQuery = getOtherQuery(query)
    }
  },
  { immediate: true }
)

/*
 *  login relative
 * */
let loading = ref(false)
let tipMessage = ref('')
const store = useStore()
let handleLogin = () => {
  let refloginForm = ''
  proxy.$refs['refloginForm'].validate((valid) => {
    if (valid) {
      loginReq()
    } else {
      return false
    }
  })
}
let loginReq = () => {
  loading.value = true
  store
    .dispatch('user/login', formInline)
    .then(() => {
      proxy.$router.push({ path: state.redirect || '/', query: state.otherQuery })
      ElMessage({ message: '登录成功', type: 'success' })
    })
    .catch((res) => {
      tipMessage.value = res.msg
      proxy.sleepMixin(30).then(() => {
        loading.value = false
      })
    })
}

/*
 *  password show or hidden
 * */
let passwordType = ref('password')
const refPassword = ref(null)
let showPwd = () => {
  if (passwordType.value === 'password') {
    passwordType.value = ''
  } else {
    passwordType.value = 'password'
  }
  proxy.$nextTick(() => {
    refPassword.value.focus()
  })
}
</script>

<style lang="scss" scoped>
$bg: #2d3a4b;
$dark_gray: #889aa4;
$light_gray: #eee;
.login-container {
  height: 100vh;
  width: 100%;
  background-color: #2d3a4b;
  .login-form {
    margin-bottom: 20vh;
    width: 360px;
  }
  .title-container {
    .title {
      font-size: 22px;
      color: #eee;
      margin: 0px auto 25px auto;
      text-align: center;
      font-weight: bold;
    }
  }
}

.svg-container {
  padding-left: 6px;
  color: $dark_gray;
  text-align: center;
  width: 30px;
}

//错误提示信息
.tip-message {
  color: #e4393c;
  height: 30px;
  margin-top: -12px;
  font-size: 12px;
}

//登录按钮
.login-btn {
  width: 100%;
  margin-bottom: 30px;
}
.show-pwd {
  width: 50px;
  font-size: 16px;
  color: $dark_gray;
  cursor: pointer;
  text-align: center;
}
</style>

<style lang="scss">
//css 样式重置 增加个前缀避免全局污染
.login-container {
  .el-form-item {
    border: 1px solid rgba(255, 255, 255, 0.1);
    background: rgba(0, 0, 0, 0.1);
    border-radius: 5px;
    color: #454545;
  }
  .el-input input {
    background: transparent;
    border: 0px;
    -webkit-appearance: none;
    border-radius: 0px;
    padding: 10px 5px 10px 15px;
    color: #fff;
    height: 42px; //此处调整item的高度
    caret-color: #fff;
  }
}
</style>
