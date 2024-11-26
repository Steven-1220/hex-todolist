<script setup>
import axios from 'axios';
import { computed, onMounted, ref } from 'vue';
import { useRouter } from 'vue-router';
import Swal from 'sweetalert2';

import Loading from 'vue-loading-overlay'


const isLoading = ref(false);
const fullPage = ref(true);
const loadingEffect = ()=> {
  isLoading.value = true
  setTimeout(() => {
    isLoading.value = false
  }, 1000)
}

onMounted(()=>{
  loadingEffect()
});
const apiUrl = `https://todolist-api.hexschool.io`;
const userToken = ref('')
const router = useRouter();


// 驗證登入
const userCheckoutResData = ref({
  uid: '',
  nickname: ''
});

const userCheckout = async () => {
  const token = document.cookie.replace(
    /(?:(?:^|.*;\s*)customTodoListToken\s*=\s*([^;]*).*$)|^.*$/,
    "$1",
  );
  userToken.value = token;
  try {
    const resData = await axios.get(`${apiUrl}/users/checkout`,{
      headers: {
        authorization:  userToken.value
      }
    })
    userCheckoutResData.value = resData.data
    getAllTodoItems()
    
  } catch (error) {
    
    Swal.fire({
      title: '驗證失效',
      icon: 'error',
      text: '將導回登入頁面',
      confirmButtonText: 'OK',
      confirmButtonColor: '#d71c1c'
    })
    router.push('/');
    setTimeout(() => {
      router.push('/')
    }, 500);
  }
};

onMounted( async () => {
  userCheckout();
});

// 登出，要注意要放一個空物件傳入
const signOutMessage = ref('');
const signOut = async () => {

  try {
    const resData = await axios.post(`${apiUrl}/users/sign_out`,{},{
      headers: {
        authorization: userToken.value,
      }
    })
    signOutMessage.value = resData.data.message;
    Swal.fire({
      title: `${resData.data.message}`,
      icon: 'success',
      text: '將回到登入頁面',
      confirmButtonText: 'OK',
      confirmButtonColor: '#ffd370'
    })
    router.push('/')
  } catch (error) {
    alert(error.response.data.message)
  }
};

// 取得所有待辦事項
const todoItems = ref([]);

const getAllTodoItems = async () =>  {
  try {
    const resData = await axios.get(`${apiUrl}/todos/`,{
      headers: {
        authorization: userToken.value,
      }
    })
    todoItems.value = resData.data.data
    
  } catch (error) {
    console.log(error.response);
  }
};


// 新增待辦事項
const newTodoContent = ref({
  content: '',
});

const addTodo = async () => {
  if(newTodoContent.value.content.trim() === ''){
    Swal.fire({
      title: '待辦事項不能是空的',
      icon: 'warning',
      confirmButtonText: 'OK',
      confirmButtonColor: '#ffd370'
    })
    return
  }
  try {
    const resData = await axios.post(`${apiUrl}/todos/`,newTodoContent.value,{
      headers: {
        authorization: userToken.value,
      }
    })
    console.log(resData);
    getAllTodoItems();
    newTodoContent.value = {
      content: ''
    }
  } catch (error) {
    console.log(error.response);
  }
};

// 刪除待辦事項
const deleteTodoItem = async (id) => {
  
  const result = await Swal.fire({
    title: "確認是否要刪除?",
    text: '刪除後無法回復',
    icon: "warning",
    showCancelButton: true,
    confirmButtonColor: "#3085d6",
    cancelButtonColor: '#d71c1c',
    confirmButtonText: '刪除',
    cancelButtonText: '取消',
  })
  
  if(result.isConfirmed) {
    try {
      await axios.delete(`${apiUrl}/todos/${id}`,{
        headers: {
          authorization: userToken.value,
        }
      })
      getAllTodoItems();
      
    } catch (error) {
      console.log(error.response);
    }
  }
};

// 編輯待辦事項
const tempEditTodo = ref({
  id:'',
  content: ''
});


const editTodo = async (id) => {
  const index = todoItems.value.findIndex(item => item.id === id)
  tempEditTodo.value.id = id;
  tempEditTodo.value.content = todoItems.value[index].content;
};

