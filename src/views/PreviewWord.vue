<template>
  <div className="my-component" ref="preview">
    <input type="file" @change="preview" ref="file">
  </div>
</template>
<script>
const docx = require('docx-preview');
window.JSZip = require('jszip')
export default {
  methods: {
    preview(e) {
      docx.renderAsync(this.$refs.file.files[0], this.$refs.preview) // 渲染到页面预览
    }
  }
};
</script>
<style lang="less" scoped>
.my-component {
  width: 100%;
  height: 90vh;
  border: 1px solid #000;
}
</style>
