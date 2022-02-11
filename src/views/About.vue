<template>
  <div class="about">
    <h1>This is an about page</h1>
    <div @click="postMessageToHome">postMessage to home</div>
    <div @click="setStorage">set storage</div>
  </div>
</template>

<script>
const domain = 'http://localhost:8080'
// let homeWindow
export default {
  created () {
    // window.alert(localStorage.getItem('testData'))
    window.addEventListener('message', function (event) {
      // homeWindow = event.source
      console.log('接收home window的消息:', event.data, event.source)
    }, false)
  },
  beforeDestroy () {
    window.opener.postMessage('hello there!', domain)
  },
  methods: {
    postMessageToHome () {
      console.log('发送消息给home')
      window.opener.postMessage('Hello Home window', domain)
    },
    setStorage () {
      localStorage.setItem('token', +new Date())
      console.log(localStorage.getItem('token'))
      console.log('test')
    }
  }
}
</script>
