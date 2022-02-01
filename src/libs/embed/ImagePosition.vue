<template>
    <div class="image-handler" v-if="handlerData.isShow" v-bind:style="handlerData.position">
        <div class="image-hander-menu">
            <button class="btn-toggle" v-on:click="imageSizing('is-normal')">
                <img src="../../assets/icons/image-align-normal-active.png" v-if="handlerData.currentSize == 'is-normal'">
                <img src="../../assets/icons/image-align-normal.png" v-if="handlerData.currentSize != 'is-normal'">
            </button>
            <button class="btn-toggle" v-on:click="imageSizing('is-expand')">
                <img src="../../assets/icons/image-align-expand-active.png" v-if="handlerData.currentSize == 'is-expand'">
                <img src="../../assets/icons/image-align-expand.png" v-if="handlerData.currentSize != 'is-expand'">
            </button>
            <button class="btn-toggle" v-on:click="imageSizing('is-full')">
                <img src="../../assets/icons/image-align-full-active.png" v-if="handlerData.currentSize == 'is-full'">
                <img src="../../assets/icons/image-align-full.png" v-if="handlerData.currentSize != 'is-full'">
            </button>
        </div>
    </div>
</template>

<script>
export default {
  props: [
    'handler'
  ],
  emits: ['onPositionChange'],
  data () {
    return {
      // copy of props
      handlerData: undefined
    }
  },
  mounted () {
    this.handlerData = this.handler
  },
  methods: {
    imageSizing (sizing) {
      this.handlerData.currentSize = sizing
      this.handlerData.currentLine.className = 'editor-image ' + sizing

      this.$emit('onPositionChange')
    }
  }
}
</script>
