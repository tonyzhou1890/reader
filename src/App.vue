<template>
  <div id="app por">
    <tu-book
      :text="message"
      :width="width"
      :height="height"
      :percent="defaultPercent"
      :single="single"
      @pageChange="pageChange"
    />
    <div
      @click="toggleSetting"
      class="middle-level poa">
    </div>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'App',
  data() {
    return {
      width: null,
      height: null,
      message: 'reader',
      total: null,
      page: null,
      percent: null,
      defaultPercent: null
    }
  },
  computed: {
    single() {
      let temp = (this.width && this.height) || false
      temp = this.width < this.height
      return temp
    }
  },
  created() {
    this.settingSize()
    this.watchClientSize()
    this.resolveUrl()
  },
  methods: {
    watchClientSize() {
      window.addEventListener('resize', this._.debounce(this.settingSize, 1000))
    },
    settingSize() {
      const s = window.getComputedStyle(document.body)
      this.defaultPercent = this.percent
      this.width = Number(s.width.split('px')[0])
      this.height = Number(s.height.split('px')[0])
    },
    pageChange(data) {
      this.total = data.total
      this.page = data.page
      this.percent = data.percent
    },
    resolveUrl() {
      const temp = window.location.search.slice(1).split('&')
      temp.map((item, index) => {
        temp[index] = item.split('=')
      })
      // 是否通过address指定文本地址
      const address = temp.filter(item => { return item[0] === 'address' })
      if (address.length) {
        this.getAddressTxt(address[0][1])
      }
    },
    getAddressTxt(url) {
      axios.get(url)
        .then(res => {
          this.message = res.data
        })
        .catch(e => {
          this.$message({
            message: '地址错误',
            type: 'error'
          })
        })
    },
    toggleSetting(e) {
      console.log(e)
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
  .middle-level {
    top: 0;
    left: 33.333%;
    width: 33.333%;
    height: 100%;
  }
}
</style>
