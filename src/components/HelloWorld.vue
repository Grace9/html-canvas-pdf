<template>
  <div class="hello">
    <img src="../assets/images/timg.jpg">
    <h1>{{ msg }}</h1>
    <h2>dom转canvas（生成pdf）</h2>
    <ul>
      <li @click="goTofile()">
        Let's go!
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Welcome to Html To Canvas'
    }
  },
  methods:{
    goTofile(){
      this.$router.push({ name: 'File', params: {}})
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
  color: #42b983;
  cursor: pointer;
}
img{
  width: 200px;
  height: 200px;
}
</style>
