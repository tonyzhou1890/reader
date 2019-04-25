<template>
  <div id="app" class="por" v-loading="loading">
    <tu-book
      :text="message"
      :width="width"
      :height="height"
      :color="setting.color"
      :background="setting.background"
      :pagePadding="setting.padding"
      :fontFamily="setting.fontFamily"
      :fontSize="setting.fontSize"
      :lineHeight="setting.lineHeight"
      :percent="defaultPercent"
      :single="single"
      @pageChange="pageChange"
      @mousedown.native="middleClick"
      @mouseup.native="middleClick"
      ref="book"
      :style="`z-index: 1;`"
    />
    <div
      v-show="showSetting"
      :style="`z-index: 2`"
      @click="toggleSetting"
      class="middle-cover poa"></div>
    <setting
      v-show="showSetting"
      :style="`z-index: 3;`"
      :position="position"
      :setting="setting"
      :defaultSetting="defaultSetting"
      :page="page"
      :total="total"
      @dataChange="settingChange"
      @pageChange="changePage"
    />
    <local
      v-if="showLocal"
      :style="`z-index: 4;`"
      @get-data="getLocalData"
    />
  </div>
</template>

<script>
import axios from 'axios'
import Setting from './components/Setting'
import Local from './components/Local'
import { storage } from '@/utils/storage'
const defaultSetting = {
  fontFamily: 'Microsoft YaHei',
  fontSize: 16,
  lineHeight: 24,
  padding: 25,
  color: '#333',
  background: '#fff5ee'
}
export default {
  name: 'App',
  components: {
    Setting,
    Local
  },
  data() {
    return {
      width: null,
      height: null,
      message: 'reader',
      total: null,
      page: null,
      defaultPercent: null,
      showSetting: false,
      defaultSetting: Object.assign({}, defaultSetting),
      setting: Object.assign({}, defaultSetting),
      mouseEvent: {
        down: null,
        lastSelectionType: ''
      },
      address: null,
      loading: false,
      local: null,
      showLocal: false
    }
  },
  computed: {
    single() {
      let temp = (this.width && this.height) || false
      temp = this.width < this.height
      return temp
    },
    position() {
      return this.width > 500 ? 'top' : 'bottom'
    }
  },
  created() {
    this.settingSize()
    this.watchClientSize()
    this.resolveUrl()
    this.getSetting()
  },
  methods: {
    // 监听窗口尺寸变化
    watchClientSize() {
      window.addEventListener('resize', this._.debounce(this.settingSize, 1000))
    },
    // 设定book组件尺寸
    settingSize() {
      const s = window.getComputedStyle(document.body)
      this.width = Number(s.width.split('px')[0])
      this.height = Number(s.height.split('px')[0])
      this.changePercent()
    },
    // 页码变化处理函数
    pageChange(data) {
      this.total = data.total
      this.page = data.page
      if (this.address || this.local) {
        this.setProcess(this.address || this.local, this.page / this.total * 100)
      }
    },
    // 判定是哪种方式获取文本
    resolveUrl() {
      const temp = window.location.search.slice(1).split('&')
      temp.map((item, index) => {
        temp[index] = item.split('=')
      })
      // 是否通过address指定文本地址
      const address = temp.filter(item => { return item[0] === 'address' })
      if (address.length) {
        this.getAddressTxt(address[0][1])
        return
      }
      // 是否通过message通信
      const message = temp.filter(item => item[0] === 'message')
      if (message.length) {
        console.log(message)
        return
      }
      // 其余默认local
      this.showLocal = true
    },
    // url指定地址需要异步获取文本
    getAddressTxt(url) {
      this.loading = true
      axios.get(url)
        .then(res => {
          this.message = res.data
          this.address = url
          this.getProcess(this.address)
          this.loading = false
        })
        .catch(e => {
          this.$message({
            message: '地址错误',
            type: 'error'
          })
          this.loading = false
        })
    },
    // 点击阅读器1/3 - 2/3区域
    middleClick(e) {
      if (e.type === 'mousedown') {
        this.mouseEvent.down = e
        const temp = document.getSelection().type
        this.mouseEvent.lastSelectionType = temp
        return
      }
      if (e.type === 'mouseup' &&
        e.which === 1 &&
        this.mouseEvent.down &&
        Math.abs(e.clientX - this.mouseEvent.down.clientX) < 1 &&
        Math.abs(e.clientY - this.mouseEvent.down.clientY) < 1 &&
        this.mouseEvent.lastSelectionType !== 'Range') {
        const rect = this.$refs.book.$el.getBoundingClientRect()
        const left = e.clientX - rect.left
        if (left > rect.width / 3 && left < rect.width / 3 * 2) {
          this.toggleSetting()
        }
      }
      this.mouseEvent.down = null
    },
    // 设置面板显示隐藏
    toggleSetting() {
      this.showSetting = !this.showSetting
    },
    // 应用设置
    settingChange(data) {
      this.setting = Object.assign(this.setting, data)
      this.changePercent()
      this.toggleSetting()
      this.storeSetting()
    },
    // 跳页
    changePage(val) {
      this.page = val
      this.changePercent()
      this.toggleSetting()
    },
    // 改变默认阅读进度
    changePercent() {
      this.defaultPercent = this.page / this.total * 100
    },
    // 将设置存储到localStorage
    storeSetting() {
      storage('setting', this.setting)
        .then(res => {})
        .catch(e => {
          const m = JSON.parse(e.message)
          this.$message({
            type: 'error',
            message: m.message
          })
        })
    },
    // 从localStorage获取设置
    getSetting() {
      if (!window.localStorage) {
        this.$message({
          type: 'tip',
          message: '此浏览器不支持本地存储，除非登录，否则您的设置和阅读进度将不会存储。'
        })
        return
      }
      const s = window.localStorage.getItem('setting')
      if (!s) {
      } else {
        this.setting = Object.assign(this.setting, JSON.parse(s))
      }
    },
    // 将进度存储到localStorage
    setProcess(key, val) {
      storage(key, val)
        .then(res => {})
        .catch(e => {
          const m = JSON.parse(e)
          this.$message({
            type: 'error',
            message: m.message
          })
        })
    },
    // 从localStorage读取进度
    getProcess(key) {
      if (!window.localStorage || window.localStorage.getItem(key) === null) {
        return
      }
      this.defaultPercent = Number(window.localStorage.getItem(key))
    },
    // 从本地读取文件处理函数
    getLocalData(e) {
      this.message = e.value
      this.local = e.key
      this.getProcess(this.local)
      this.showLocal = false
    }
  }
}
</script>

<style lang='scss' scoped>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  .middle-cover {
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
  }
}
</style>
