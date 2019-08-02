<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" label-width="80px">
    <el-form-item label="文章标签" prop="tag">
      <!--<el-input v-model="dataForm.tag" placeholder="文章标签"></el-input>-->
      <el-checkbox-group v-model="dataForm.tags">
        <el-checkbox v-for="tag in tagList" :key="tag.id" :label="String(tag.id)" border>{{ tag.tagName }}</el-checkbox>
      </el-checkbox-group>
    </el-form-item>
    <el-form-item label="标题" prop="title">
      <el-input v-model="dataForm.title" placeholder="标题"></el-input>
    </el-form-item>
    <el-form-item label="文章内容" prop="content">
      <!--<el-input v-model="dataForm.content" placeholder="文章内容"></el-input>-->
      <quill-editor class="ql-editor-class"
        v-model="dataForm.content"
        ref="myQuillEditor"
        :options="editorOption"
        @blur="onEditorBlur($event)" @focus="onEditorFocus($event)"
        @change="onEditorChange($event)">
      </quill-editor>
    </el-form-item>
    <el-form-item label="置顶功能" prop="topNum">
      <!--<el-input v-model="dataForm.topNum" placeholder="置顶功能"></el-input>-->
      <el-radio-group v-model="dataForm.topNum">
        <el-radio :label="0">不置顶</el-radio>
        <el-radio :label="1">置顶</el-radio>
      </el-radio-group>
    </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  import { quillEditor } from 'vue-quill-editor'
  import * as Quill from 'quill' // 引入编辑器
  import { ImageDrop } from 'quill-image-drop-module'
  import ImageResize from 'quill-image-resize-module'
  Quill.register('modules/imageResize', ImageResize)
  Quill.register('modules/imageDrop', ImageDrop)
  export default {
    data () {
      return {
        editorOption: {
          modules: {
            toolbar: [
              ['bold', 'italic', 'underline', 'strike'],    //加粗，斜体，下划线，删除线
              ['blockquote', 'code-block'],     //引用，代码块

              [{ 'header': 1 }, { 'header': 2 }],        // 标题，键值对的形式；1、2表示字体大小
              [{ 'list': 'ordered'}, { 'list': 'bullet' }],     //列表
              [{ 'script': 'sub'}, { 'script': 'super' }],   // 上下标
              [{ 'indent': '-1'}, { 'indent': '+1' }],     // 缩进
              [{ 'direction': 'rtl' }],             // 文本方向

              [{ 'size': ['small', false, 'large', 'huge'] }], // 字体大小
              [{ 'header': [1, 2, 3, 4, 5, 6, false] }],     //几级标题

              [{ 'color': [] }, { 'background': [] }],     // 字体颜色，字体背景颜色
              [{ 'font': [] }],     //字体
              [{ 'align': [] }],    //对齐方式

              ['clean'],    //清除字体样式
              ['image','video']    //上传图片、上传视频

            ],
            imageDrop: true,
            imageResize: {}
          },
          theme: 'snow'
        },
        visible: false,
        dataForm: {
          id: 0,
          tag: '',
          title: '',
          content: '',
          topNum: '',
          tags: []
        },
        tagList: [],
        dataRule: {
          tags: [
            { required: true, message: '文章标签不能为空', trigger: 'blur' }
          ],
          title: [
            { required: true, message: '标题不能为空', trigger: 'blur' }
          ],
          content: [
            { required: true, message: '文章内容不能为空', trigger: 'blur' }
          ],
          topNum: [
            { required: true, message: '置顶功能不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    mounted () {
      this.getTagList()
    },
    computed: {
      editor () {
        return this.$refs.myQuillEditor.quill
      }
    },
    methods: {
      onEditorReady(editor) { // 准备编辑器
      },
      onEditorBlur(){}, // 失去焦点事件
      onEditorFocus(){}, // 获得焦点事件
      onEditorChange(){}, // 内容改变事件
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/blog/blogarticle/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.tags = data.blogArticle.tag.split(',')
                this.dataForm.title = data.blogArticle.title
                this.dataForm.content = data.blogArticle.content
                this.dataForm.topNum = data.blogArticle.topNum
              }
            })
          }
        })
      },
      getTagList () {
        this.$http({
          url: this.$http.adornUrl('/blog/blogtag/list'),
          method: 'get'
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.tagList = data.page.list
          } else {
            this.tagList = []
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/blog/blogarticle/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id,
                'tag': this.dataForm.tags.join(','),
                'title': this.dataForm.title,
                'content': this.dataForm.content,
                'topNum': this.dataForm.topNum,
                'createBy': this.$store.state.user.name
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
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
  .ql-editor-class {
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    line-height: 1.42;
    height: 100%;
    outline: none;
    padding: 0 !important;
    tab-size: 4;
    -moz-tab-size: 4;
    text-align: left;
    word-wrap: break-word;
  }
  .ql-editor{
    height:300px;
  }
</style>
