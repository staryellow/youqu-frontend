<template>
  <div style="padding-top:60px;  margin-left: -10px;;min-width: 380px">
    <div class="uploaderCenter">
      <img class="img" :src="defaultPicture">
    </div>
    <div style="padding-top: 8px"/>
    <van-row justify="center">
      <h3>"游趣 —— 匹配你的趣友"</h3>
    </van-row>
    <van-divider></van-divider>
    <!--  <van-uploader class="uploaderCenter" :after-read="afterRead"/>-->
    <div>
      <van-form @submit="onSubmit">
        <van-cell-group inset>
          <van-field
              v-model="username"
              :rules="[{ required: true, message: '请填写昵称!' }]"
              label="昵称"
              name="昵称"
              placeholder="昵称"
          />
          <van-field
              v-model="userAccount"
              :rules="[{ required: true, message: '请填写账号!' }]"
              label="账号"
              name="账号"
              placeholder="账号"
          />
          <van-field
              v-model="password"
              :rules="[{ required: true, message: '请填写密码!' }]"
              label="密码"
              name="密码"
              placeholder="密码"
              type="password"
          />
          <van-field
              v-model="checkPassword"
              :rules="[{ required: true, message: '请确认密码!' }]"
              label="确认密码"
              name="确认密码"
              placeholder="确认密码"
              type="password"
          />
          <div class="longin">
            <van-button block class="defaultLogin" native-type="submit" round type="primary">
              注册账号
            </van-button>
            <van-cell title="" to="/user/login" value="已有账号？点击登录"></van-cell>
          </div>
        </van-cell-group>
      </van-form>
    </div>
  </div>
</template>
<script lang="ts" setup>
import {ref} from "vue";
import {useRouter} from "vue-router";
import {Toast} from "vant";
import {defaultPicture} from "../../components/userCommon";
import myAxios from "../../plugins/myAxios";

const router = useRouter()
const userAccount = ref('');
const username = ref('');
const checkPassword = ref('');
const password = ref('');
const onSubmit = async () => {
  const register = await myAxios.post("/user/register", {
    "checkPassword": checkPassword.value,
    "userAccount": userAccount.value,
    "userPassword": password.value,
    "username": username.value
  })
  console.log(register)
  if (register.code == 0) {
    Toast.success('注册成功')
    await router.push("/user/login")
  }else{
    Toast.fail(register.description+'，请重试')
  }
};
</script>

<style scoped>
@import "../../assets/css/public.css";

.longin {
  margin: 16px 14px 0 5%;
}

.defaultLogin {
  flex: auto;
}

.uploaderCenter {
  margin-top: 8px;
  display: flex;
  justify-content: center;
}

</style>
