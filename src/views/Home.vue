<template>
  <div class="file">
    <div class="upload">
      <v-btn @click="upload">上传文件</v-btn>
      <input ref="file" type="file" style="display:none" accept=".doc, .docx" @change="fileChange">
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
      <template v-slot:item.iron>
        <div>
          <v-btn color="error"
                 @click="previewDoc()">预览文件
          </v-btn>

        </div>
      </template>


    </v-data-table>
    <v-btn @click="selectedExport()">导出doc</v-btn>

    <!--      遮罩层-->
    <v-overlay :value="overlay">
      <v-btn
          icon
          @click="overlay = false"
      >
        <v-icon>mdi-close</v-icon>
      </v-btn>

      <div v-for="item in templateList">
        <v-card-text>{{item}}</v-card-text>
      </div>
    </v-overlay>
  </div>
</template>

<script>

export default {
  name: "home",
  data: () => ({

    overlay: false,
    file: null,
    templateList: [],

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

  }),

  methods: {
    previewDoc() {
      this.overlay = !this.overlay;
      this.templateList = this.selected
    },
    selectedExport() {
      console.log(this.selected)
    },

    upload() {
      this.$refs.file.click()
    },
    fileChange(e) {
      let formData = new FormData()
      formData.append('file', e.target.files[0])
      this.$axios({
        url: 'http://192.168.1.142:9090/api/elicit',
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
.file {
  margin-top: 20px;
}
</style>
