# WangEditor使用方法

[toc]



## 参考文档

[官方文档](https://www.wangeditor.com/v5/installation.html)

[官方示例](https://github.com/wangfupeng1988/vue3-wangeditor-demo)



## 安装

```shell
npm install @wangeditor/editor --save
npm install @wangeditor/editor-for-vue@next --save
```



## 示例代码

主要修改`Editor.vue`组件中绑定的函数以及编辑器的配置`editorConfig`。

代码路径：`/src/components/MyEditor.vue`

```vue
<template>
  <div style="border: 1px solid #ccc">
    <Toolbar
        style="border-bottom: 1px solid #ccc"
        :editor="editorRef"
        :defaultConfig="toolbarConfig"
        :mode="mode"
    />
    <Editor
        style="height: 500px; overflow-y: hidden;"
        v-model="valueHtml"
        :defaultConfig="editorConfig"
        :mode="mode"
        @onCreated="handleCreated"
        @onChange="handleChange"
        @onDestroyed="handleDestroyed"
        @onFocus="handleFocus"
        @onBlur="handleBlur"
        @customAlert="customAlert"
        @customPaste="customPaste"
    />
  </div>
</template>

<script>
import '@wangeditor/editor/dist/css/style.css' // 引入 css

import { onBeforeUnmount, ref, shallowRef, onMounted } from 'vue'
import { Editor, Toolbar } from '@wangeditor/editor-for-vue'

export default {
  components: { Editor, Toolbar },
  setup() {
    // 编辑器实例，必须用 shallowRef
    const editorRef = shallowRef()

    // 内容 HTML
    const valueHtml = ref('<p>hello</p>')

    // 模拟 ajax 异步获取内容
    onMounted(() => {
      setTimeout(() => {
        valueHtml.value = '<p>模拟 Ajax 异步设置内容</p>'
      }, 1500)
    })

    const toolbarConfig = {}

    // 编辑器配置
    const editorConfig = {
      placeholder: '请输入内容...',
      // 配置菜单
      MENU_CONF: {
        // 上传图片配置
        uploadImage: {
          fieldName: 'file',
          server: 'http://localhost:8080/upload/single-file',
          // server: 'http://localhost:8080/upload/cloud',
          customInsert(res, insertFn) {
            // 返回值必须是键值对的形式，否则会报错
            insertFn(res.path, '', '')
          },
          onBeforeUpload(file) {
            console.log('开始上传')
          },
          onSuccess(file, res) {
            console.log(`${file.name} 上传成功`, res)
          }

        }
      }
    }

    // 组件销毁时，也及时销毁编辑器
    onBeforeUnmount(() => {
      const editor = editorRef.value
      if (editor == null) return
      editor.destroy()
    })

    // 编辑器创建时调用此函数
    const handleCreated = (editor) => {
      console.log('created', editor);
      editorRef.value = editor // 记录 editor 实例，重要！
    }

    // 编辑器内容改变后调用此函数
    const handleChange = (editor) => {
      console.log('change:', editor.getHtml());
    }

    //
    const handleDestroyed = (editor) => {
      console.log('destroyed', editor)
    }

    // 编辑器聚焦时调用此函数
    const handleFocus = (editor) => {
      console.log('focus', editor)
    }

    // 编辑器失焦时调用此函数
    const handleBlur = (editor) => {
      console.log('blur', editor)
    }

    const customAlert = (info, type) => {
      alert(`【自定义提示】${type} - ${info}`)
    }

    // 编辑器中发送粘贴事件时调用此函数
    const customPaste = (editor, event, callback) => {
      console.log('ClipboardEvent 粘贴事件对象', event)
      // 自定义插入内容
      editor.insertText('xxx')
      // 返回值（注意，vue 事件的返回值，不能用 return）
      callback(false) // 返回 false ，阻止默认粘贴行为
      // callback(true) // 返回 true ，继续默认的粘贴行为
    }

    const insertText = () => {
      const editor = editorRef.value
      if (editor == null) return
      editor.insertText('hello world')
    }

    const printHtml = () => {
      const editor = editorRef.value
      if (editor == null) return
      console.log(editor.getHtml())
    }

    const disable = () => {
      const editor = editorRef.value
      if (editor == null) return
      editor.disable()
    }

    return {
      editorRef,
      valueHtml,
      mode: 'default', // 'default' 或 'simple'
      toolbarConfig,
      editorConfig,
      handleCreated,
      handleChange,
      handleDestroyed,
      handleFocus,
      handleBlur,
      customAlert,
      customPaste,
      insertText,
      printHtml,
      disable,
    };
  }
}
</script>

<style scoped>

</style>
```



`/src/App.vue`中引入该组件：

```vue
<script setup>

import MyEditor from "./components/MyEditor.vue";
</script>

<template>
  <div class="editor">
    <MyEditor></MyEditor>
  </div>
</template>

<style scoped>
  .editor {
    width: 800px;
    margin: 0 auto;
  }
</style>
```

