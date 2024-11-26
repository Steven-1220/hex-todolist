<script setup>
import axios from 'axios';
import { ref } from 'vue';
import { useRouter } from 'vue-router';
import Swal from 'sweetalert2';

const apiUrl = `https://todolist-api.hexschool.io`;
const router = useRouter();


// 註冊
const signUpData = ref({
  email: '',
  password: '',
  passwordConfirm: '',
  nickname: '',
});
const signUpResMessage = ref('');
const signUpResData = ref('');

const signUp = async () => {
  // 確認密碼是否相同
  if( signUpData.value.password !== signUpData.value.passwordConfirm) {
    Swal.fire({
      title: '兩次密碼輸入不相同',
      text: '請重新輸入密碼',
      icon: 'error',
      confirmButtonText: 'OK',
      confirmButtonColor: '#d71c1c'
    })
    signUpData.value.password = '';
    signUpData.value.passwordConfirm = '';
    return;
  }
  try {
    const resData = await axios.post(`${apiUrl}/users/sign_up`,signUpData.value);
    console.log(resData);
    signUpResData.value = resData.data.uid
    
    resetForm();
    Swal.fire({
      title: '註冊成功',
      icon: 'success',
      confirmButtonText: 'OK',
      confirmButtonColor: '#ffd370'
    })
    setTimeout(() => {
      router.push('/')
    }, 1000);
  } catch (error) {
    signUpResMessage.value = error.response.data.message;
    Swal.fire({
      title: `${signUpResMessage.value}`,
      text: '請使用其他的 email',
      icon: 'error',
      confirmButtonText: 'OK',
      confirmButtonColor: '#d71c1c'
    })
    resetForm();
  }
};

// reset 表單
const formRef = ref(null);
const resetForm = () => {
  formRef.value.resetForm();
};

</script>

<template>
  <div id="signUpPage" class="bg-yellow">
  <div class="container signUpPage vhContainer">
    <div class="side">
      <a href="#"><img class="logoImg" src="https://raw.githubusercontent.com/hexschool/2022-web-layout-training/main/todolist/logo.png" alt=""></a>
      <img class="d-m-n" src="https://raw.githubusercontent.com/hexschool/2022-web-layout-training/main/todolist/img.png" alt="workImg">
    </div>
    <div>
      <VForm class="formControls" ref="formRef" v-slot="{ errors, meta }" @submit="signUp()">
        <h2 class="formControls_txt">註冊帳號</h2>

        <label class="formControls_label" for="email">Email</label>
        <VField class="formControls_input form-control" type="email" id="email" name="email" placeholder="請輸入 email" rules="required|email" :class="{ 'is-invalid': errors['email'] }" v-model="signUpData.email" />
        <ErrorMessage name="email" class="invalid-feedback"/>

        <label class="formControls_label" for="name">您的暱稱</label>
        <VField class="formControls_input form-control" type="text" name="暱稱" id="name" placeholder="請輸入您的暱稱" rules="required" :class="{ 'is-invalid': errors['暱稱'] }" v-model="signUpData.nickname" />
        <ErrorMessage name="暱稱"/>

        <label class="formControls_label" for="pwd">密碼</label>
        <VField class="formControls_input form-control" type="password" name="密碼" id="pwd" placeholder="請輸入密碼" rules="required|min:6" :class="{ 'is-invalid': errors['密碼'] }" v-model="signUpData.password" />
        <ErrorMessage name="密碼"/>

        <label class="formControls_label" for="pwd-confirm">再次輸入密碼</label>
        <VField class="formControls_input form-control" type="password" name="再次輸入密碼" id="pwd-confirm" placeholder="請再次輸入密碼"  rules="required|min:6" :class="{ 'is-invalid': errors['再次輸入密碼'] }" v-model="signUpData.passwordConfirm" />
        <ErrorMessage name="再次輸入密碼"/>

        <button class="formControls_btnSubmit" type="submit" :disabled="!meta.valid">註冊帳號</button>
        <RouterLink class="formControls_btnLink" to="/">登入</RouterLink>
        
      </VForm>
    </div>
  </div>

</div>
</template>