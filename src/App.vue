<template>
  <div id="app">
    <div class="header-title header-color"><img src="/logo.png" alt="" height="100"/></div>
    <div></div>
    
    <div id="nav">
      <router-link to="/">搜索图片</router-link> |
      <router-link to="/about">我的收藏</router-link>
    </div>
    <router-view/>
  </div>
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

#nav {
  padding: 20px;
  font-size: 20px;
}

#nav a {
  font-weight: bold;
  color: #2c3e50;
}

#nav a.router-link-exact-active {
  color: #42b983;
}

.header-title {
  font-size: 48px;
  font-weight: bold;
  font-family: 'Helvetica Neue', sans-serif;
  background-image: linear-gradient(to right, #ff0037, #ff7034, #ffcd32, #9fff38, #28fffb, #7d5fff, #ff28a9);
  mix-blend-mode: overlay;
  opacity: 0.7;
}
</style>
