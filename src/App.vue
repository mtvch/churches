<template>
  <el-container>
    <el-header id="header">
      <el-affix>
      <page-top
        @select="setPlace"
      />
      </el-affix>
    </el-header>
    <el-body id="body">
      <page-main
        v-if="place"
        :placeGeo="place"
      />
      <page-default
        v-else
      />
        <el-image :src="churchImg" />
    </el-body>
    <el-footer id="footer">
    </el-footer>
  </el-container>
</template>

<script>
import PageTop from './components/PageTop.vue'
import PageDefault from './components/PageDefault.vue'
import PageMain from './components/PageMain.vue'

const churchImg = require("@/assets/church.png");

export default {
  components: { PageTop, PageDefault, PageMain },
  name: 'App',
  data() {
    return {
      churchImg,
      place: null
    }
  },
  methods: {
    setPlace(value) {
      if (value.point) {
        this.place = value;
      } 
    }
  }
}
</script>

<style lang="scss">
.el-image {
  position: absolute;
  bottom: 0px;
  left: 60%;
  width: 40%;
  display: block;
  margin-left: auto;
}

  * {
    margin: 0;
    padding: 0;
    box-sizing: content-box;
  }
  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@500&family=Nunito&display=swap');

  #body {
    min-height: 100vh;
    position: relative;
  }

  #footer {
    background-color: #E4E7ED;
  }

</style>
