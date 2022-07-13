<template>
  <div class="file">
    <div class="header">
      <v-btn @click="upload">上传文件</v-btn>
      <input ref="file" type="file" style="display:none" accept=".doc, .docx" @change="fileChange">
      <v-btn style="margin-left: 20px" @click="selectedExport()">导出doc</v-btn>

    </div>

    <v-data-table
        v-model="selected"
        :headers="headers"
        :items="desserts"
        :items-per-page="5"
        :single-select="singleSelect"
        item-key="name"
        show-select
        class="elevation-1"
    >
      <template v-slot:top>
        <v-switch v-model="singleSelect" label="Single select" class="pa-3"></v-switch>
      </template>

    </v-data-table>
    <div>
      <!--      预览-->
      <v-btn
          color="error"
          dark
          @click.stop="previewDoc"
      >
        预览
      </v-btn>
      <!--          dialog对话框-->
      <v-dialog
          v-model="dialog"
          width="90%"
      >
        <v-card>
          <v-card-text>
            <div className="my-component" ref="preview"></div>
          </v-card-text>
          <v-divider></v-divider>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn
                color="primary"
                text
                @click="dialog = false"
            >
              关闭
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

    </div>


  </div>
</template>

<script>
import JSZipUtils from "jszip-utils"
import docxtemplater from "docxtemplater"
import saveAs from 'file-saver'
import JSZip from 'jszip'
import PreviewWord from "@/views/PreviewWord";

const docx = require('docx-preview');
window.JSZip = require('jszip')

export default {
  name: "home",
  components: {
    PreviewWord
  },
  data: () => ({
    selectList: [],//选择需要导出的文档数据
    overlay: false,
    file: null,
    dialog: false,
    templateList: [],
    wordname: "",
    singleSelect: false,
    selected: [],
    headers: [
      {
        text: 'Dessert (100g serving)',
        align: 'start',
        sortable: false,
        value: 'name',
      },
      {text: 'Calories', value: 'calories'},
      {text: 'Fat (g)', value: 'fat'},
      {text: 'Carbs (g)', value: 'carbs'},
      {text: 'Protein (g)', value: 'protein'},
      {text: 'Iron (%)', value: 'iron'},
    ],
    desserts: [],
    fileDoc: null
  }),

  methods: {
    preview(blob) {
      docx.renderAsync(blob, this.$refs.preview) // 渲染到页面预览
    },
    modifyword(i) {
      let _this = this;
      // 读取并获得模板文件的二进制内容
      JSZipUtils.getBinaryContent("a1.docx", function (error, content) {
        // input.docx是模板。我们在导出的时候，会根据此模板来导出对应的数据
        // 抛出异常
        if (error) {
          throw error;
        }
        // 创建一个JSZip实例，内容为模板的内容
        let zip = new JSZip(content);
        // 创建并加载docxtemplater实例对象
        let doc = new docxtemplater().loadZip(zip);
        // 设置模板变量的值
        doc.setData({
          ..._this.selectList[i], //这里是上面form表单的内容
        });
        try {
          // 用模板变量的值替换所有模板变量
          doc.render();
        } catch (error) {
          // 抛出异常
          let e = {
            message: error.message,
            name: error.name,
            stack: error.stack,
            properties: error.properties
          };
          console.log(JSON.stringify({
            error: e
          }));
          throw error;
        }
        // 生成一个代表docxtemplater对象的zip文件（不是一个真实的文件，而是在内存中的表示）
        let out = doc.getZip().generate({
          type: "blob",
          mimeType: "application/vnd.openxmlformats-officedocument.wordprocessingml.document"
        });
        // 将目标文件对象保存为目标类型的文件，并命名
        console.log(out)
        saveAs(out, _this.outwordname);

        // _this.fileDoc = out;
        // _this.$nextTick(() => {
        //   _this.preview(out)
        // })
      });
    },
    previewDoc() {
      if (this.selected.length === 0) {
        this.dialog = false;
        alert("请选择需要预览的文件")
      } else if (this.selected.length > 1) {
        this.dialog = false;
        alert("你选中了多条数据 预览模式只支持单个文件演示")
      } else {
        this.dialog = true;
        this.selectList = [];
        this.selectList = this.selected;
        this.previewDocOne();
      }
    },

    previewDocOne() {
      let _this = this;
      // 读取并获得模板文件的二进制内容
      JSZipUtils.getBinaryContent("a1.docx", function (error, content) {
        // input.docx是模板。我们在导出的时候，会根据此模板来导出对应的数据
        // 抛出异常
        if (error) {
          throw error;
        }
        // 创建一个JSZip实例，内容为模板的内容
        let zip = new JSZip(content);
        // 创建并加载docxtemplater实例对象
        let doc = new docxtemplater().loadZip(zip);
        // 设置模板变量的值
        doc.setData({
          ..._this.selectList[0], //这里是上面form表单的内容
        });
        try {
          // 用模板变量的值替换所有模板变量
          doc.render();
        } catch (error) {
          // 抛出异常
          let e = {
            message: error.message,
            name: error.name,
            stack: error.stack,
            properties: error.properties
          };
          console.log(JSON.stringify({
            error: e
          }));
          throw error;
        }
        // 生成一个代表docxtemplater对象的zip文件（不是一个真实的文件，而是在内存中的表示）
        let out = doc.getZip().generate({
          type: "blob",
          mimeType: "application/vnd.openxmlformats-officedocument.wordprocessingml.document"
        });
        // 将目标文件对象保存为目标类型的文件，并命名
        console.log(out)
        // saveAs(out, _this.outwordname);
        _this.$nextTick(() => {
          _this.preview(out)
        })
      });
    },
    selectedExport() {
      this.selectList = this.selected;
      if (this.selectList.length === 0) {
        alert("请选中需要导出的文件")
      } else {
        for (let i = 0; i < this.selected.length; i++) {
          this.modifyword(i);
          console.log(i)
        }
      }

    },

    upload() {
      this.$refs.file.click()
    },
    fileChange(e) {
      let formData = new FormData()
      formData.append('file', e.target.files[0])
      this.$axios({
        url: 'http://127.0.0.1:9090/api/elicit',
        method: 'post',
        data: formData,
        processData: false,// 告诉axios不要去处理发送的数据(重要参数)
        contentType: false,   // 告诉axios不要去设置Content-Type请求头
      }).then((res) => {
        // this.form = res.data.data
        console.log(res)
        if (res.data.data.check == 'false') {
          res.data.data.check = false
          res.data.data.checked = false
        }
        this.desserts.push(res.data.data)
        console.log(this.desserts)
      })
    },
  }
}
</script>

<style lang="less">
.header {
  margin: 20px;
}
</style>
