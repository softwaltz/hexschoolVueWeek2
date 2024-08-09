<template>
  <h2>註冊</h2>
  <div>
    <input type="email" placeholder="Email" v-model="signUpData.email" />
    <input type="password" placeholder="Password" v-model="signUpData.password" />
    <input type="text" placeholder="Nickname" v-model="signUpData.nickname" />
    <button type="button" @click="signUp">Sign Up</button>
    <div>Uid: {{ signUpUid }}</div>
    <div v-if="signUpErrMsg != ''">{{ signUpErrMsg }}</div>
  </div>
  <br />
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'

const apiUrl = 'https://todolist-api.hexschool.io'
const signUpData = ref({
  email: 'terryd@kali.org',
  password: 'terryd123',
  nickname: 'Terry.D'
})
const signUpErrMsg = ref('')
const signUpUid = ref('')
const signUp = () => {
  axios
    .post(apiUrl + '/users/sign_up', signUpData.value)
    .then((resp) => {
      signUpErrMsg.value = ''
      signUpUid.value = resp.data.uid
    })
    .catch((error) => {
      signUpErrMsg.value = error.message
      signUpUid.value = ''
    })
}
</script>

<style></style>
