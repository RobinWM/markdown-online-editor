<!-- @format -->

<template>
  <div class="index-page" v-loading="isLoading">
    <HeaderNav />
    <div id="vditor" class="vditor" />
  </div>
</template>

<script>
import Vditor from 'vditor'
import HeaderNav from './partials/HeaderNav'
import defaultText from '@config/default'

export default {
  name: 'index-page',

  data() {
    return {
      isLoading: true,
      isMobile: window.innerWidth <= 960,
      vditor: null,
    }
  },

  created() {
    this.setDefaultText()
    // console.log = () => {}
  },

  components: {
    HeaderNav,
  },

  mounted() {
    this.initVditor()
    this.$nextTick(() => {
      this.isLoading = false
    })
    this.$root.$on('reload-content', this.reloadContent)
  },

  beforeDestroy() {
    this.$root.$off('reload-content', this.reloadContent)
  },

  methods: {
    initVditor() {
      const that = this
      const options = {
        width: this.isMobile ? '100%' : '80%',
        height: '0',
        tab: '\t',
        counter: '999999',
        typewriterMode: true,
        mode: 'sv',
        preview: {
          delay: 100,
          show: !this.isMobile,
        },
        outline: true,
        upload: {
          max: 5 * 1024 * 1024,
          handler(file) {
            console.log('file', file)
            let formData = new FormData()
            formData.append('file', file[0])
            formData.append(
              'key',
              `ship-more/blog/${new Date()
                .toISOString()
                .slice(0, 7)
                .replace(/-/g, '/')}/${new Date().getTime()}.${file[0].name.split('.').pop()}`
            )
            let request = new XMLHttpRequest()
            request.open('POST', `https://cf-api.allinaigc.org/cf-r2-upload`)
            request.onload = that.onloadCallback
            request.send(formData)
          },
        },
        after: () => {
          const content = localStorage.getItem('vditorvditor') || defaultText
          this.vditor.setValue(content)
          this.vditor.focus()
        },
      }
      this.vditor = new Vditor('vditor', options)
    },
    onloadCallback(oEvent) {
      const currentTarget = oEvent.currentTarget
      if (currentTarget.status !== 200) {
        return this.$message({
          type: 'error',
          message: currentTarget.status + ' ' + currentTarget.statusText,
        })
      }
      let resp = JSON.parse(currentTarget.response)
      console.log('resp', resp)
      let imgMdStr = ''
      if (resp.code === 'invalid_source') {
        return this.$message({
          type: 'error',
          message: resp.message,
        })
      }
      if (resp.code === 'image_repeated') {
        imgMdStr = `![](${resp.images})`
      } else if (resp.code === 'success' || resp.success) {
        imgMdStr = `![${resp.data.filename}](${resp.data.url})`
      }
      this.vditor.insertValue(imgMdStr)
    },
    setDefaultText() {
      const savedMdContent = localStorage.getItem('vditorvditor') || ''
      if (!savedMdContent.trim()) {
        localStorage.setItem('vditorvditor', defaultText)
      }
    },
    reloadContent() {
      if (this.vditor && this.vditor.getValue) {
        const content = localStorage.getItem('vditorvditor') || ''
        this.vditor.setValue(content)
        this.vditor.focus()
      }
    },
  },
}
</script>

<style lang="less">
@import './../assets/styles/style.less';

.index-page {
  width: 100%;
  height: 100%;
  background-color: @white;
  .flex-box-center(column);

  .vditor {
    position: absolute;
    top: @header-height;
    max-width: @max-body-width;
    width: 80%;
    height: calc(100vh - 100px);
    margin: 20px auto;
    text-align: left;

    .vditor-toolbar {
      border-left: 1px solid #d1d5da;
      border-right: 1px solid #d1d5da;
    }

    .vditor-content {
      height: auto;
      min-height: auto;
      border: 1px solid #d1d5da;
      border-top: none;
    }
  }

  .vditor-reset {
    font-size: 14px;
  }

  .vditor-textarea {
    font-size: 14px;
    height: 100% !important;
  }
}

@media (max-width: 960px) {
  .index-page {
    .vditor {
      height: calc(100vh - 60px);
      padding: 10px;
      margin: 0px auto;
    }
  }
}
</style>
