<template>
  <div id="app">
    <div class="container">
      <div class="left">
        <el-tree :data="data" :props="defaultProps" @node-click="handleNodeClick"></el-tree>
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
              src: item,
              children: []
            })
          } else {
            const obj = arr.find((a) => item.includes(a.label))
            if (obj) {
              obj.children.push({
                src: item,
                label: item.replace(obj.label + '/', '')
              })
            }
          }
        })
        arr.forEach(a => {
          a.children.sort((x, y) => {y.label - x.label})
        })
        this.data = arr
      }
    }).catch((error) => {
      console.log(error)
    })
  },
  methods: {
    handleNodeClick(data) {
      if (!data.children) {
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
  width: 1000px;
  display: flex;
}
.left {
  width: 200px;
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
}
pre {
  margin: 0;
  font-family: "Microsoft YaHei", Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
