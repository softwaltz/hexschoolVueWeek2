<template>
  <div v-if="loginUser.token == ''">
    <h2>登入</h2>
    <input type="email" placeholder="Email" v-model="signInData.email" />
    <input type="password" placeholder="Password" v-model="signInData.password" />
    <button type="button" @click="signIn">Sign In</button>
    <div>Token: {{ loginUser.token }}</div>
    <div v-if="signInErrMsg != ''">{{ signInErrMsg }}</div>
  </div>
  <br />
  <div v-if="loginUser.token != ''">
    <h2>驗證</h2>
    <input type="text" placeholder="Token" :value="loginUser.token" />
    <button type="button" @click="checkOut">Check out</button>
    <div>Check Status: {{ checkOutStatus }}</div>
    <div v-if="checkOutErrMsg != ''">{{ checkOutErrMsg }}</div>
  </div>
  <div v-if="loginUser.token != ''">
    <h2>登出</h2>
    <input type="text" placeholder="Token" :value="loginUser.token" />
    <button type="button" @click="signOut">Sign out</button>
    <div>Status: {{ signOutStatus }}</div>
    <div v-if="signOutErrMsg != ''">{{ signOutErrMsg }}</div>
  </div>
  <hr />
  <div v-if="loginUser.status">
    <h2>Hi, {{ loginUser.nickname }}</h2>
    待辦清單
    <br />
    <input type="text" placeholder="工作項目" v-model="newTodo.content" />
    <button type="button" @click="addNewTodo">新增</button>
    <br />
    <br />
    <br />
    <table>
      <thead>
        <tr>
          <td></td>
          <td>項目</td>
          <td>狀態</td>
          <td></td>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(todo, i) in todos" :key="todo.id">
          <td>{{ i + 1 }}</td>
          <td>{{ todo.content }}</td>
          <td>{{ todo.status ? '完成' : '待辦中' }}</td>
          <td>
            <input type="text" v-model="todo.tmpContent" />
            <button type="button" @click="updateTodo(todo)">Update</button>
            <button type="button" @click="deleteTodo(todo.id)">Delete</button>
            <button type="button" @click="toggleStatus(todo.id)">Toggle Status</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'

const loginUser = ref({
  nickname: '',
  status: '',
  token: '',
  exp: 0
})

const apiUrl = 'https://todolist-api.hexschool.io'

const signInData = ref({
  email: 'terryd@kali.org',
  password: 'terryd123'
})
const signInErrMsg = ref('')
const signIn = () => {
  axios
    .post(apiUrl + '/users/sign_in', signInData.value)
    .then((resp) => {
      loginUser.value.nickname = resp.data.nickname
      loginUser.value.exp = resp.data.exp
      loginUser.value.status = resp.data.status
      loginUser.value.token = resp.data.token
      signInErrMsg.value = ''

      document.cookie = `hexschoolTodo=${loginUser.value.token}`
      getAllTodos()
    })
    .catch((error) => {
      signInErrMsg.value = error.message
    })
}

const checkOutStatus = ref('')
const checkOutErrMsg = ref('')
const checkOut = () => {
  const tokenInCookie = document.cookie.replace(
    /(?:(?:^|.*;\s*)hexschoolTodo\s*=\s*([^;]*).*$)|^.*$/,
    '$1'
  )
  axios
    .get(apiUrl + '/users/checkout', {
      headers: {
        Authorization: tokenInCookie
      }
    })
    .then((resp) => {
      checkOutStatus.value = resp.data.uid
      checkOutErrMsg.value = ''
    })
    .catch((error) => {
      checkOutStatus.value = false
      checkOutErrMsg.value = error.message
    })
}

const signOutStatus = ref('')
const signOutErrMsg = ref('')
const signOut = () => {
  const tokenInCookie = document.cookie.replace(
    /(?:(?:^|.*;\s*)hexschoolTodo\s*=\s*([^;]*).*$)|^.*$/,
    '$1'
  )
  axios
    .post(
      apiUrl + '/users/sign_out',
      {},
      {
        headers: {
          Authorization: tokenInCookie
        }
      }
    )
    .then((resp) => {
      signOutStatus.value = resp.data.message
      signOutErrMsg.value = ''
      loginUser.value.nickname = ''
      loginUser.value.exp = ''
      loginUser.value.status = false
      loginUser.value.token = ''

      document.cookie = `hexschoolTodo=`
    })
    .catch((error) => {
      signOutStatus.value = ''
      signOutErrMsg.value = error.message
    })
}

const todos = ref({})
const getAllTodos = () => {
  const tokenInCookie = document.cookie.replace(
    /(?:(?:^|.*;\s*)hexschoolTodo\s*=\s*([^;]*).*$)|^.*$/,
    '$1'
  )
  axios
    .get(apiUrl + '/todos', {
      headers: {
        Authorization: tokenInCookie
      }
    })
    .then((resp) => {
      todos.value = resp.data.data
      todos.value.forEach((todo) => {
        todo.tmpContent = todo.content
      })
    })
    .catch((error) => {
      console.log(error)
    })
}

const newTodo = ref({
  content: ''
})
const addNewTodo = () => {
  if (newTodo.value.content != '') {
    const tokenInCookie = document.cookie.replace(
      /(?:(?:^|.*;\s*)hexschoolTodo\s*=\s*([^;]*).*$)|^.*$/,
      '$1'
    )
    axios
      .post(apiUrl + '/todos', newTodo.value, {
        headers: {
          Authorization: tokenInCookie
        }
      })
      .then(() => {
        getAllTodos()
        addNewTodo.content = ''
      })
      .catch((error) => {
        console.log(error)
      })
  }
}

const updateTodo = async (todo) => {
  const tokenInCookie = document.cookie.replace(
    /(?:(?:^|.*;\s*)hexschoolTodo\s*=\s*([^;]*).*$)|^.*$/,
    '$1'
  )
  await axios
    .put(
      apiUrl + '/todos/' + todo.id,
      {
        content: todo.tmpContent
      },
      {
        headers: {
          Authorization: tokenInCookie
        }
      }
    )
    .then(() => {
      getAllTodos()
    })
    .catch((error) => {
      console.log(error)
    })
}

const deleteTodo = async (id) => {
  const tokenInCookie = document.cookie.replace(
    /(?:(?:^|.*;\s*)hexschoolTodo\s*=\s*([^;]*).*$)|^.*$/,
    '$1'
  )
  await axios
    .delete(apiUrl + '/todos/' + id, {
      headers: {
        Authorization: tokenInCookie
      }
    })
    .then(() => {
      getAllTodos()
    })
    .catch((error) => {
      console.log(error)
    })
}

const toggleStatus = async (id) => {
  const tokenInCookie = document.cookie.replace(
    /(?:(?:^|.*;\s*)hexschoolTodo\s*=\s*([^;]*).*$)|^.*$/,
    '$1'
  )
  await axios
    .patch(
      apiUrl + '/todos/' + id + '/toggle',
      {},
      {
        headers: {
          Authorization: tokenInCookie
        }
      }
    )
    .then(() => {
      getAllTodos()
    })
    .catch((error) => {
      console.log(error)
    })
}
</script>

<style></style>
