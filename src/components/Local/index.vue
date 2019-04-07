<template>
  <div class="local-container poa">
    <el-input
      type="file"
      class="file-input"
    >
      选择本地文件
    </el-input>
  </div>
</template>

<script>
export default {

}
</script>

<style lang="scss" scoped>
.local-container {
  width: 100%;
  height: 100%;
  left: 0;
  top: 0;
  background-color: seashell;
  .file-input {
    position: absolute;
    max-width: 300px;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
  }
}
</style>
