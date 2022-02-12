<template>
  <div id="app">
    <div id="nav">
      <router-link to="/">Home</router-link> |
      <router-link to="/about">About</router-link> |
      <router-link to="/fileUpload">FileUpload</router-link> |
      <router-link to="/fileSliceUpload">FileSliceUpload</router-link>
      <Button @click="toAboutPage" type>to about page</Button>
    </div>
    <p>qiankun-micro-vue1</p>
    <router-view/>
  </div>
</template>

<script>
import { Button } from 'ant-design-vue'

export default {
  name: 'Home',
  components: {
    Button
    // HelloWorld
  },
  methods: {
    toAboutPage () {
      window.history.pushState({
        pageName: 'about'
      }, '', '/system/about')
    }
  }
}
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

#nav {
  padding: 30px;

  a {
    font-weight: bold;
    color: #2c3e50;

    &.router-link-exact-active {
      color: #42b983;
    }
  }
}
</style>
