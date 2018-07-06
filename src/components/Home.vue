<template>
  <div id="home">
    <h1>{{ msg }}</h1>
    <button @click="googleLogin">Google アカウントでログイン</button>
  </div>
</template>

<script>
  export default {
    name: 'home',
    data () {
      return {
        msg: 'Welcome to MyMarkdown'
      }
    },
    methods: {
      googleLogin: function() {
        firebase.auth().signInWithRedirect(new firebase.auth.GoogleAuthProvider());
      }
    }
  }
</script>

<style lang="scss" scoped>
  button {
    font-size: 16px;
    font-weight: 600;
    padding: 10px 20px;
  }
</style>

