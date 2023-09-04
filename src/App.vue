<template>
  <div id="app">
    <router-view/>
  </div>
</template>

<script>
export default {
  data() {
    return {

    }
  },
  methods: {

  },
  mounted() {
    let login_token = localStorage.getItem('user_login_token');
    if(login_token) {
      let decoded_login_token = atob(login_token)
      console.log(decoded_login_token)
      // this.$router.push('/')
    }
  }
}
</script>

<style lang="scss">
#app {
  background: #fff;
  height: 100%;
}
body {
  font-family: Poppins, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #171725;
  box-sizing: border-box;
}
html, body {
  width: 100%;
  height: 100%;
  margin: 0;
  overflow: hidden;
}
.text-center {
  text-align: center;
}
.w-100 {
  width: 100%;
}
.align-self-end {
  align-self: flex-end;
}
</style>
