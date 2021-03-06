<template>
    <div class="image-handler-container">
        <div class="insert-image-container" v-show="insert && insert.isShow" v-bind:style="insert.position">
            <div class="insert-image-toggle">
                <button @click="toggle" class="btn-toggle">
                    <i class="fas fa-plus"></i>
                </button>
            </div>
            <div class="insert-image-menu" v-show="insert && insert.isToggle">
                <insert-image
                    v-if="!hideImage"
                    :editor="editor"
                    :insert="insert"
                    :editorRef="editorRef"
                    :uploadUrl="uploadUrl"
                    :uploadUrlHeader="uploadUrlHeader"
                    :file_input_name="file_input_name"
                    :file_size="file_size"
                    :imgur_bool="imgur_bool"
                    :handler="handler"
                    @uploaded="uploadCallback"
                    @imageClick="imageClickHandler"
                    title="Insert Image"
                ></insert-image>
                <insert-gist v-if="insert && !hideGist" :editor="editor"
                    @onChange="onChange" :insert="insert" title="Insert gist"></insert-gist>
            </div>
        </div>
        <image-position
            :handler="handler"
            @onPositionChange="onChange"
            ></image-position>
    </div>
</template>

<script>
import InsertImage from './embed/InsertImage'
import InsertGist from './embed/InsertGist'
import ImagePosition from './embed/ImagePosition'

import _ from 'underscore'

export default {
  components: {
    InsertImage,
    InsertGist,
    ImagePosition
  },
  data () {
    return {
      insert: {
        position: {
          top: '0',
          left: '0'
        },
        isShow: false,
        isToggle: false,
        embedElm: null,
        files: [],
        focusLine: null
      },
      handler: {
        currentLine: null,
        currentImg: null,
        currentSize: 'is-normal',
        position: {
          top: '0'
        },
        isShow: false
      }
    }
  },
  props: [
    'editor',
    'uploadUrl',
    'uploadUrlHeader',
    'file_input_name',
    'file_size',
    'imgur_bool',
    'editorRef',
    'onChange',
    'hideGist',
    'hideImage'
  ],
  emits: ['uploaded'],
  methods: {
    subscribe () {
      this.editor.subscribe('editableKeyup', this.detectShowToggle)
      this.editor.subscribe('editableClick', this.detectShowToggle)
      this.editor.subscribe('editableKeyup', this.detectImageDescription)
      this.editor.subscribe('editableClick', this.detectImageDescription)
    },
    unsubscribe () {
      this.editor.unsubscribe('editableKeyup', this.detectShowToggle)
      this.editor.unsubscribe('editableClick', this.detectShowToggle)
      this.editor.unsubscribe('editableKeyup', this.detectImageDescription)
      this.editor.unsubscribe('editableClick', this.detectImageDescription)
    },
    detectImageDescription () {
      const focused = this.editor.getFocusedElement()
      if (!focused) return

      const editorImages = focused.getElementsByClassName('editor-image-description')
      _.map(editorImages, (elm) => {
        const description = elm.innerHTML.trim()
        // eslint-disable-next-line eqeqeq
        if (!description || description == '<br>') {
          elm.className = 'editor-image-description is-empty'
        } else {
          elm.className = 'editor-image-description'
        }
      })
    },
    detectShowToggle (e) {
      if (this.insert.isShow && this.insert.isToggle) {
        this.toggle()
      }

      if (e.keyCode === 13) {
        const focused = this.editor.getSelectedParentElement()
        const nextElm = focused.nextElementSibling
        const prevElm = focused.previousElementSibling
        if (nextElm && prevElm && nextElm.className.indexOf('editor-image-description') > -1 && prevElm.className.indexOf('editor-image') > -1) {
          nextElm.parentNode.insertBefore(nextElm, focused)
        }
      }
      this.handler.isShow = false
      if (e.target.className.indexOf('editor-image-description') <= -1) {
        const editorImages = this.editor.getFocusedElement().getElementsByClassName('editor-image')
        _.map(editorImages, (imgElm) => {
          imgElm.className = imgElm.className.replace('is-focus', '')
        })
      }
      const currentLine = this.editor.getSelectedParentElement()
      const outerHtml = currentLine.outerHTML
      const isList = outerHtml.indexOf('<li>') > -1
      const content = currentLine.innerHTML.replace(/^(<br\s*\/?>)+/, '').trim()
      if (content || isList) {
        // Not show toggle if focus line has content & list
        this.insert.isShow = false
        this.insert.isToggle = false
        this.insert.focusLine = null
      } else {
        const currentPos = currentLine.getBoundingClientRect()
        this.insert.position.top = currentPos.top + 'px'
        this.insert.position.left = currentPos.left + 'px'
        this.insert.isShow = true
        this.insert.focusLine = currentLine
      }
    },
    toggle () {
      this.insert.isToggle = !this.insert.isToggle
      const el = document.getElementsByClassName('editor medium-editor-element')
      if (el.length > 0) el[0].removeAttribute('data-placeholder')
    },
    imageClickHandler (value) {
      this.handler = value
    },
    uploadCallback (url) {
      this.$emit('uploaded', url)
    },
    handleScroll () {
      this.handler.isShow = false
      if (this.insert.isShow) {
        const currentLine = this.editor.getSelectedParentElement()
        const currentPos = currentLine.getBoundingClientRect()
        this.insert = {
          ...this.insert,
          isShow: true,
          focusLine: currentLine,
          position: {
            top: currentPos.top + 'px',
            left: currentPos.left + 'px'
          }
        }
      }
    }
  },
  mounted () {
    this.subscribe()
    const el = document.getElementsByClassName('editor medium-editor-element')
    if (el.length > 0) el[0].setAttribute('data-medium-focused', true)
  },
  unmounted () {
    this.unsubscribe()
  },
  beforeMount () {
    this.window = window
    window.addEventListener('scroll', this.handleScroll)
  },
  beforeUnmount () {
    window.removeEventListener('scroll', this.handleScroll)
  }
}
</script>