// 確認編輯待辦事項
const confirmEdit = async (id) => {
  try {
    const resData = await axios.put(`${apiUrl}/todos/${id}`,
      {
        content: tempEditTodo.value.content
      },
      {
        headers: {
          authorization: userToken.value,
        }
      }
    )
    console.log(resData);
    cancelEdit();
    getAllTodoItems();
  } catch (error) {
    console.log(error.response);
  }
  
};

// 取消編輯
const cancelEdit = () => {
  tempEditTodo.value = {};
};

// 切換待辦事項狀態
const toggleTodoStatus = async (id) => {
  try {
    const resData = await axios.patch(`${apiUrl}/todos/${id}/toggle`,{},
      {
        headers: {
          authorization: userToken.value,
        }
      }
    )
    console.log(resData.data);
  } catch (error) {
    console.log(error.response);
    
  }
};

// 顯示不同待辦事項狀態清單
const showTodoStatus = ref('all');
const showDifferentTodoList = computed(()=> {
  if(showTodoStatus.value === 'all'){
    return todoItems.value
  } else if (showTodoStatus.value === 'incomplete') {
    return todoItems.value.filter(item => item.status === false)
  } else {
    return todoItems.value.filter(item => item.status === true)
  }
});

// 顯示已完成待辦數量
const showCompletedTodoNum = computed(()=>{
  const completeItems =  todoItems.value.filter(item => item.status === true)
  return completeItems.length
});
</script>

<template>
  <Loading v-model:active="isLoading"  :is-full-page="fullPage" />
  <div id="todoListPage" class="bg-half">
    <nav>
      <h1><a href="#">ONLINE TODO LIST</a></h1>
      <ul>
        <li class="todo_sm"><a href="#" @click.prevent="showTodoStatus = 'all'"><span>{{ userCheckoutResData.nickname }}的代辦</span></a></li>
        <li><a href="#" @click.prevent="signOut()">登出</a></li>
      </ul>
    </nav>
    <div class="container todoListPage vhContainer">
      <div class="todoList_Content">
        <div class="inputBox">
          <input type="text" placeholder="請輸入待辦事項" v-model="newTodoContent.content">
          <a href="#" class="d-flex align-items-center justify-content-center" @click.prevent="addTodo()">
            <i class="fa fa-plus"></i>
          </a>
        </div>
        <div class="todoList_list">
          <ul class="todoList_tab">
            <li>
              <a href="#" :class="{'active': showTodoStatus === 'all'}" @click.prevent="showTodoStatus = 'all'">全部</a>
            </li>
            <li>
              <a href="#" :class="{'active': showTodoStatus === 'incomplete'}" @click.prevent="showTodoStatus = 'incomplete'">待完成</a>
            </li>
            <li>
              <a href="#" :class="{'active': showTodoStatus === 'done'}" @click.prevent="showTodoStatus = 'done'">已完成</a>
            </li>
          </ul>
          <div class="todoList_items">
            <h2 class="fs-3" v-if="todoItems.length === 0">目前尚無待辦事項</h2>
            <ul class="todoList_item" v-else>
              <li v-for="todo in showDifferentTodoList" :key="todo.id">
                <div class="w-100 d-flex" v-if="tempEditTodo.id === todo.id">
                  <input class="form-control" type="text" v-model.trim="tempEditTodo.content">
                  <button href="#" class="btn btn-sm btn-warning text-nowrap mx-2" @click.prevent="confirmEdit(tempEditTodo.id)">
                    確認
                  </button>
                  <button href="#" class="btn btn-sm btn-danger text-nowrap" @click.prevent="cancelEdit()">
                    取消
                  </button>
                </div>
                <div class="d-flex w-100" v-else>
                  <label class="todoList_label">
                    <input class="todoList_input" type="checkbox" v-model="todo.status" @click="toggleTodoStatus(todo.id)">
                    <span :class="{'text-decoration-line-through': todo.status}">{{ todo.content }}</span>
                  </label>
                  <a href="#" @click.prevent="editTodo(todo.id)">
                    <i class="bi bi-pencil"></i>
                  </a>
                  <a href="#" @click.prevent="deleteTodoItem(todo.id)">
                    <i class="fa fa-times"></i>
                  </a>
                </div>
              </li>
            </ul>
            <div class="todoList_statistics">
              <p> {{ showCompletedTodoNum }}個已完成項目</p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
  
</template>