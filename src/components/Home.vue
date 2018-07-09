<template>
  <div id="home">
    <img class="mv" src="../assets/md.svg">
    <h1>{{ msg }}</h1>
    <button @click="googleLogin">Google アカウントでログイン</button>
  </div>
</template>

<script>
  export default {
    name: 'home',
    data () {
      return {
        msg: 'Simple Markdown'
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
  @import '../scss/variable';


  button {
    font-size: 16px;
    font-weight: 600;
    margin-top: 40px;
    padding: 10px 20px;
    transition: all .2s;
    color: white;
    background-color: $color-blue;

    &:hover {
      background-color: darken($color-blue, 4%);
    }
  }

  .mv {
    width: 16vw;
  }
</style>
