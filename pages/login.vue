<template>
  <div>
    <button v-if="!loggedIn" @click="login">Login</button>
    <button v-else @click="logout">logout</button>
    <h1 v-if="userName">{{ userName }}</h1>
  </div>
</template>
<script>
import axios from 'axios'
axios.defaults.withCredentials = true

export default {
  data() {
    return {
      email: 'admin@admin.com',
      password: 'password',
      baseURL: 'http://localhost:8000/',
      loggedIn: localStorage.getItem('me'),
      userName: localStorage.getItem('me')
        ? JSON.parse(localStorage.getItem('me')).data.name
        : 'Guest User',
    }
  },
  mounted() {
    console.log(this.loggedIn)
  },
  methods: {
    laravelCsrfCoockie() {
      axios.get(this.baseURL + 'sanctum/csrf-cookie').then((response) => {
        return true
      })
    },
    login() {
      this.laravelCsrfCoockie()
      axios
        .post(this.baseURL + 'api/login', {
          email: this.email,
          password: this.password,
        })
        .then((res) => {
          localStorage.setItem('me', JSON.stringify(res.data))
          this.loggedIn = localStorage.getItem('me')
          this.userName = JSON.parse(localStorage.getItem('me')).data.name
          console.log(this.loggedIn)
        })
        .catch((err) => {
          console.log(err)
        })
    },
    logout() {
      this.laravelCsrfCoockie()
      axios
        .post(this.baseURL + 'api/logout')
        .then((res) => {
          localStorage.removeItem('me')
          this.loggedIn = null
          this.userName = 'Guest User'
          console.log(this.loggedIn)
        })
        .catch((err) => {
          console.log(err)
        })
    },
  },
}
</script>
