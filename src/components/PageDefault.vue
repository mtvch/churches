<template>
  <el-row>
    <el-col :offset="6" :span="12" justify="center">
      <h2>Find a church anywhere on the globe!</h2>
      <ol>
        <li>Enter place where you want to find a church</li>
        <li>Get a list of churches around</li>
        <li>And weather!</li>
        <li>Read description</li>
      </ol>
    </el-col>
  </el-row>
</template>

<script>
export default {

};
</script>

<style lang="scss">
h2 {
  text-align: center;
  font-weight: 500;
  padding: 20px;
}
ol {
  line-height: 2;
}

ol li {
  list-style-type: none;
  counter-increment: item;
}

ol li:before {
  content: counter(item);
  display: inline-block;
  width: 12px;
  height: 20px;
  margin-right: 5px;
  color: #606266;
  font-weight: bold;
  font-size: 140%;
  padding: 0 8px 8px;
  border-radius: 3px;
  line-height: 1.3;
}
</style>