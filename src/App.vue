<template>
  <div id="app">
    <div class="container">
      <div class="left">
        <el-tree :data="data" :props="defaultProps" @node-click="handleNodeClick" accordion></el-tree>
      </div>
      <pre class="right" v-html="html" />
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      data: [],
      defaultProps: {
        children: 'children',
        label: 'label'
      },
      html: '',
      api: 'http://118.24.168.221:9998'
    }
  },
  mounted() {
    this.$axios({
      method:'get',
      url: this.api
    }).then((response) =>{
      if (response && response.data.length > 0) {
        const arr = []
        response.data.forEach((item) => {
          if (!item.includes('/')) {
            arr.push({
              label: item,
              src: item
            })
          } else {
            const folder = item.split('/')[0]
            const obj = arr.find((a) => folder === a.label)
            if (obj) {
              if (!obj.children) {
                obj.children = []
              }
              obj.children.push({
                src: item,
                label: item.replace(obj.label + '/', '')
              })
            }
          }
        })
        arr.forEach(a => {
          if (a.children) {
            a.children.sort((x, y) => y.label.replace(/.log/g, '').replace(/-/g, '') - x.label.replace(/.log/g, '').replace(/-/g, ''))
          }
        })
        this.data = arr
      }
    }).catch((error) => {
      console.log(error)
    })
  },
  methods: {
    handleNodeClick(data) {
      document.documentElement.scrollTop = 0
      if (!data.children) {
        this.html = ''
        this.$axios({
          method:'get',
          url: this.api + '/log',
          params: {
            src: data.src
          }
        })
        .then(response => {
          this.html = response.data
        }).catch((error) => {
          console.log(error)
        })
      }
    }
  }
}
</script>

<style>
body {
  background-color: #f7f7f7;
}
#app {
  font-family: "Microsoft YaHei", Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  display: flex;
  justify-content: center;
}
.container {
  width: 100%;
  display: flex;
}
.left {
  width: 250px;
  padding: 15px;
  background-color: #fff;
  min-height: 100ch;
  flex-shrink: 0;
}
.right {
  padding: 15px;
  width: 100%;
  margin-left: 10px;
  background-color: #fff;
  word-break: break-all;
  white-space: pre-line;
}
pre {
  margin: 0;
  font-family: "Microsoft YaHei", Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
