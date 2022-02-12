<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png">
    <input @change="fileChange" type="file">
    <img :src="base64String" alt="" style="width: 400px;">
    <p>123</p>
  </div>
</template>

<script>
import axios from 'axios'
import SparkMD5 from 'spark-md5'

export default {
  name: 'Home',
  components: {
    // HelloWorld
  },
  data () {
    return {
      base64String: ''
    }
  },
  created () {

  },
  methods: {
    async fileChange (e) {
      const _this = this
      const files = e.target.files
      const file = files[0]
      const size = file.size
      // const blob = file.slice(0, size)
      const blob1 = file.slice(0, 8000000)
      const blob2 = file.slice(5000000, size)
      console.log('file', file, blob1, blob2)

      // 读取本地文件，以gbk编码方式输出
      if (file) {
        // 文件上传
        const promises = []
        const hash = await this.hashFile(file)
        const { list, chunks, fileSize } = this._fileSlice(file, 2)
        console.log('fileLIst:::', list, file)
        for (let i = 0; i < list.length; i++) {
          const formData = new FormData() // 创建formData对象
          formData.append('hash', hash)
          formData.append('chunks', chunks)
          formData.append('index', i)
          formData.append('fileSize', fileSize)
          formData.append('file', list[i]) // 通过append向form对象添加数据
          console.log('formData', formData, hash)

          const config = {
            headers: { 'Content-Type': 'multipart/form-data' }
          }
          console.log(7777777, file, config)
          promises.push(axios.post('http://localhost:7001/api/v1/file/slice', formData, config))
        }

        console.log('promises', promises)
        axios.all(promises)
          .then(axios.spread(function (acct, perms) {
            // 两个请求现在都执行完成
            console.log('=========', acct, perms)
          }))

        const reader = new FileReader()
        reader.readAsArrayBuffer(file)
        reader.onload = function () {
          console.log('readAsArrayBuffer', this.result)
          console.log(new Blob([this.result]))
        }
      }

      const myfile = new window.File(
        [blob1],
        'logo.png',
        { type: 'image/png' }
      )
      // console.log('myfile', myfile)

      var reader = new FileReader()
      reader.readAsDataURL(myfile)
      reader.onloadend = function (e) {
        _this.base64String = e.target.result
        // console.log('base64String', _this.base64String)
      }
    },

    /**
     * 文件切片
     * file：文件，chunkSize：切片大小
     * 返回值，list：切片列表，chunks：切片数量，fileSize：文件大小
     */
    _fileSlice (file, chunkSize) {
      const list = []
      chunkSize = chunkSize * 1024 * 1024
      const fileSize = file.size // 文件大小
      const chunks = Math.ceil(fileSize / chunkSize) // 切片数量

      for (let i = 0; i < chunks; i++) {
        const start = i * chunkSize
        const end = (i + 1) * chunkSize > file.size ? file.size : (i + 1) * chunkSize
        const blob = file.slice(start, end)
        // list.push(blob)
        list.push(new window.File(
          [blob],
          `file_${i}.png`,
          { type: 'image/png' }
        ))
      }
      return { list, chunks, fileSize }
    },

    // 获取文件hash值
    hashFile (file) {
      const chunkSize = 2 * 1024 * 1024
      const blobSlice = File.prototype.slice || File.prototype.mozSlice || File.prototype.webkitSlice
      return new Promise((resolve, reject) => {
        const chunks = Math.ceil(file.size / chunkSize)
        let currentChunk = 0
        const spark = new SparkMD5.ArrayBuffer()
        const fileReader = new FileReader()
        function loadNext () {
          const start = currentChunk * chunkSize
          const end = start + chunkSize >= file.size ? file.size : start + chunkSize
          fileReader.readAsArrayBuffer(blobSlice.call(file, start, end))
        }
        fileReader.onload = e => {
          spark.append(e.target.result) // Append array buffer
          currentChunk += 1
          if (currentChunk < chunks) {
            loadNext()
          } else {
            console.log('finished loading')
            const result = spark.end()
            // 如果单纯的使用result 作为hash值的时候, 如果文件内容相同，而名称不同的时候
            // 想保留两个文件无法保留。所以把文件名称加上。
            const sparkMd5 = new SparkMD5()
            sparkMd5.append(result)
            sparkMd5.append(file.name)
            const hexHash = sparkMd5.end()
            resolve(hexHash)
          }
        }
        fileReader.onerror = () => {
          console.warn('文件读取失败！')
        }
        loadNext()
      }).catch(err => {
        console.log(err)
      })
    }
  }
}
</script>

<style lang="scss" scoped>

</style>
