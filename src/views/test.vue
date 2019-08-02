<template>
  <div class="edit_container">
    <quill-editor
      v-model="content"
      ref="myQuillEditor"
      :options="editorOption"
      @blur="onEditorBlur($event)" @focus="onEditorFocus($event)"
      @change="onEditorChange($event)">
    </quill-editor>
    <button v-on:click="saveHtml">保存</button>
  </div>
</template>

<script>
  import { quillEditor } from 'vue-quill-editor'
  import * as Quill from 'quill' // 引入编辑器
  import { ImageDrop } from 'quill-image-drop-module'
  Quill.register('modules/imageDrop', ImageDrop)
  export default {
    name: 'App',
    data(){
      return {
        content: `<p>hello world</p>`,
        editorOption: {
          modules: {
            imageDrop: true,
          },
          theme: 'snow'
        }
      }
    },
    computed: {
      editor () {
        return this.$refs.myQuillEditor.quill
      },
    },
    methods: {
      onEditorReady(editor) { // 准备编辑器
      },
      onEditorBlur(){}, // 失去焦点事件
      onEditorFocus(){}, // 获得焦点事件
      onEditorChange(){}, // 内容改变事件
      saveHtml:function(event){
        alert(this.content);
      }
    }
  }
</script>

<style>
  .edit_container {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
  .ql-editor{
    height:400px;
  }
</style>
