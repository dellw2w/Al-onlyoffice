<template>
  <div>
    <input type="file" id="file" accept=".docx">
  </div>
  <DocumentEditor
    id="docEditor"
    documentServerUrl="http://192.168.147.137:8082/"
    :config="config"
    :events_onDocumentReady="onDocumentReady"
    :onLoadComponentError="onLoadComponentError"
  />
  <div style="height: 100vh; display: inline-block;width:20%;">
    <label for="" >当前选中内容：
      <textarea type="textarea" v-model="selectedText" @blur="searchTextBlur"></textarea>
    </label>

    <label for="" >智能生成内容：
      <textarea type="textarea" v-model="searchText"></textarea>
    </label>
    <div>
      <button @click="pastHtml">合并内容</button>
    </div>
  </div>
  
</template>
<style>
  iframe {
    width: 80%!important;
  }
  textarea {
    width: 100%;
    height: 200px;
  }
</style>
<script>
import { defineComponent } from "vue";
import { DocumentEditor } from "@onlyoffice/document-editor-vue";
import axios from "axios";

export default defineComponent({
  name: "ExampleComponent",
  components: {
    DocumentEditor,
  },
  data() {
    return {
      config: {
        document: {
          fileType: "docx",
          key: "Khirz6zTPdfd7"+Math.random(),
          title: "Example Document Title.docx",
          url: window.location.protocol+'//'+window.location.host+"/new.docx",
          permissions:{
            review:true
          }
        },
        documentType: "word",
        editorConfig: {
          callbackUrl: window.location.protocol+'//'+window.location.host+"/track",
          "lang": "zh-CN", //语言环境
          "user": { //用户信息
						"id": "admin", //用户ID
						"name": "15313637890" //用户全名称
					},
          "plugins":{
              "pluginsData": [
                  window.location.protocol+'//'+window.location.host+"/plugins/example_work_with_content_controls_navigation/config.json"
              ],
              autostart : ["asc.{C36DDFB5-08F0-4A68-B829-5FB1F7D49728}"]
          },
          "customization":{
            forcesave:true
          }
          
        },
      },
      selectedText:'',
      searchText:'',
    };
  },
  watch: {
   
  },
  methods: {
    onDocumentReady() {
      console.log(window.location.protocol+'//'+window.location.host)
      console.log(DocumentEditor)
      console.log("Document is loaded");
      window.connector = window.DocEditor.instances.docEditor.createConnector();
      this.textSelected();
      this.postFile()
    },
    onLoadComponentError(errorCode, errorDescription) {
      switch (errorCode) {
        case -1: // Unknown error loading component
          console.log(errorDescription);
          break;

        case -2: // Error load DocsAPI from http://documentserver/
          console.log(errorDescription);
          break;

        case -3: // DocsAPI is not defined
          console.log(errorDescription);
          break;
      }
    },
    textSelected() {
      window.addEventListener("message", (event) => {
        if(event.data=='isSelected') {
            connector.executeMethod("GetSelectedText", [], (text) => {
              this.selectedText = text
              this.searchText = '我是被智能生成的文本:'+this.selectedText
            })
            // POST请求传参
            fetch("http://localhost:3000/books", {
              method: "post",
              body: JSON.stringify({
                uname: "张三",
                pwd: "456"
              }),
              headers: {
                "Content-Type": "application/json"
              }
            })
            .then(function (data) {
              this.searchText = '我是被智能生成的文本:'+data.text()
              return data.text();
            })
            .then(function (data) {
              console.log(data);
            });
        }
      });
    },
    pastHtml() {
      connector.executeMethod("PasteText", [this.searchText]);
    },
    searchTextBlur() {
      connector.executeMethod("PasteText", [this.selectedText]);
    },
    postFile() {
      document.getElementById('file').addEventListener('change', function() {
          var file = this.files[0];
          var reader = new FileReader();
          reader.readAsDataURL(file);
          reader.onload = function() {
              var content = reader.result;
              // POST请求传参
            axios.post(window.location.protocol+'//'+window.location.host+'/upLocalFile', {
                content:content
            }).then(function(data) {
                window.location.reload()
            })  
          };
      });
    }
  },
});
</script>
