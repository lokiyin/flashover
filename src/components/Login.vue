<template>
  <div class="container container-table col-xs-8 col-xs-offset-2 col-lg-4 col-lg-offset-4">
      <div class="row vertical-40p ">
        <div class="container-fluid ">
          <div class="text-center ">
            <!-- login form -->
            <form class="ui form loginForm"  @submit.prevent="checkCreds">

              <div class="input-group">
                <span class="input-group-addon"><i class="fa fa-envelope"></i></span>
                <input class="form-control" name="username" placeholder="Nombre de Usuario" type="text" v-model="username">
              </div>

              <div class="input-group">
                <span class="input-group-addon"><i class="fa fa-lock"></i></span>
                <input class="form-control" name="password" placeholder="Clave" type="password" v-model="password">
              </div>
              <button type="submit" v-bind:class="'btn btn-primary btn-lg ' + loading">Entrar</button>
            </form>

            <!-- errors -->
            <div v-if=response class="text-red"><p>{{response}}</p></div>
          </div>
        </div>
      </div>
  </div>
</template>

<script>
import api from '../api'

export default {
  name: 'Login',
  data (router) {
    return {
      section: 'Login',
      loading: '',
      username: '',
      password: '',
      response: ''
    }
  },
  methods: {
    checkCreds () {
      const {username, password} = this

      this.toggleLoading()
      this.resetResponse()
      this.$store.commit('TOGGLE_LOADING')

      /* Making API call to authenticate a user */
      api.request('post', '/login', {username, password})
      .then(response => {
        this.toggleLoading()

        var data = response.data
        /* Checking if error object was returned from the server */
        if (data.error) {
          var errorName = data.error.name
          if (errorName) {
            this.response = errorName === 'InvalidCredentialsError'
            ? 'Username/Password incorrect. Please try again.'
            : errorName
          } else {
            this.response = data.error
          }

          return
        }

        /* Setting user in the state and caching record to the localStorage */
        if (data.user) {
          var token = 'Bearer ' + data.token

          this.$store.commit('SET_USER', data.user)
          this.$store.commit('SET_TOKEN', token)

          if (window.localStorage) {
            window.localStorage.setItem('user', JSON.stringify(data.user))
            window.localStorage.setItem('token', token)
          }

          this.$router.push(data.redirect)
        }
      })
      .catch(error => {
        this.$store.commit('TOGGLE_LOADING')
        console.log(error)

        this.response = 'Server appears to be offline'
        this.toggleLoading()
      })
    },
    toggleLoading () {
      this.loading = (this.loading === '') ? 'loading' : ''
    },
    resetResponse () {
      this.response = ''
    }
  }
}
</script>

<style>
html, body, .container-table {
  height: 100%;
  
}
body{
  background: url("https://images5.alphacoders.com/695/695931.jpg") no-repeat center center fixed ;
       -webkit-background-size:cover;
       -moz-background-size:cover;
       -o-background-size:cover;
       background-size:cover;
}

.container-table {
    display: table;
    color: white;
    background: rgba(255,255,255,0.75) !important ;
    margin-top: 10%;
    border-radius: 30px;
      -webkit-border-radius: 30px;
      -moz-border-radius: 30px;
}
.text-center{
  margin-bottom: 5%;
  margin-top: 10%;
}
.vertical-center-row {
    display: table-cell;
    vertical-align: middle;
}
.vertical-20p {
  padding-top: 20%;
}
.vertical-10p {
  padding-top: 10%;
}
.logo {
  width: 15em;
  padding: 3em;
}
.loginForm .input-group {
  padding-bottom: 1em;
  height: 4em;
}
.input-group input {
  height: 4em;
}
</style>
