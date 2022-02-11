<template>
  <div class="home">
    <img @click="createCanvas" alt="Vue logo" src="../assets/logo.png">
    <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
    <div id="test">
      <span>Hello World</span>
    </div>
    <div @click="openWindow">onpen about window</div>
    <div @click="postMessageToAbout">postMessage to about window</div>
    <div @click="closeAboutWindow">close about window</div>
    <input @change="fileChange" type="file">
    <img :src="base64String" alt="" style="width: 400px;">

    <Button type="primary">
      Primary
    </Button>
    <Button type="danger">
      Danger
    </Button>
  </div>
</template>

<script>
// @ is an alias to /src
// import HelloWorld from '@/components/HelloWorld.vue'
import html2canvas from 'html2canvas'
import { jsPDF as JsPDF } from 'jspdf'
import { Button } from 'ant-design-vue'

export default {
  name: 'Home',
  components: {
    Button
    // HelloWorld
  },
  data () {
    return {
      domain: 'http://localhost:8080',
      aboutWindow: null,
      base64String: ''
    }
  },
  created () {
    window.addEventListener('message', function (event) {
      console.log('接收about window数据：', event.data)
    }, false)

    // 订阅localstorage
    window.addEventListener('storage', (e) => {
      if (e.key === 'token') console.log('token改变了', e.newValue)
    })
  },
  methods: {
    fileChange (e) {
      const _this = this
      const files = e.target.files
      const file = files[0]
      const size = file.size
      // const blob = file.slice(0, size)
      const blob1 = file.slice(0, 8000000)
      const blob2 = file.slice(5000000, size)
      console.log('file', file, blob1, blob2)

      const myfile = new window.File(
        [blob1],
        'myfile',
        { type: 'image/jpeg' }
      )
      console.log('myfile', myfile)

      var reader = new FileReader()
      reader.readAsDataURL(myfile)
      reader.onloadend = function (e) {
        _this.base64String = e.target.result
        console.log('base64String', _this.base64String)
      }
    },
    openWindow () {
      localStorage.setItem('testData', '{ name: "guang" }')
      console.log('open window')
      this.aboutWindow = window.open(this.domain + '/about', 'aboutPage')
      // this.aboutWindow.postMessage('第一次发送信息给about window', this.domain)
    },
    postMessageToAbout () {
      console.log('postMessageToAbout')
      this.aboutWindow.postMessage('hello about window', this.domain)
    },
    closeAboutWindow () {
      this.aboutWindow.close()
    },
    createCanvas () {
      const _this = this
      html2canvas(document.getElementById('test')).then(function (canvas) {
        const canvasData = canvas.toDataURL('image/jpeg', 1.0)
        _this.createPDF(canvasData)
        document.body.appendChild(canvas)
      })
    },
    createPDF (data) {
      const doc = new JsPDF()

      doc.addImage(data, 'JPEG', 0, 10, 100, 100)
      doc.addImage(data, 'JPEG', 0, 120, 100, 100)
      doc.addImage(data, 'JPEG', 0, 230, 100, 100)
      doc.addImage(data, 'JPEG', 0, 340, 100, 100)
      doc.addImage(data, 'JPEG', 0, 450, 100, 100)
      doc.addPage()
      doc.addImage(data, 'JPEG', 0, 0, 100, 100)
      doc.save('a4.pdf')
    }
  }
}
</script>

<style lang="scss" scoped>
#test {
  width: 100px;
  height: 100px;
  background: #f8f8;
  span {
    display: block;
    line-height: 100px;
    text-align: center;
  }
}
</style>
