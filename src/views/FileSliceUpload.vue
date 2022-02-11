<template>
  <div class="file-slice-upload">
    <h1>文件分片上传</h1>
    <Upload
      action="http://localhost:7001/api/v1/file/slice"
      :multiple="false"
      :beforeUpload="beforeUpload"
      :customRequest="customRequest"
      @change="handleChange"
    >
      <Button type="primary"> <Icon type="upload" />上传</Button>
    </Upload>
  </div>
</template>

<script>
import axios from 'axios'
import SparkMD5 from 'spark-md5'
import { Button, Upload, Icon } from 'ant-design-vue'

const token = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2NDMyMTMyNjAsInVzZXJOYW1lIjoiZ3VhbmciLCJ1dWlkIjoiMTIzIiwiaWF0IjoxNjQzMjA2MDYwfQ.gCqQbdCkwTqhRfoFtew5SAnREju7VIL-kmJWNxEgFf0'

export default {
  name: 'FileSliceUpload',
  components: {
    Button,
    Upload,
    Icon
  },
  data () {
    return {
      fileInfo: null
    }
  },
  created () {

  },
  methods: {
    handleChange (e) {
      console.log('handleChange', e)
      this.fileInfo = e.file
    },
    async beforeUpload (file) {
      return { name: 'gaung' }
    },
    customRequest (e) {
      this.fileSliceUpload(e.file).then(res => {
        console.log('fileSliceUpload', res)
        this.fileInfo.percent = 100
        this.fileInfo.status = 'done'
        return true
      })
    },

    // 文件切片上传
    fileSliceUpload (file) {
      // eslint-disable-next-line no-async-promise-executor
      return new Promise(async (resolve, reject) => {
        // 获取文件hash值
        const hash = await this.hashFile(file)
        const { list, chunks, fileSize } = this._fileSlice(file, 1)

        // 为每块切片添加文件信息
        const fileList = list.map((item, index) => {
          const formData = new FormData() // 创建formData对象
          formData.append('hash', hash)
          formData.append('chunks', chunks)
          formData.append('index', index)
          formData.append('fileSize', fileSize)
          formData.append('file', item) // 通过append向form对象添加数据
          return formData
        })

        const tempArr = fileList.map(item => {
          const config = {
            headers: {
              'Content-Type': 'multipart/form-data',
              Authorization: 'Bearer ' + token
            }
          }
          return axios.post('http://localhost:7001/api/v1/file/slice', item, config)
        })
        // console.log('tempArr', tempArr)
        // 循环上传切片
        // const promiseList = []
        const _this = this
        function batchUpload (promises) {
          Promise.allSettled(promises).then(res => {
            _this.fileInfo.percent += (promises.length / chunks) * 100
            console.log('promises', _this.fileInfo.percent)
            if (tempArr.length) {
              batchUpload(tempArr.splice(0, 2))
            } else {
              _this.mergeFile({ hash, chunks })
              resolve(true)
            }
          })
        }
        batchUpload(tempArr.splice(0, 2))
      })
    },

    // 合并文件请求
    async mergeFile ({ hash, chunks }) {
      const headers = {
        'Content-Type': 'application/json; charset=utf-8',
        Authorization: 'Bearer ' + token
      }
      const res = await axios({
        method: 'post',
        url: 'http://localhost:7001/api/v1/file/merge',
        headers,
        data: { hash, chunks }
      })
      console.log('mergeFile', res)
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
      const filePath = file.name
      const index = filePath.lastIndexOf('.')
      // const fileName = filePath.slice(0, index)
      // 获取文件后缀
      const ext = filePath.slice(index)
      const type = file.type

      for (let i = 0; i < chunks; i++) {
        const start = i * chunkSize
        const end = (i + 1) * chunkSize > file.size ? file.size : (i + 1) * chunkSize
        const blob = file.slice(start, end)

        list.push(new window.File(
          [blob],
          `temp_${i}${ext}`,
          { type }
        ))
        // console.log('fileslice', list)
      }
      return { list, chunks, fileSize, fileType: type }
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
