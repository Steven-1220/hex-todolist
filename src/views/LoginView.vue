<script setup>
import axios from 'axios';
import { ref } from 'vue';
import { useRouter } from 'vue-router';

const apiUrl = `https://todolist-api.hexschool.io`;
const userToken = ref('')
const router = useRouter();

// 登入
const signInData = ref({
  email: '',
  password: '',
});

const signIn = async () => {
  try {
    const resData = await axios.post(`${apiUrl}/users/sign_in`,signInData.value); 
    userToken.value = resData.data.token;
    document.cookie = `customTodoListToken=${userToken.value}`;
    resetForm();
    setTimeout(() => {
      router.push('/todo')
    }, 500);
  } catch (error) {
    alert(error.response.data.message)
  }
};

// reset 表單
const formRef = ref(null);
const resetForm = () => {
  formRef.value.resetForm();
};
</script>

<template>
  <div id="loginPage" class="bg-yellow">
  <div class="container loginPage vhContainer ">
    <div class="side">
      <a href="#"><img class="logoImg" src="https://raw.githubusercontent.com/hexschool/2022-web-layout-training/main/todolist/logo.png" alt=""></a>
      <img class="d-m-n" src="https://raw.githubusercontent.com/hexschool/2022-web-layout-training/main/todolist/img.png" alt="workImg">
    </div>
    <div>
      <VForm class="formControls" ref="formRef" v-slot="{ errors, meta }" @submit="signIn()">
        <h2 class="formControls_txt">最實用的線上代辦事項服務</h2>
        <label class="formControls_label" for="email">Email</label>
        <VField class="formControls_input form-control" type="email" id="email" name="email" placeholder="請輸入 email" rules="required|email" :class="{ 'is-invalid': errors['email'] }" v-model="signInData.email" />
        <ErrorMessage name="email" class="invalid-feedback"/>

        <label class="formControls_label" for="pwd">密碼</label>
        <VField class="formControls_input form-control" type="password" name="密碼" id="pwd" placeholder="請輸入密碼" rules="required|min:6" :class="{ 'is-invalid': errors['密碼'] }" v-model="signInData.password" />
        <ErrorMessage name="密碼"/>

        <button type="submit" class="formControls_btnSubmit" :disabled="!meta.valid">登入</button>
        <RouterLink class="formControls_btnLink" to="/register" >註冊帳號</RouterLink>
      </VForm>
    </div>
  </div>
</div>
</template>
